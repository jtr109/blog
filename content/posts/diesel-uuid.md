---
title: Diesel UUID Problem Note
date: 2019-11-07 22:36:29
categories:
  - tech
tags:
  - Rust
  - Diesel
typora-root-url: ../../static
---
## 背景描述
在使用 [Diesel](http://diesel.rs/) 和 [uuid](https://crates.io/crates/uuid) 开发的过程中, 使用发生报错:

```shell
error[E0277]: the trait bound `uuid::Uuid: diesel::Queryable<diesel::sql_types::Uuid, _>` is not satisfied
  --> src/models.rs:45:14
   |
45 |             .load::<App>(conn)
   |              ^^^^ the trait `diesel::Queryable<diesel::sql_types::Uuid, _>` is not implemented for `uuid::Uuid`
   |
   = note: required because of the requirements on the impl of `diesel::Queryable<(diesel::sql_types::Uuid, diesel::sql_types::Text), _>` for `(uuid::Uuid, std::string::String)`
   = note: required because of the requirements on the impl of `diesel::Queryable<(diesel::sql_types::Uuid, diesel::sql_types::Text), _>` for `models::App`
   = note: required because of the requirements on the impl of `diesel::query_dsl::LoadQuery<_, models::App>` for `diesel::query_builder::SelectStatement<schema::app::table, dies
el::query_builder::select_clause::DefaultSelectClause, diesel::query_builder::distinct_clause::NoDistinctClause, diesel::query_builder::where_clause::NoWhereClause, diesel::query
_builder::order_clause::NoOrderClause, diesel::query_builder::limit_clause::LimitClause<diesel::expression::bound::Bound<diesel::sql_types::BigInt, i64>>>`
```

代码如下:

`Cargo.toml`:

```toml
[dependencies]
diesel = { version = "1.4.2", features = ["postgres", "uuid"] }
uuid = { version = "0.7.4", features = ["v4"] }
```

`schema.rs`

```rust
table! {
    app (id) {
        id -> Uuid,
        jwt_secret -> Varchar,
    }
}
```

`models.rs`:

```rust
use diesel::pg::PgConnection;
use diesel::prelude::*;
use uuid::Uuid;

use crate::schema::app;
use crate::token::{create_claims, create_jwt_secret, encode_token, Claims, TokenError};

#[derive(Queryable)]
struct App {
    id: Uuid,
    jwt_secret: String,
}

#[derive(Insertable)]
#[table_name = "app"]
struct NewApp {
    jwt_secret: String,
}

impl App {
    fn get_by_id(&self, conn: &PgConnection) -> Vec<App> {
        app::table
            .limit(1)
            .load::<App>(conn)
            .expect("Error loading app")
    }
}
```

## 经过
在往上查询未果, 经人提点可能是项目中 uuid 和版本不一致的问题导致的.

### UUID 降级

查看 Diesel 的 [diesel/Cargo.toml at 1.4.x · diesel-rs/diesel · GitHub](https://github.com/diesel-rs/diesel/blob/1.4.x/diesel/Cargo.toml) 发现 Diesel 依赖的 [uuid 版本](https://github.com/diesel-rs/diesel/blob/1.4.x/diesel/Cargo.toml#L26)为 `>=0.2.0, <0.7.0`. 

```toml
uuid = { version = ">=0.2.0, <0.7.0", optional = true, features = ["use_std"] }
```

而我在项目中使用的 `uuid` 为 `0.7.4`. 尝试将 uuid 降级到 `0.6`.

上述问题不再报错, 不过有一个新的报错出现:

```shell
error[E0599]: no method named `to_simple` found for type `uuid::Uuid` in the current scope
  --> src/models.rs:37:45
   |
37 |         let claims = create_claims(&self.id.to_simple().to_string());
   |                                             ^^^^^^^^^ help: there is a method with a similar name: `simple`
```

可以推断, uuid 在 0.6 到 0.7 的过程中有较大的功能改变. 那新问题就来了:

_我希望使用 uuid 最新版本的能力._

### 修改 Dependence

好消息是我在 `Cargo.toml` 紧接着的一行发现了一个 dependence: [uuidv07](https://github.com/diesel-rs/diesel/blob/1.4.x/diesel/Cargo.toml#L27):

```toml
uuidv07 = { version = "0.7.0", optional = true, package = "uuid"}
```

从配置信息可以看到, 这个 `uuidv07` 同样使用的是 uuid 包 (`package = "uuid"`). 但是版本为 "0.7.0" (`version = "0.7.0"`).

尝试修改本地项目的 `Cargo.toml` 文件:

```toml
[dependencies]
diesel = { version = "1.4.2", features = ["postgres", "uuidv07"] }
uuid = { version = "0.7.4", features = ["v4"] }
```

_测试通过!_

## 结论
网上几乎没法找到这个[改动](https://github.com/diesel-rs/diesel/blob/master/CHANGELOG.md#fixed-3)相关的问题和解决方案, 甚至文档中都没有提及.

定位问题的能力还是有待提高.

#tech/rust
