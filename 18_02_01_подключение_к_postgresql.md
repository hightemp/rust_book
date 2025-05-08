# Подключение к PostgreSQL в Rust

## Введение

PostgreSQL — одна из самых мощных, надежных и функциональных реляционных систем управления базами данных с открытым исходным кодом. Она поддерживает широкий спектр типов данных, имеет богатый набор функций и расширений, а также обеспечивает высокую производительность и масштабируемость.

В этой главе мы рассмотрим различные способы подключения к PostgreSQL из Rust-приложений, включая синхронные и асинхронные подходы, использование пулов соединений и ORM-библиотек.

## Основные библиотеки для работы с PostgreSQL в Rust

Для работы с PostgreSQL в Rust существует несколько основных библиотек:

1. **postgres** - нативный драйвер для синхронного подключения
2. **tokio-postgres** - асинхронная версия драйвера postgres
3. **r2d2-postgres** - пул соединений для postgres
4. **sqlx** - асинхронная библиотека с поддержкой PostgreSQL
5. **diesel** - ORM с поддержкой PostgreSQL
6. **SeaORM** - асинхронный ORM с поддержкой PostgreSQL

## Установка PostgreSQL

Прежде чем начать работу с PostgreSQL в Rust, необходимо установить и настроить сервер PostgreSQL.

### Установка на Linux (Ubuntu/Debian)

```bash
sudo apt update
sudo apt install postgresql postgresql-contrib
```

### Установка на macOS

```bash
brew install postgresql
```

### Установка на Windows

Для Windows рекомендуется использовать установщик с официального сайта PostgreSQL: https://www.postgresql.org/download/windows/

### Проверка установки

После установки можно проверить, что PostgreSQL работает:

```bash
sudo systemctl status postgresql  # для Linux с systemd
# или
pg_isready  # для проверки доступности сервера
```

### Создание базы данных и пользователя

Для работы с примерами создадим базу данных и пользователя:

```bash
sudo -u postgres psql

postgres=# CREATE USER rustuser WITH PASSWORD 'password';
postgres=# CREATE DATABASE rustdb OWNER rustuser;
postgres=# GRANT ALL PRIVILEGES ON DATABASE rustdb TO rustuser;
postgres=# \q
```

## Синхронное подключение с библиотекой postgres

Библиотека `postgres` предоставляет синхронный API для работы с PostgreSQL.

### Добавление зависимостей

Добавьте в `Cargo.toml`:

```toml
[dependencies]
postgres = "0.19"
```

### Базовое подключение

```rust
use postgres::{Client, NoTls, Error};

fn main() -> Result<(), Error> {
    // Строка подключения содержит все необходимые параметры
    let conn_string = "host=localhost user=rustuser password=password dbname=rustdb";
    
    // Создаем клиента и подключаемся к базе данных
    let mut client = Client::connect(conn_string, NoTls)?;
    
    // Проверяем подключение простым запросом
    let rows = client.query("SELECT version()", &[])?;
    let version: String = rows[0].get(0);
    
    println!("Подключение к PostgreSQL успешно!");
    println!("Версия PostgreSQL: {}", version);
    
    Ok(())
}
```

### Выполнение запросов

```rust
use postgres::{Client, NoTls, Error};

fn main() -> Result<(), Error> {
    let mut client = Client::connect(
        "host=localhost user=rustuser password=password dbname=rustdb",
        NoTls,
    )?;
    
    // Создаем таблицу
    client.execute(
        "CREATE TABLE IF NOT EXISTS users (
            id SERIAL PRIMARY KEY,
            name VARCHAR NOT NULL,
            email VARCHAR NOT NULL UNIQUE,
            active BOOLEAN NOT NULL DEFAULT TRUE
        )",
        &[],
    )?;
    
    // Вставляем данные
    let rows_affected = client.execute(
        "INSERT INTO users (name, email) VALUES ($1, $2) 
         ON CONFLICT (email) DO NOTHING",
        &[&"Иван Иванов", &"ivan@example.com"],
    )?;
    
    println!("Вставлено строк: {}", rows_affected);
    
    // Выполняем запрос
    let rows = client.query(
        "SELECT id, name, email, active FROM users WHERE active = $1",
        &[&true],
    )?;
    
    // Обрабатываем результаты
    for row in rows {
        let id: i32 = row.get(0);
        let name: &str = row.get(1);
        let email: &str = row.get(2);
        let active: bool = row.get(3);
        
        println!("Пользователь #{}: {} ({}) - активен: {}", 
                 id, name, email, active);
    }
    
    Ok(())
}
```

### Использование транзакций

```rust
use postgres::{Client, NoTls, Error};

fn main() -> Result<(), Error> {
    let mut client = Client::connect(
        "host=localhost user=rustuser password=password dbname=rustdb",
        NoTls,
    )?;
    
    // Начинаем транзакцию
    let transaction = client.transaction()?;
    
    // Выполняем операции в рамках транзакции
    transaction.execute(
        "INSERT INTO users (name, email) VALUES ($1, $2)",
        &[&"Петр Петров", &"petr@example.com"],
    )?;
    
    transaction.execute(
        "UPDATE users SET active = $1 WHERE email = $2",
        &[&false, &"ivan@example.com"],
    )?;
    
    // Фиксируем транзакцию
    transaction.commit()?;
    
    println!("Транзакция успешно выполнена");
    
    Ok(())
}
```

### Подготовленные выражения

```rust
use postgres::{Client, NoTls, Error};

fn main() -> Result<(), Error> {
    let mut client = Client::connect(
        "host=localhost user=rustuser password=password dbname=rustdb",
        NoTls,
    )?;
    
    // Подготавливаем выражение
    let stmt = client.prepare("INSERT INTO users (name, email) VALUES ($1, $2)")?;
    
    // Выполняем подготовленное выражение несколько раз
    client.execute(&stmt, &[&"Алексей Алексеев", &"alex@example.com"])?;
    client.execute(&stmt, &[&"Мария Иванова", &"maria@example.com"])?;
    client.execute(&stmt, &[&"Сергей Сидоров", &"sergey@example.com"])?;
    
    println!("Данные успешно вставлены");
    
    Ok(())
}
```

## Пул соединений с r2d2

Для эффективной работы с базой данных в многопоточных приложениях рекомендуется использовать пул соединений.

### Добавление зависимостей

```toml
[dependencies]
postgres = "0.19"
r2d2 = "0.8"
r2d2_postgres = "0.18"
```

### Создание и использование пула

```rust
use r2d2_postgres::{postgres::NoTls, PostgresConnectionManager};
use r2d2::Pool;

fn main() -> Result<(), Box<dyn std::error::Error>> {
    // Создаем менеджер соединений
    let manager = PostgresConnectionManager::new(
        "host=localhost user=rustuser password=password dbname=rustdb".parse()?,
        NoTls,
    );
    
    // Создаем пул с максимум 10 соединениями
    let pool = Pool::builder()
        .max_size(10)
        .build(manager)?;
    
    // Получаем соединение из пула
    let client = pool.get()?;
    
    // Используем соединение
    let rows = client.query("SELECT version()", &[])?;
    let version: String = rows[0].get(0);
    
    println!("Подключение через пул успешно!");
    println!("Версия PostgreSQL: {}", version);
    
    // Соединение автоматически возвращается в пул при выходе из области видимости
    
    Ok(())
}
```

### Использование пула в многопоточном приложении

```rust
use r2d2_postgres::{postgres::NoTls, PostgresConnectionManager};
use r2d2::Pool;
use std::thread;

fn main() -> Result<(), Box<dyn std::error::Error>> {
    // Создаем пул соединений
    let manager = PostgresConnectionManager::new(
        "host=localhost user=rustuser password=password dbname=rustdb".parse()?,
        NoTls,
    );
    
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
            let conn = pool_clone.get().unwrap();
            
            // Выполняем запрос
            let rows = conn.query("SELECT pg_sleep(1), $1::INT", &[&i]).unwrap();
            let value: i32 = rows[0].get(1);
            
            println!("Поток {} выполнил запрос, результат: {}", i, value);
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

## Асинхронное подключение с tokio-postgres

Для асинхронной работы с PostgreSQL можно использовать библиотеку `tokio-postgres`.

### Добавление зависимостей

```toml
[dependencies]
tokio = { version = "1", features = ["full"] }
tokio-postgres = "0.7"
```

### Базовое асинхронное подключение

```rust
use tokio_postgres::{NoTls, Error};

#[tokio::main]
async fn main() -> Result<(), Error> {
    // Подключаемся к базе данных
    let (client, connection) = tokio_postgres::connect(
        "host=localhost user=rustuser password=password dbname=rustdb", 
        NoTls,
    ).await?;
    
    // Запускаем обработку соединения в отдельной задаче
    tokio::spawn(async move {
        if let Err(e) = connection.await {
            eprintln!("Ошибка соединения: {}", e);
        }
    });
    
    // Выполняем запрос асинхронно
    let rows = client
        .query("SELECT version()", &[])
        .await?;
    
    let version: String = rows[0].get(0);
    println!("Асинхронное подключение успешно!");
    println!("Версия PostgreSQL: {}", version);
    
    Ok(())
}
```

### Асинхронное выполнение запросов

```rust
use tokio_postgres::{NoTls, Error};

#[tokio::main]
async fn main() -> Result<(), Error> {
    let (client, connection) = tokio_postgres::connect(
        "host=localhost user=rustuser password=password dbname=rustdb", 
        NoTls,
    ).await?;
    
    tokio::spawn(async move {
        if let Err(e) = connection.await {
            eprintln!("Ошибка соединения: {}", e);
        }
    });
    
    // Создаем таблицу
    client.execute(
        "CREATE TABLE IF NOT EXISTS async_users (
            id SERIAL PRIMARY KEY,
            name VARCHAR NOT NULL,
            email VARCHAR NOT NULL UNIQUE
        )",
        &[],
    ).await?;
    
    // Вставляем данные
    let rows_affected = client.execute(
        "INSERT INTO async_users (name, email) VALUES ($1, $2) 
         ON CONFLICT (email) DO NOTHING",
        &[&"Анна Смирнова", &"anna@example.com"],
    ).await?;
    
    println!("Вставлено строк: {}", rows_affected);
    
    // Выполняем запрос
    let rows = client.query(
        "SELECT id, name, email FROM async_users",
        &[],
    ).await?;
    
    // Обрабатываем результаты
    for row in rows {
        let id: i32 = row.get(0);
        let name: &str = row.get(1);
        let email: &str = row.get(2);
        
        println!("Пользователь #{}: {} ({})", id, name, email);
    }
    
    Ok(())
}
```

### Асинхронные транзакции

```rust
use tokio_postgres::{NoTls, Error};

#[tokio::main]
async fn main() -> Result<(), Error> {
    let (client, connection) = tokio_postgres::connect(
        "host=localhost user=rustuser password=password dbname=rustdb", 
        NoTls,
    ).await?;
    
    tokio::spawn(async move {
        if let Err(e) = connection.await {
            eprintln!("Ошибка соединения: {}", e);
        }
    });
    
    // Начинаем транзакцию
    let transaction = client.transaction().await?;
    
    // Выполняем операции в рамках транзакции
    transaction.execute(
        "INSERT INTO async_users (name, email) VALUES ($1, $2)",
        &[&"Дмитрий Козлов", &"dmitry@example.com"],
    ).await?;
    
    transaction.execute(
        "INSERT INTO async_users (name, email) VALUES ($1, $2)",
        &[&"Елена Петрова", &"elena@example.com"],
    ).await?;
    
    // Фиксируем транзакцию
    transaction.commit().await?;
    
    println!("Асинхронная транзакция успешно выполнена");
    
    Ok(())
}
```

## Асинхронный пул соединений с deadpool

Для асинхронных приложений можно использовать `deadpool-postgres` для управления пулом соединений.

### Добавление зависимостей

```toml
[dependencies]
tokio = { version = "1", features = ["full"] }
deadpool-postgres = "0.10"
tokio-postgres = "0.7"
```

### Создание и использование асинхронного пула

```rust
use deadpool_postgres::{Config, Pool, Runtime};
use tokio_postgres::NoTls;

#[tokio::main]
async fn main() -> Result<(), Box<dyn std::error::Error>> {
    // Настраиваем конфигурацию
    let mut cfg = Config::new();
    cfg.host = Some("localhost".to_string());
    cfg.user = Some("rustuser".to_string());
    cfg.password = Some("password".to_string());
    cfg.dbname = Some("rustdb".to_string());
    
    // Создаем пул
    let pool = cfg.create_pool(Some(Runtime::Tokio1), NoTls)?;
    
    // Получаем клиента из пула
    let client = pool.get().await?;
    
    // Используем клиента
    let rows = client.query("SELECT version()", &[]).await?;
    let version: String = rows[0].get(0);
    
    println!("Асинхронное подключение через пул успешно!");
    println!("Версия PostgreSQL: {}", version);
    
    Ok(())
}
```

### Использование асинхронного пула в параллельных задачах

```rust
use deadpool_postgres::{Config, Pool, Runtime};
use tokio_postgres::NoTls;
use futures::future;

#[tokio::main]
async fn main() -> Result<(), Box<dyn std::error::Error>> {
    // Создаем пул
    let mut cfg = Config::new();
    cfg.host = Some("localhost".to_string());
    cfg.user = Some("rustuser".to_string());
    cfg.password = Some("password".to_string());
    cfg.dbname = Some("rustdb".to_string());
    cfg.pool = Some(deadpool_postgres::PoolConfig::new(10));
    
    let pool = cfg.create_pool(Some(Runtime::Tokio1), NoTls)?;
    
    // Создаем вектор задач
    let mut tasks = Vec::new();
    
    // Запускаем 5 асинхронных задач
    for i in 0..5 {
        let pool_clone = pool.clone();
        
        let task = tokio::spawn(async move {
            // Получаем клиента из пула
            let client = pool_clone.get().await.unwrap();
            
            // Выполняем запрос с задержкой
            let rows = client
                .query("SELECT pg_sleep(1), $1::INT", &[&i])
                .await
                .unwrap();
            
            let value: i32 = rows[0].get(1);
            println!("Задача {} выполнила запрос, результат: {}", i, value);
            
            value
        });
        
        tasks.push(task);
    }
    
    // Ожидаем завершения всех задач
    let results = future::join_all(tasks).await;
    
    println!("Все задачи завершены с результатами: {:?}", 
             results.into_iter().map(|r| r.unwrap()).collect::<Vec<_>>());
    
    Ok(())
}
```

## Работа с PostgreSQL через SQLx

SQLx — это асинхронная библиотека для работы с SQL базами данных, которая обеспечивает проверку запросов на этапе компиляции.

### Добавление зависимостей

```toml
[dependencies]
sqlx = { version = "0.6", features = ["runtime-tokio-rustls", "postgres", "macros"] }
tokio = { version = "1", features = ["full"] }
```

### Базовое подключение и запросы

```rust
use sqlx::postgres::PgPoolOptions;

#[tokio::main]
async fn main() -> Result<(), sqlx::Error> {
    // Создаем пул соединений
    let pool = PgPoolOptions::new()
        .max_connections(5)
        .connect("postgres://rustuser:password@localhost/rustdb").await?;
    
    // Создаем таблицу
    sqlx::query(
        "CREATE TABLE IF NOT EXISTS sqlx_users (
            id SERIAL PRIMARY KEY,
            name TEXT NOT NULL,
            email TEXT NOT NULL UNIQUE,
            created_at TIMESTAMPTZ NOT NULL DEFAULT NOW()
        )"
    )
    .execute(&pool)
    .await?;
    
    // Вставляем данные
    let result = sqlx::query(
        "INSERT INTO sqlx_users (name, email) VALUES ($1, $2) RETURNING id"
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
        created_at: chrono::DateTime<chrono::Utc>,
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

### Заключение

В этой главе мы рассмотрели различные способы подключения к PostgreSQL из Rust-приложений. Мы изучили как синхронные, так и асинхронные подходы, использование пулов соединений и основы работы с ORM-библиотеками. Эти знания позволят вам эффективно работать с PostgreSQL в ваших Rust-проектах.