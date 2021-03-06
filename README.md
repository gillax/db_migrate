# db_migrate
Railsのマイグレーションに似せたflyway-gradle-pluginの使い方。

アイグレーションファイルの作成場所を `${プロジェクトルート}/db/migrate/` に指定し、
ファイル名は `yyyyMMddHHmmss_description.sql` で作成する。

また、flyway-gradle-pluginではマイグレーションファイルを作成するtaskがないので、
上記フォーマットにしたがってマイグレーションファイルを作成する `flywayNew` タスクを追加した。


Requirements
-----

+ JDK 6 以上

[flyway-gradle-plugin](https://github.com/flyway/flyway/tree/master/flyway-gradle-plugin)
を利用してDBマイグレーションを行うため、
詳細な内容、使用方法については [Gradle Plugin - Documentation](http://flywaydb.org/documentation/gradle/)を参照。



How to Use
-----
gradle タスクは以下のとおり。

|Task|Description|
|----|-----------|
|flywayNew|マイグレーションファイルのテンプレートを作成する。バージョンは現在日時(UTC)。|
|flywayMigrate|Migrates the database|
|flywayClean|Drops all objects in the configured schemas|
|flywayInfo|Prints the details and status information about all the migrations|
|flywayValidate|Validates the applied migrations against the ones available on the classpath|
|flywayBaseline|Baselines an existing database, excluding all migrations up to and including baselineVersion|
|flywayRepair|Repairs the metadata table|

flywayNewはオプション `desc` を引数にとり、マイグレーションファイルの要約を渡すことができる。


### マイグレーションファイルを作成

for Linux or mac

```
$ ./gradlew flywayNew
:flywayNew
created a migration file!
     => db/migrate/yyyyMMddHHmmss_description_here.sql

BUILD SUCCESSFUL

Total time: 6.402 secs
```

必要に応じて `description_here` 部分を今回のマイグレートの要約に書き換える。

もしくは `:flywayNew` 実行時に引数(desc)で要約を渡す。

```
$ ./gradlew flywayNew -Pdesc="Add users to enabled"
:flywayNew
created a migration file!
     => db/migrate/yyyyMMddHHmmss_add_users_to_enabled.sql

BUILD SUCCESSFUL

Total time: 6.402 secs
```


マイグレーションファイルの中身を実装する。

```
$ cat <<EOF >> db/migrate/yyyyMMddHHmmss_add_users_to_enabled.sql
> ALTER TABLE users
> ADD COLUMN enabled TINYINT NOT NULL DEFAULT 1;
> EOF

$ cat db/migrate/yyyyMMddHHmmss_add_users_to_enabled.sql
/*
created on yyyy-MM-dd hh:mm:ss UTC.
*/
ALTER TABLE users
ADD COLUMN enabled TINYINT NOT NULL DEFAULT 1;
```


### マルチスキーマ対応
DB毎にマイグレーションファイルの設置場所と `gradle.properties` 内のDB接続情報を設定することで
flywayの実行を切り替える。

e.g. DB1とDB2のマイグレーションを管理する

1. `gradle.properties` にDB1とDB2の設定を記載する

```gradle.properties
# DB1
flyway.test1.url=jdbc:h2:file:./test1
flyway.test1.uesr=sa
flyway.test1.password=
flyway.test1.locations=filesystem:db/test1/migrate

# DB2
flyway.test2.url=jdbc:h2:file:./test2
flyway.test2.uesr=sa
flyway.test2.password=
flyway.test2.locations=filesystem:db/test2/migrate
```

1. locationsで指定したパスにマイグレーションファイルを作成

```
$ ./gradlew flywayNew -Pdesc="Create new migration for DB1" -Penv="test1"
```

`-Penv="xxx"` で環境をスイッチする

1. migrate

```
$ ./gradlew flywayMigrate -Penv="test1"
```
