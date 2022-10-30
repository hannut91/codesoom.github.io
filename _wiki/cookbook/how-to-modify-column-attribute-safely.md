---
layout  : wiki
title   : 안전하게 테이블 컬럼 속성 변경하기
date    : 2022-10-31 00:03:00 +0900
updated : 2022-10-31 00:03:00 +0900
author  : 한지영
tag     : 
toc     : true
public  : true
parent  : cookbook
latex   : false
---
* TOC
{:toc}

## 문제
코드숨 공부방은 최근 회원의 이메일 인증 기능을 도입했습니다. 이메일 인증에는 '이메일 토큰'이 사용되는데, 처음에 토큰의 길이를 30으로 제한해둔 상태에서 실 서버에 배포를 했다가, 
`Data too long for column 'id' at row 1` 오류가 발생하여 길이 제한을 없앴습니다. 로컬에서 컬럼 길이가 255자로 잘 바뀐 것을 확인한 후 다시 배포를 했는데, 실 서버에서는 여전히 같은 오류가 발생했고, 스키마를 확인해 보니 컬럼 길이도 여전히 30으로 제한되어 있었습니다.

## 원인
코드숨 공부방 프로젝트는 설정 파일을 2개 사용하고 있습니다. 로컬 테스트용 설정 파일은 hibernate.ddl-auto가 `create-drop`으로 되어 있고, 실제 서버 배포용 설정 파일은 `update`로 되어 있습니다. 왜냐하면 로컬에서 실행할 때에는 항상 샘플 데이터를 넣어주기 때문에 SesseionFactory 종료 시 모든 스키마를 삭제해야 하는 반면 실제 서비스에 사용되는 DB는 기존 데이터를 삭제하면 안 되기 때문이죠.  

### 참고) DDL 자동 생성 옵션
- create: 이전 데이터를 삭제하고 스키마 생성  
- create-drop: `SessionFactory`가 명시적으로 종료될 때(응용 프로그램이 중지될 때) 스키마 삭제  
- none: 아무것도 하지 않음  
- update: 스키마 업데이트  
- validate : 스키마의 유효성을 검증하고 DB를 변경하지 않음  

`ddl-auto: update`일 경우 Schema Managing이 어떻게 이루어지는지 살펴보며 문제의 원인을 찾아보았습니다.

### Schema Managing 과정

실험 환경)
- 현재 EmailToken 테이블의 스키마는 아래와 같습니다.

|Field|Type|Null|Key|Default|Extra|
|---|---|---|---|---|---|
|id|varchar(30)|NO|PRI|NULL||
|expiration_date|datetime(6)|YES|| NULL||
|expired|bit(1)|NO||NULL||
|user_id|bigint(20)|YES|| NULL||

- 실 서버의 설정 파일과 같이 ddl-auto를 update로 설정하고 EmailToken 엔티티의 id 컬럼 길이 제한을 255로 변경한 후 디버깅을 진행했습니다.

#### SchemaManagementTool 결정
```java
public class SchemaManagementToolCoordinator {
    ...
    private static void performDatabaseAction(...) {
        switch ( action ) { // 1)
        ...
            case UPDATE: { // 2)
            ...
                tool.getSchemaMigrator( executionOptions.getConfigurationValues() )
                .doMigration(
                    metadata,
                    executionOptions,
                    migrateDescriptor
                );
                break;
            }
        ...
```
1) action[^action]에 따라 사용되는 `SchemaManagementTool`이 결정됩니다.  
2) action이 UPDATE인 경우 `SchemaMigrator.doMigration()`메서드가 호출됩니다.

#### 마이그레이션
```java
public class GroupedSchemaMigratorImpl extends AbstractSchemaMigrator {
    ...
    @Override
    protected NameSpaceTablesInformation performTablesMigration(...) {
        final NameSpaceTablesInformation tablesInformation = 
            new NameSpaceTablesInformation( metadata.getDatabase().getJdbcEnvironment().getIdentifierHelper() );

        if ( schemaFilter.includeNamespace( namespace ) ) {
            ...
            for ( Table table : namespace.getTables() ) { // 3)
                if ( schemaFilter.includeTable( table ) && table.isPhysicalTable() ) {
                    ...
                    final TableInformation tableInformation = tables.getTableInformation( table ); // 4)
                    if ( tableInformation == null ) {
                        createTable( table, dialect, metadata, formatter, options, targets );
                    }
                    else if ( tableInformation.isPhysicalTable() ) { // 5)
                        tablesInformation.addTableInformation( tableInformation ); // 6)
                        migrateTable( table, tableInformation, dialect, metadata, formatter, options, targets ); // 7)
                    }
            ...
```

3) table에는 **엔티티**의 정보가 들어있습니다. 따라서 table(email_token 테이블)의 id 컬럼 길이는 255입니다.  
4) **데이터베이스에서** 기존에 존재하는 email_token 테이블의 정보를 가져옵니다. 따라서 tableInformation에서의 id 컬럼 길이는 30입니다.  
5) 4)에서 가져온 정보가 존재하므로 `else if`문이 실행됩니다.  
6) tablesInformation에 4)에서 가져온 테이블 정보를 추가합니다(length=30).  
7) `migrateTable()` 메서드를 호출합니다.

```java
public abstract class AbstractSchemaMigrator implements SchemaMigrator {
    ...
    protected void migrateTable(...) {
        final Database database = metadata.getDatabase();
        
        applySqlStrings(
            false,
            table.sqlAlterStrings( // 8)
                dialect,
                metadata,
                tableInformation,
                database.getDefaultNamespace().getPhysicalName().getCatalog(),
                database.getDefaultNamespace().getPhysicalName().getSchema()
            ),
            formatter,
            options,
            targets
        );
    }
    ...
```
8) 3)에서 엔티티로 만든 Table에 대해 `sqlAlterStrings()`메서드를 호출하여 alter 명령어를 만듭니다.

```java
public class Table implements RelationalModel, Serializable, Exportable {
    ...
    public Iterator<String> sqlAlterStrings(...) throws HibernateException {
        ...
        Iterator<Column> iter = getColumnIterator();
        List<String> results = new ArrayList<>();
        
        while ( iter.hasNext() ) {
            final Column column = (Column) iter.next();
            final ColumnInformation columnInfo = tableInfo.getColumn( Identifier.toIdentifier( column.getName(), column.isQuoted() ) );
            
            if ( columnInfo == null ) { // 9)
                // the column doesnt exist at all.
                StringBuilder alter = new StringBuilder( root.toString() )
                    .append( ' ' )
                    .append( column.getQuotedName( dialect ) )
                    .append( ' ' )
                    .append( column.getSqlType( dialect, metadata ) ); // 9-1)
                    ...
            }
        }
        ...
        if ( results.isEmpty() ) { // 10)
            log.debugf( "No alter strings for table : %s", getQuotedName() );
        }
        
        return results.iterator();
    }
    ...
```

9) 만약 기존 컬럼 정보가 없었다면,  
9-1) 여기서 컬럼 length에 대한 설정이 추가됩니다.  
10) 하지만 우리는 이미 email_token 테이블의 모든 컬럼에 대한 정보를 가지고 있기 때문에, sql alter 명령어가 만들어지지 않습니다.

즉 한번 테이블이 생성되어 있으면 기존 정보의 유무에 따라 alter 명령어가 실행되는데, id 컬럼의 length가 30인 email_token 테이블이 이미 존재하기 때문에 테이블의 속성이 바뀌지 않았던 것이죠.

## 해결 방법
코드숨 공부방 애플리케이션은 이미 운영 중이기 때문에, 최대한 안전한 방법으로 email token의 id 컬럼 길이만 변경해야 합니다.  
ddl-auto를 다른 옵션으로 설정해 주면 어떨까요? create나 create-drop은 제가 원하는 대로 컬럼 속성이 변경되긴 하지만 스키마를 삭제한다는 점에서 실 서버에 절대 사용하면 안 되는 속성들입니다. none, update, validate는 스키마를 삭제하지는 않지만 컬럼 속성이 변하지 않죠.

따라서 DDL 자동 생성 옵션에 의존하지 않고 DB를 직접 변경시키는 방법밖에 없습니다.
그래서 실 서버의 DB에 접속한 후 다음 명령어를 실행시켜 컬럼 길이를 직접 변경해 주는 방법으로 해결하였습니다.

```bash
alter table email_token modify id varchar(255);
```

## 참고 자료
- [Hibernate ORM 6.1.5.Final User Guide](https://docs.jboss.org/hibernate/stable/orm/userguide/html_single/Hibernate_User_Guide.html)
- [What are the possible values of the Hibernate hbm2ddl.auto configuration and what do they do](https://stackoverflow.com/questions/438146/what-are-the-possible-values-of-the-hibernate-hbm2ddl-auto-configuration-and-wha)

## 주석
[^action]: [Enum Action](https://docs.jboss.org/hibernate/orm/6.1/javadocs/org/hibernate/tool/schema/Action.html)
