# Подключение к MySQL/MariaDB в Rust

## Введение

MySQL и MariaDB — популярные реляционные системы управления базами данных, широко используемые в веб-разработке и других областях. MySQL является одной из самых распространенных СУБД в мире, а MariaDB — это ответвление (форк) MySQL с открытым исходным кодом, созданное сообществом после приобретения MySQL компанией Oracle.

В этой главе мы рассмотрим различные способы подключения к MySQL и MariaDB из Rust-приложений, включая синхронные и асинхронные подходы, использование пулов соединений и ORM-библиотек.

## Основные библиотеки для работы с MySQL/MariaDB в Rust

Для работы с MySQL и MariaDB в Rust существует несколько основных библиотек:

1. **mysql** - нативный драйвер для синхронного подключения
2. **mysql_async** - асинхронная версия драйвера mysql
3. **r2d2-mysql** - пул соединений для mysql
4. **sqlx** - асинхронная библиотека с поддержкой MySQL
5. **diesel** - ORM с поддержкой MySQL
6. **SeaORM** - асинхронный ORM с поддержкой MySQL

## Установка MySQL/MariaDB

Прежде чем начать работу с MySQL/MariaDB в Rust, необходимо установить и настроить сервер.

### Установка MySQL на Linux (Ubuntu/Debian)

```bash
sudo apt update
sudo apt install mysql-server
```

### Установка MariaDB на Linux (Ubuntu/Debian)

```bash
sudo apt update
sudo apt install mariadb-server
```

### Установка на macOS

```bash
# MySQL
brew install mysql

# или MariaDB
brew install mariadb
```

### Установка на Windows

Для Windows рекомендуется использовать установщики с официальных сайтов:
- MySQL: https://dev.mysql.com/downloads/installer/
- MariaDB: https://mariadb.org/download/

### Настройка безопасности

После установки рекомендуется запустить скрипт настройки безопасности:

```bash
sudo mysql_secure_installation
```

### Создание базы данных и пользователя

Для работы с примерами создадим базу данных и пользователя:

```bash
sudo mysql -u root -p

mysql> CREATE DATABASE rustdb;
mysql> CREATE USER 'rustuser'@'localhost' IDENTIFIED BY 'password';
mysql> GRANT ALL PRIVILEGES ON rustdb.* TO 'rustuser'@'localhost';
mysql> FLUSH PRIVILEGES;
mysql> EXIT;
```

## Синхронное подключение с библиотекой mysql

Библиотека `mysql` предоставляет синхронный API для работы с MySQL/MariaDB.

### Добавление зависимостей

Добавьте в `Cargo.toml`:

```toml
[dependencies]
mysql = "23.0.1"
```

### Базовое подключение

```rust
use mysql::*;
use mysql::prelude::*;

fn main() -> Result<(), mysql::Error> {
    // Создаем опции подключения
    let opts = OptsBuilder::new()
        .ip_or_hostname(Some("localhost"))
        .user(Some("rustuser"))
        .pass(Some("password"))
        .db_name(Some("rustdb"));
    
    // Подключаемся к базе данных
    let mut conn = Conn::new(opts)?;
    
    // Проверяем подключение простым запросом
    let version: String = conn.query_first("SELECT VERSION()")?
        .unwrap_or_else(|| "Unknown".to_string());
    
    println!("Подключение к MySQL/MariaDB успешно!");
    println!("Версия MySQL/MariaDB: {}", version);
    
    Ok(())
}
```

### Выполнение запросов

```rust
use mysql::*;
use mysql::prelude::*;

fn main() -> Result<(), mysql::Error> {
    let opts = OptsBuilder::new()
        .ip_or_hostname(Some("localhost"))
        .user(Some("rustuser"))
        .pass(Some("password"))
        .db_name(Some("rustdb"));
    
    let mut conn = Conn::new(opts)?;
    
    // Создаем таблицу
    conn.query_drop(
        "CREATE TABLE IF NOT EXISTS users (
            id INT AUTO_INCREMENT PRIMARY KEY,
            name VARCHAR(100) NOT NULL,
            email VARCHAR(100) NOT NULL UNIQUE,
            active BOOLEAN NOT NULL DEFAULT TRUE
        )"
    )?;
    
    // Вставляем данные
    conn.exec_drop(
        "INSERT INTO users (name, email) VALUES (?, ?)
         ON DUPLICATE KEY UPDATE name = VALUES(name)",
        ("Иван Иванов", "ivan@example.com")
    )?;
    
    // Выполняем запрос
    let selected_users = conn.query_map(
### Использование транзакций

```rust
use mysql::*;
use mysql::prelude::*;

fn main() -> Result<(), mysql::Error> {
    let opts = OptsBuilder::new()
        .ip_or_hostname(Some("localhost"))
        .user(Some("rustuser"))
        .pass(Some("password"))
        .db_name(Some("rustdb"));
    
    let mut conn = Conn::new(opts)?;
    
    // Начинаем транзакцию
    let mut tx = conn.start_transaction(TxOpts::default())?;
    
    // Выполняем операции в рамках транзакции
    tx.exec_drop(
        "INSERT INTO users (name, email) VALUES (?, ?)",
        ("Петр Петров", "petr@example.com")
    )?;
    
    tx.exec_drop(
        "UPDATE users SET active = ? WHERE email = ?",
        (false, "ivan@example.com")
    )?;
    
    // Фиксируем транзакцию
    tx.commit()?;
    
    println!("Транзакция успешно выполнена");
    
    Ok(())
}
```

### Подготовленные выражения

```rust
use mysql::*;
use mysql::prelude::*;

fn main() -> Result<(), mysql::Error> {
    let opts = OptsBuilder::new()
        .ip_or_hostname(Some("localhost"))
        .user(Some("rustuser"))
        .pass(Some("password"))
        .db_name(Some("rustdb"));
    
    let mut conn = Conn::new(opts)?;
    
    // Подготавливаем выражение
    let mut stmt = conn.prepare(
        "INSERT INTO users (name, email) VALUES (?, ?)"
    )?;
    
    // Выполняем подготовленное выражение несколько раз
    stmt.execute(("Алексей Алексеев", "alex@example.com"))?;
    stmt.execute(("Мария Иванова", "maria@example.com"))?;
    stmt.execute(("Сергей Сидоров", "sergey@example.com"))?;
    
    println!("Данные успешно вставлены");
    
    Ok(())
}
```

### Пакетная вставка данных

```rust
use mysql::*;
use mysql::prelude::*;

fn main() -> Result<(), mysql::Error> {
    let opts = OptsBuilder::new()
        .ip_or_hostname(Some("localhost"))
        .user(Some("rustuser"))
        .pass(Some("password"))
        .db_name(Some("rustdb"));
    
    let mut conn = Conn::new(opts)?;
    
    // Подготавливаем данные для пакетной вставки
    let users = vec![
        ("Анна Смирнова", "anna@example.com"),
        ("Дмитрий Козлов", "dmitry@example.com"),
        ("Елена Петрова", "elena@example.com"),
    ];
    
    // Выполняем пакетную вставку
    conn.exec_batch(
        "INSERT INTO users (name, email) VALUES (?, ?)
         ON DUPLICATE KEY UPDATE name = VALUES(name)",
        users.iter().map(|user| (user.0, user.1))
    )?;
    
    println!("Пакетная вставка успешно выполнена");
    
    Ok(())
}
```

## Пул соединений с r2d2-mysql

Для эффективной работы с базой данных в многопоточных приложениях рекомендуется использовать пул соединений.

### Добавление зависимостей

```toml
[dependencies]
mysql = "23.0.1"
r2d2 = "0.8"
r2d2_mysql = "23.0.0"
```

### Создание и использование пула

```rust
use mysql::*;
use mysql::prelude::*;
use r2d2_mysql::MysqlConnectionManager;
use r2d2::Pool;

fn main() -> Result<(), Box<dyn std::error::Error>> {
    // Создаем опции подключения
    let opts = OptsBuilder::new()
        .ip_or_hostname(Some("localhost"))
        .user(Some("rustuser"))
        .pass(Some("password"))
        .db_name(Some("rustdb"));
    
    // Создаем менеджер соединений
    let manager = MysqlConnectionManager::new(opts);
    
    // Создаем пул с максимум 10 соединениями
    let pool = Pool::builder()
        .max_size(10)
        .build(manager)?;
    
    // Получаем соединение из пула
    let mut conn = pool.get()?;
    
    // Используем соединение
    let version: String = conn.query_first("SELECT VERSION()")?
        .unwrap_or_else(|| "Unknown".to_string());
    
    println!("Подключение через пул успешно!");
## Асинхронное подключение с mysql_async

Для асинхронной работы с MySQL/MariaDB можно использовать библиотеку `mysql_async`.

### Добавление зависимостей

```toml
[dependencies]
mysql_async = "0.31.3"
tokio = { version = "1", features = ["full"] }
```

### Базовое асинхронное подключение

```rust
use mysql_async::*;
use mysql_async::prelude::*;

#[tokio::main]
async fn main() -> Result<(), mysql_async::Error> {
    // Создаем опции подключения
    let opts = OptsBuilder::new()
        .ip_or_hostname(Some("localhost"))
        .user(Some("rustuser"))
        .pass(Some("password"))
        .db_name(Some("rustdb"));
    
    // Создаем пул соединений
    let pool = Pool::new(opts);
    
    // Получаем соединение из пула
    let mut conn = pool.get_conn().await?;
    
    // Выполняем запрос асинхронно
    let version: Option<String> = conn.query_first("SELECT VERSION()").await?;
    
    println!("Асинхронное подключение успешно!");
    println!("Версия MySQL/MariaDB: {}", version.unwrap_or_else(|| "Unknown".to_string()));
    
    // Закрываем пул
    pool.disconnect().await?;
    
    Ok(())
}
```

### Асинхронное выполнение запросов

```rust
use mysql_async::*;
use mysql_async::prelude::*;

#[tokio::main]
async fn main() -> Result<(), mysql_async::Error> {
    let opts = OptsBuilder::new()
        .ip_or_hostname(Some("localhost"))
        .user(Some("rustuser"))
        .pass(Some("password"))
        .db_name(Some("rustdb"));
    
    let pool = Pool::new(opts);
    let mut conn = pool.get_conn().await?;
    
    // Создаем таблицу
    conn.query_drop(
        "CREATE TABLE IF NOT EXISTS async_users (
            id INT AUTO_INCREMENT PRIMARY KEY,
            name VARCHAR(100) NOT NULL,
            email VARCHAR(100) NOT NULL UNIQUE
        )"
    ).await?;
    
    // Вставляем данные
    conn.exec_drop(
        "INSERT INTO async_users (name, email) VALUES (?, ?)
         ON DUPLICATE KEY UPDATE name = VALUES(name)",
        ("Анна Смирнова", "anna@example.com")
    ).await?;
    
    // Выполняем запрос
    let users = conn.exec_map(
        "SELECT id, name, email FROM async_users",
        (),
        |(id, name, email)| {
            AsyncUser { id, name, email }
        }
    ).await?;
    
    // Выводим результаты
    println!("Найдено {} пользователей:", users.len());
    for user in users {
        println!("#{}: {} ({})", user.id, user.name, user.email);
    }
    
    // Закрываем пул
    pool.disconnect().await?;
    
    Ok(())
}

// Структура для хранения данных пользователя
struct AsyncUser {
    id: i32,
    name: String,
    email: String,
}
```

### Асинхронные транзакции

```rust
use mysql_async::*;
use mysql_async::prelude::*;

#[tokio::main]
async fn main() -> Result<(), mysql_async::Error> {
    let opts = OptsBuilder::new()
        .ip_or_hostname(Some("localhost"))
        .user(Some("rustuser"))
        .pass(Some("password"))
        .db_name(Some("rustdb"));
    
    let pool = Pool::new(opts);
    let mut conn = pool.get_conn().await?;
    
    // Начинаем транзакцию
    let mut tx = conn.start_transaction(TxOpts::default()).await?;
    
    // Выполняем операции в рамках транзакции
    tx.exec_drop(
        "INSERT INTO async_users (name, email) VALUES (?, ?)",
        ("Дмитрий Козлов", "dmitry@example.com")
    ).await?;
    
    tx.exec_drop(
        "INSERT INTO async_users (name, email) VALUES (?, ?)",
        ("Елена Петрова", "elena@example.com")
    ).await?;
    
    // Фиксируем транзакцию
    tx.commit().await?;
    
    println!("Асинхронная транзакция успешно выполнена");
    
    // Закрываем пул
    pool.disconnect().await?;
    
    Ok(())
}
```

### Пакетная вставка данных асинхронно

```rust
use mysql_async::*;
use mysql_async::prelude::*;

#[tokio::main]
async fn main() -> Result<(), mysql_async::Error> {
    let opts = OptsBuilder::new()
        .ip_or_hostname(Some("localhost"))
        .user(Some("rustuser"))
        .pass(Some("password"))
        .db_name(Some("rustdb"));
    
    let pool = Pool::new(opts);
    let mut conn = pool.get_conn().await?;
    
    // Подготавливаем данные для пакетной вставки
    let users = vec![
        ("Игорь Соколов", "igor@example.com"),
        ("Наталья Морозова", "natalia@example.com"),
        ("Олег Волков", "oleg@example.com"),
    ];
    
    // Выполняем пакетную вставку
    conn.exec_batch(
        "INSERT INTO async_users (name, email) VALUES (?, ?)
         ON DUPLICATE KEY UPDATE name = VALUES(name)",
        users.iter().map(|user| params! {
            "name" => user.0,
            "email" => user.1,
        })
    ).await?;
    
    println!("Асинхронная пакетная вставка успешно выполнена");
    
    // Закрываем пул
    pool.disconnect().await?;
    
    Ok(())
}
```

## Работа с MySQL/MariaDB через SQLx

SQLx — это асинхронная библиотека для работы с SQL базами данных, которая обеспечивает проверку запросов на этапе компиляции.

### Добавление зависимостей

### Проверка запросов на этапе компиляции

Одно из главных преимуществ SQLx — возможность проверки SQL-запросов на этапе компиляции:

```rust
#[tokio::main]
async fn main() -> Result<(), sqlx::Error> {
    let pool = MySqlPoolOptions::new()
        .max_connections(5)
        .connect("mysql://rustuser:password@localhost/rustdb").await?;
    
    // Этот запрос будет проверен на этапе компиляции
    let user = sqlx::query!(
        "SELECT id, name, email FROM sqlx_users WHERE id = ?",
        1
    )
    .fetch_one(&pool)
    .await?;
    
    println!("Найден пользователь: {} ({})", user.name, user.email);
    
    Ok(())
}
```

Для работы проверки на этапе компиляции необходимо настроить переменную окружения `DATABASE_URL` или файл `.env` с URL базы данных.

### Транзакции в SQLx

```rust
use sqlx::mysql::MySqlPoolOptions;

#[tokio::main]
async fn main() -> Result<(), sqlx::Error> {
    let pool = MySqlPoolOptions::new()
        .max_connections(5)
        .connect("mysql://rustuser:password@localhost/rustdb").await?;
    
    // Начинаем транзакцию
    let mut tx = pool.begin().await?;
    
    // Выполняем операции в рамках транзакции
    sqlx::query(
        "INSERT INTO sqlx_users (name, email) VALUES (?, ?)"
    )
    .bind("Игорь Соколов")
    .bind("igor@example.com")
    .execute(&mut tx)
    .await?;
    
    sqlx::query(
        "INSERT INTO sqlx_users (name, email) VALUES (?, ?)"
    )
    .bind("Наталья Морозова")
    .bind("natalia@example.com")
    .execute(&mut tx)
    .await?;
    
    // Фиксируем транзакцию
    tx.commit().await?;
    
    println!("Транзакция успешно выполнена");
    
    Ok(())
}
```

## Работа с MySQL/MariaDB через Diesel

Diesel — это ORM и Query Builder для Rust, который обеспечивает безопасность типов и проверку на этапе компиляции.

### Добавление зависимостей

```toml
[dependencies]
diesel = { version = "2.0.0", features = ["mysql", "chrono"] }
dotenvy = "0.15"
chrono = "0.4"
```

### Настройка схемы и моделей

```rust
// src/schema.rs
table! {
    diesel_users (id) {
        id -> Integer,
        name -> Text,
        email -> Text,
        created_at -> Timestamp,
    }
}
```

```rust
// src/models.rs
use diesel::prelude::*;
use chrono::NaiveDateTime;

#[derive(Queryable, Debug)]
pub struct User {
    pub id: i32,
    pub name: String,
    pub email: String,
    pub created_at: NaiveDateTime,
}

#[derive(Insertable)]
#[diesel(table_name = diesel_users)]
pub struct NewUser<'a> {
    pub name: &'a str,
    pub email: &'a str,
}
```

### Подключение и выполнение операций

```rust
use diesel::prelude::*;
use diesel::mysql::MysqlConnection;
use dotenvy::dotenv;
use std::env;

// Импортируем схему и модели
mod schema;
mod models;

use models::{User, NewUser};
use schema::diesel_users;

fn main() {
    dotenv().ok();
    
    let database_url = env::var("DATABASE_URL")
        .expect("DATABASE_URL должна быть установлена");
    
    let mut conn = MysqlConnection::establish(&database_url)
        .expect("Ошибка подключения к базе данных");
    
    // Создаем таблицу (обычно это делается через миграции)
    diesel::sql_query(
        "CREATE TABLE IF NOT EXISTS diesel_users (
            id INTEGER PRIMARY KEY AUTO_INCREMENT,
            name TEXT NOT NULL,
            email TEXT NOT NULL UNIQUE,
            created_at TIMESTAMP NOT NULL DEFAULT CURRENT_TIMESTAMP
        )"
    ).execute(&mut conn).expect("Ошибка создания таблицы");
    
    // Вставляем данные
    let new_user = NewUser {
        name: "Андрей Николаев",
        email: "andrey@example.com",
    };
    
    let user: User = diesel::insert_into(diesel_users::table)
        .values(&new_user)
        .get_result(&mut conn)
        .expect("Ошибка при вставке пользователя");
    
    println!("Добавлен новый пользователь: {:?}", user);
    
    // Выполняем запрос
    let users = diesel_users::table
        .limit(5)
        .load::<User>(&mut conn)
        .expect("Ошибка при загрузке пользователей");
    
    println!("Найдено {} пользователей:", users.len());
    for user in users {
        println!("#{}: {} ({})", user.id, user.name, user.email);
    }
}
```

## Особенности работы с MariaDB

MariaDB в основном совместима с MySQL, поэтому большинство примеров, приведенных выше, будут работать и с MariaDB. Однако есть некоторые особенности и отличия:

### Версионирование

При подключении к MariaDB может быть полезно проверить, с какой СУБД вы работаете:

```rust
use mysql::*;
use mysql::prelude::*;

fn main() -> Result<(), mysql::Error> {
    let opts = OptsBuilder::new()
        .ip_or_hostname(Some("localhost"))
        .user(Some("rustuser"))
        .pass(Some("password"))
        .db_name(Some("rustdb"));
    
    let mut conn = Conn::new(opts)?;
    
    // Проверяем версию и тип СУБД
    let version: String = conn.query_first("SELECT VERSION()")?
        .unwrap_or_else(|| "Unknown".to_string());
    
    if version.to_lowercase().contains("mariadb") {
        println!("Подключено к MariaDB: {}", version);
    } else {
        println!("Подключено к MySQL: {}", version);
    }
    
    Ok(())
}
```

### Специфичные функции MariaDB

MariaDB имеет некоторые функции, которых нет в MySQL:

```rust
use mysql::*;
use mysql::prelude::*;

fn main() -> Result<(), mysql::Error> {
    let opts = OptsBuilder::new()
        .ip_or_hostname(Some("localhost"))
        .user(Some("rustuser"))
        .pass(Some("password"))
        .db_name(Some("rustdb"));
    
    let mut conn = Conn::new(opts)?;
    
    // Используем функцию, специфичную для MariaDB
    let result: Option<String> = conn.query_first(
        "SELECT REGEXP_REPLACE('Hello, world!', 'world', 'MariaDB')"
    )?;
    
    if let Some(text) = result {
        println!("Результат: {}", text); // Выведет "Hello, MariaDB!"
    } else {
        println!("Функция не поддерживается (возможно, это MySQL)");
    }
    
    Ok(())
}
```

## Заключение

В этой главе мы рассмотрели различные способы подключения к MySQL и MariaDB из Rust-приложений. Мы изучили как синхронные, так и асинхронные подходы, использование пулов соединений и основы работы с ORM-библиотеками. Эти знания позволят вам эффективно работать с MySQL/MariaDB в ваших Rust-проектах.

MySQL и MariaDB являются мощными и широко используемыми СУБД, которые хорошо подходят для многих типов приложений, особенно для веб-разработки. Rust, благодаря своей производительности и безопасности, является отличным выбором для создания надежных приложений, работающих с этими базами данных.
```toml
[dependencies]
sqlx = { version = "0.6", features = ["runtime-tokio-rustls", "mysql", "macros"] }
tokio = { version = "1", features = ["full"] }
```

### Базовое подключение и запросы

```rust
use sqlx::mysql::MySqlPoolOptions;

#[tokio::main]
async fn main() -> Result<(), sqlx::Error> {
    // Создаем пул соединений
    let pool = MySqlPoolOptions::new()
        .max_connections(5)
        .connect("mysql://rustuser:password@localhost/rustdb").await?;
    
    // Создаем таблицу
    sqlx::query(
        "CREATE TABLE IF NOT EXISTS sqlx_users (
            id INT AUTO_INCREMENT PRIMARY KEY,
            name TEXT NOT NULL,
            email TEXT NOT NULL UNIQUE,
            created_at TIMESTAMP NOT NULL DEFAULT CURRENT_TIMESTAMP
        )"
    )
    .execute(&pool)
    .await?;
    
    // Вставляем данные
    let result = sqlx::query(
        "INSERT INTO sqlx_users (name, email) VALUES (?, ?) RETURNING id"
    )
    .bind("Ольга Кузнецова")
    .bind("olga@example.com")
    .fetch_one(&pool)
    .await?;
    
    let id: i32 = result.get(0);
    println!("Вставлена запись с id: {}", id);
    
    // Выполняем запрос с использованием макроса query_as
    #[derive(Debug)]
    struct User {
        id: i32,
        name: String,
        email: String,
        created_at: chrono::NaiveDateTime,
    }
    
    let users = sqlx::query_as!(
        User,
        "SELECT id, name, email, created_at FROM sqlx_users"
    )
    .fetch_all(&pool)
    .await?;
    
    for user in users {
        println!("Пользователь: {:?}", user);
    }
    
    Ok(())
}
```

### Использование пула в многопоточном приложении

```rust
use mysql::*;
use mysql::prelude::*;
use r2d2_mysql::MysqlConnectionManager;
use r2d2::Pool;
use std::thread;

fn main() -> Result<(), Box<dyn std::error::Error>> {
    // Создаем пул соединений
    let opts = OptsBuilder::new()
        .ip_or_hostname(Some("localhost"))
        .user(Some("rustuser"))
        .pass(Some("password"))
        .db_name(Some("rustdb"));
    
    let manager = MysqlConnectionManager::new(opts);
    let pool = Pool::builder()
        .max_size(10)
        .build(manager)?;
    
    // Создаем вектор для хранения дескрипторов потоков
    let mut handles = vec![];
    
    // Запускаем 5 потоков, каждый из которых использует соединение из пула
    for i in 0..5 {
        // Клонируем пул для использования в потоке
        let pool_clone = pool.clone();
        
        // Создаем новый поток
        let handle = thread::spawn(move || {
            // Получаем соединение из пула
            let mut conn = pool_clone.get().unwrap();
            
            // Выполняем запрос
            let result: Option<i32> = conn.exec_first(
                "SELECT SLEEP(1), ?",
                (i,)
            ).unwrap();
            
            println!("Поток {} выполнил запрос, результат: {:?}", i, result);
        });
        
        handles.push(handle);
    }
    
    // Ожидаем завершения всех потоков
    for handle in handles {
        handle.join().unwrap();
    }
    
    println!("Все потоки завершены");
    
    Ok(())
}
```
