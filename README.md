# Parvati

![Parvati](logo.png)

Parvati library: ORM library with no boilerplate code, written in Rust. The goal of the project is to provide a library that reduces boilerplate code. Comparing Parvati with Diesel and SeaORM will highlight the differences.

## Features

The main idea that I put into my ORM library is a minimum of stupid code and easy use of the library. I wanted users not to have to write long chains of function calls to construct a simple SQL query.

- [x] SQLite support
- [x] MySQL support

## Usage

Cargo.toml

```toml
[dependencies]
parvati = {version = "1.0.2", features = ["sqlite"]} # or "mysql"
parvati_derive = "1.0.2"
```

```rust
#[tokio::test]
async fn test() -> Result<(), ORMError> {

    let file = std::path::Path::new("file1.db");
    if file.exists() {
        std::fs::remove_file(file)?;
    }

    let _ = env_logger::Builder::from_env(env_logger::Env::new().default_filter_or("debug")).try_init();

    let conn = ORM::connect("file1.db".to_string())?;
    let init_script = "create_table_sqlite.sql";
    conn.init(init_script).await?;

    #[derive(TableDeserialize, TableSerialize, Serialize, Deserialize, Debug, Clone)]
    #[table(name = "user")]
    pub struct User {
        pub id: i32,
        pub name: Option<String>,
        pub age: i32,
    }

    let mut user = User {
        id: 0,
        name: Some("John".to_string()),
        age: 30,
    };

    let mut user_from_db: User = conn.add(user.clone()).apply().await?;

    user.name = Some("Mary".to_string());
    let  _: User = conn.add(user.clone()).apply().await?;

    let user_opt: Option<User> = conn.find_one(user_from_db.id as u64).run().await?;
    log::debug!("User = {:?}", user_opt);

    let user_all: Vec<User> = conn.find_all().run().await?;
    log::debug!("Users = {:?}", user_all);

    user_from_db.name = Some("Mike".to_string());
    let _updated_rows: usize = conn.modify(user_from_db.clone()).run().await?;


    let user_many: Vec<User> = conn.find_many("id > 0").limit(2).run().await?;
    log::debug!("Users = {:?}", user_many);

    let query = format!("select * from user where name like {}", conn.protect("M%"));
    let result_set: Vec<Row> = conn.query(query.as_str()).exec().await?;
    for row in result_set {
        let id: i32 = row.get(0).unwrap();
        let name: Option<String> = row.get(1);
        log::debug!("User = id: {}, name: {:?}", id, name);
    }

    let updated_rows = conn.query_update("update user set age = 100").exec().await?;
    log::debug!("updated_rows: {}", updated_rows);
    let updated_rows: usize = conn.remove(user_from_db.clone()).run().await?;
    log::debug!("updated_rows: {}", updated_rows);
    conn.close().await?;

    Ok(())
}
```


Example output:

```text 
[2023-09-08T13:33:22Z DEBUG parvati] "CREATE TABLE user (id INTEGER PRIMARY KEY AUTOINCREMENT, name  TEXT,age INTEGER)"
[2023-09-08T13:33:22Z DEBUG parvati] "insert into user (name,age) values (\"John\",30)"
[2023-09-08T13:33:22Z DEBUG parvati] "select * from user where rowid = 1"
[2023-09-08T13:33:22Z DEBUG parvati] "insert into user (name,age) values (\"Mary\",30)"
[2023-09-08T13:33:22Z DEBUG parvati] "select * from user where rowid = 2"
[2023-09-08T13:33:22Z DEBUG parvati] "select * from user where id = 1"
[2023-09-08T13:33:22Z DEBUG test::tests] User = Some(User { id: 1, name: Some("John"), age: 30 })
[2023-09-08T13:33:22Z DEBUG parvati] "select * from user"
[2023-09-08T13:33:22Z DEBUG test::tests] Users = [User { id: 1, name: Some("John"), age: 30 }, User { id: 2, name: Some("Mary"), age: 30 }]
[2023-09-08T13:33:22Z DEBUG parvati] "update user set name = \"Mike\",age = 30 where id = 1"
[2023-09-08T13:33:22Z DEBUG parvati] "select * from user where id > 0 limit 2"
[2023-09-08T13:33:22Z DEBUG test::tests] Users = [User { id: 1, name: Some("Mike"), age: 30 }, User { id: 2, name: Some("Mary"), age: 30 }]
[2023-09-08T13:33:22Z DEBUG parvati] "select * from user where name like \"M%\""
[2023-09-08T13:33:22Z DEBUG test::tests] User = id: 1, name: Some("Mike")
[2023-09-08T13:33:22Z DEBUG test::tests] User = id: 2, name: Some("Mary")
[2023-09-08T13:33:22Z DEBUG parvati] "update user set age = 100"
[2023-09-08T13:33:22Z DEBUG test::tests] updated_rows: 2
[2023-09-08T13:33:22Z DEBUG parvati] "delete from user where id = 1"
[2023-09-08T13:33:22Z DEBUG test::tests] updated_rows: 1
```

## Contributing
I would love to see contributions from the community. If you experience bugs, feel free to open an issue. If you would like to implement a new feature or bug fix, please follow the steps:
1. Read "[Contributor License Agreement (CLA)](https://github.com/igumnoff/parvati/blob/main/CLA)"
2. Contact with me via telegram @ievkz or discord @igumnovnsk
3. Confirm e-mail invitation in repository
4. Do "git clone"
5. Create branch with your assigned issue
6. Create pull request to main branch
