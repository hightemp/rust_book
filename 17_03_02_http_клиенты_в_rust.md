# HTTP-клиенты в Rust

В Rust существует несколько библиотек для создания HTTP-клиентов, каждая из которых имеет свои особенности и преимущества. В этой главе мы рассмотрим наиболее популярные библиотеки и их применение.

## Содержание
- [Обзор HTTP-клиентских библиотек](#обзор-http-клиентских-библиотек)
- [Библиотека reqwest](#библиотека-reqwest)
- [Библиотека ureq](#библиотека-ureq)
- [Библиотека hyper](#библиотека-hyper)
- [Обработка ответов](#обработка-ответов)
- [Работа с заголовками и cookies](#работа-с-заголовками-и-cookies)
- [Отправка форм и JSON](#отправка-форм-и-json)
- [Асинхронные HTTP-запросы](#асинхронные-http-запросы)
- [Сравнение библиотек](#сравнение-библиотек)

## Обзор HTTP-клиентских библиотек

В экосистеме Rust существует несколько HTTP-клиентских библиотек, каждая из которых имеет свои особенности:

| Библиотека | Описание | Особенности |
|------------|----------|-------------|
| **reqwest** | Высокоуровневый HTTP-клиент | Простой API, поддержка async/await, JSON, cookies |
| **ureq** | Простой синхронный HTTP-клиент | Минимум зависимостей, блокирующий API |
| **hyper** | Низкоуровневый HTTP-клиент/сервер | Высокая производительность, гибкость, async/await |
| **surf** | Асинхронный HTTP-клиент | Простой API, middleware, кроссплатформенность |
| **isahc** | HTTP-клиент на основе libcurl | Высокая производительность, поддержка HTTP/2 |

## Библиотека reqwest

`reqwest` - одна из самых популярных HTTP-клиентских библиотек в Rust. Она предоставляет удобный API для выполнения HTTP-запросов и поддерживает как синхронный, так и асинхронный режимы работы.

### Установка

Добавьте зависимость в `Cargo.toml`:

```toml
[dependencies]
reqwest = { version = "0.11", features = ["json", "blocking"] }
serde = { version = "1.0", features = ["derive"] }
serde_json = "1.0"
tokio = { version = "1", features = ["full"] } # для асинхронного API
```

### Блокирующие запросы

Для простых блокирующих запросов используйте модуль `reqwest::blocking`:

```rust
use reqwest::blocking::Client;
use reqwest::Error;

fn main() -> Result<(), Error> {
    // Создаем клиент
    let client = Client::new();
    
    // Выполняем GET-запрос
    let response = client.get("https://api.github.com/repos/rust-lang/rust")
        .header("User-Agent", "Rust HTTP Client")
        .send()?;
    
    // Проверяем статус
    println!("Статус: {}", response.status());
    
    // Получаем тело ответа как текст
    let body = response.text()?;
    println!("Тело ответа: {}", body);
    
    Ok(())
}
```

### Асинхронные запросы

Для асинхронных запросов используйте основной API `reqwest`:

```rust
use reqwest::Client;
use serde::{Deserialize, Serialize};

#[derive(Debug, Deserialize)]
struct Repository {
    name: String,
    description: Option<String>,
    stargazers_count: u32,
    forks_count: u32,
}

#[tokio::main]
async fn main() -> Result<(), Box<dyn std::error::Error>> {
    // Создаем клиент
    let client = Client::new();
    
    // Выполняем GET-запрос
    let response = client.get("https://api.github.com/repos/rust-lang/rust")
        .header("User-Agent", "Rust HTTP Client")
        .send()
        .await?;
    
    // Десериализуем JSON-ответ
    let repo: Repository = response.json().await?;
    
    println!("Репозиторий: {}", repo.name);
    println!("Описание: {}", repo.description.unwrap_or_default());
    println!("Звезды: {}", repo.stargazers_count);
    println!("Форки: {}", repo.forks_count);
    
    Ok(())
}
```

### Основные возможности reqwest

- **Методы HTTP**: GET, POST, PUT, DELETE, HEAD, OPTIONS, PATCH
- **Заголовки**: установка пользовательских заголовков
- **Cookies**: автоматическое управление cookies
- **Редиректы**: автоматическое следование по редиректам
- **Таймауты**: настройка таймаутов для запросов
- **Прокси**: поддержка HTTP и SOCKS прокси
- **HTTPS**: поддержка TLS/SSL с помощью rustls или native-tls
- **JSON**: сериализация и десериализация JSON
- **Формы**: отправка данных формы
- **Multipart**: отправка multipart/form-data (файлы)
- **Потоковая передача**: потоковая обработка тела ответа

## Библиотека ureq

`ureq` - легковесная HTTP-клиентская библиотека без внешних зависимостей, ориентированная на простоту использования. Она предоставляет только синхронный API.

### Установка

```toml
[dependencies]
ureq = { version = "2.6", features = ["json"] }
serde = { version = "1.0", features = ["derive"] }
serde_json = "1.0"
```

### Простой запрос

```rust
fn main() -> Result<(), ureq::Error> {
    // Выполняем GET-запрос
    let response = ureq::get("https://httpbin.org/get")
        .set("User-Agent", "Rust HTTP Client")
        .call()?;
    
    // Получаем статус и заголовки
    println!("Статус: {}", response.status());
    println!("Сервер: {}", response.header("Server").unwrap_or("Unknown"));
    
    // Получаем тело ответа
    let body = response.into_string()?;
    println!("Тело ответа: {}", body);
    
    Ok(())
}
```

### Работа с JSON

```rust
use serde::{Deserialize, Serialize};

#[derive(Debug, Deserialize)]
struct Repository {
    name: String,
    description: Option<String>,
    stargazers_count: u32,
    forks_count: u32,
}

fn main() -> Result<(), Box<dyn std::error::Error>> {
    // Выполняем GET-запрос
    let response = ureq::get("https://api.github.com/repos/rust-lang/rust")
        .set("User-Agent", "Rust HTTP Client")
        .call()?;
    
    // Десериализуем JSON-ответ
    let repo: Repository = response.into_json()?;
    
    println!("Репозиторий: {}", repo.name);
    println!("Описание: {}", repo.description.unwrap_or_default());
    println!("Звезды: {}", repo.stargazers_count);
    println!("Форки: {}", repo.forks_count);
    
    Ok(())
}
```

### Основные возможности ureq

- **Простой API**: минималистичный и понятный API
- **Минимум зависимостей**: не требует внешних зависимостей
- **Блокирующий API**: синхронные запросы
- **Методы HTTP**: поддержка всех стандартных методов
- **Заголовки**: установка пользовательских заголовков
- **Cookies**: базовая поддержка cookies
- **Редиректы**: автоматическое следование по редиректам
- **Таймауты**: настройка таймаутов для запросов
- **JSON**: сериализация и десериализация JSON
- **Формы**: отправка данных формы

## Библиотека hyper

`hyper` - низкоуровневая HTTP-библиотека, обеспечивающая высокую производительность и гибкость. Она используется как основа для многих других HTTP-библиотек, включая reqwest.

### Установка

```toml
[dependencies]
hyper = { version = "0.14", features = ["full"] }
tokio = { version = "1", features = ["full"] }
hyper-tls = "0.5" # для HTTPS
```

### Асинхронный запрос

```rust
use hyper::{Client, Uri};
use hyper_tls::HttpsConnector;
use hyper::body::HttpBody;
use tokio::io::{stdout, AsyncWriteExt};

#[tokio::main]
async fn main() -> Result<(), Box<dyn std::error::Error>> {
    // Создаем HTTPS-клиент
    let https = HttpsConnector::new();
    let client = Client::builder().build::<_, hyper::Body>(https);
    
    // Парсим URI
    let uri = "https://www.rust-lang.org".parse::<Uri>()?;
    
    // Выполняем запрос
    let mut response = client.get(uri).await?;
    
    println!("Статус: {}", response.status());
    
    // Читаем тело ответа по частям
    while let Some(chunk) = response.body_mut().data().await {
        let chunk = chunk?;
        stdout().write_all(&chunk).await?;
    }
    
    Ok(())
}
```

### Основные возможности hyper

- **Высокая производительность**: оптимизирован для скорости и эффективности
- **Асинхронный API**: полная поддержка async/await
- **Низкоуровневый контроль**: детальный контроль над HTTP-запросами
- **HTTP/1 и HTTP/2**: поддержка обоих версий протокола
- **Клиент и сервер**: API для создания как клиентов, так и серверов
- **Потоковая обработка**: эффективная обработка больших объемов данных

## Обработка ответов

Обработка ответов включает проверку статуса, чтение заголовков и тела ответа:

```rust
use reqwest::blocking::Client;
use reqwest::Error;

fn main() -> Result<(), Error> {
    let client = Client::new();
    let response = client.get("https://httpbin.org/get").send()?;
    
    // Проверяем статус
    if response.status().is_success() {
        println!("Успешный запрос!");
    } else if response.status().is_client_error() {
        println!("Ошибка клиента: {}", response.status());
    } else if response.status().is_server_error() {
        println!("Ошибка сервера: {}", response.status());
    }
    
    // Получаем заголовки
    println!("Тип контента: {}", response.headers().get("content-type")
        .map_or("Unknown", |v| v.to_str().unwrap_or("Invalid")));
    
    // Получаем тело ответа
    let body = response.text()?;
    println!("Тело ответа: {}", body);
    
    Ok(())
}
```

## Работа с заголовками и cookies

### Установка заголовков

```rust
use reqwest::blocking::{Client, ClientBuilder};
use reqwest::header::{HeaderMap, HeaderValue, USER_AGENT, CONTENT_TYPE};

fn main() -> Result<(), Box<dyn std::error::Error>> {
    // Создаем пользовательские заголовки
    let mut headers = HeaderMap::new();
    headers.insert(USER_AGENT, HeaderValue::from_static("My Rust Client"));
    headers.insert(CONTENT_TYPE, HeaderValue::from_static("application/json"));
    
    // Создаем клиент с настройками
    let client = ClientBuilder::new()
        .default_headers(headers)
        .build()?;
    
    // Выполняем запрос с дополнительным заголовком
    let response = client.get("https://httpbin.org/headers")
        .header("X-Custom-Header", "custom value")
        .send()?;
    
    println!("Ответ: {}", response.text()?);
    
    Ok(())
}
```

### Работа с cookies

```rust
use reqwest::blocking::{Client, ClientBuilder};
use std::time::Duration;

fn main() -> Result<(), Box<dyn std::error::Error>> {
    // Создаем клиент с включенным хранилищем cookies
    let client = ClientBuilder::new()
        .cookie_store(true)
        .timeout(Duration::from_secs(30))
        .build()?;
    
    // Первый запрос устанавливает cookie
    let response1 = client.get("https://httpbin.org/cookies/set?name=value").send()?;
    println!("Статус первого запроса: {}", response1.status());
    
    // Второй запрос использует сохраненные cookies
    let response2 = client.get("https://httpbin.org/cookies").send()?;
    println!("Cookies: {}", response2.text()?);
    
    Ok(())
}
```

## Отправка форм и JSON

### Отправка формы

```rust
use reqwest::blocking::Client;
use std::collections::HashMap;

fn main() -> Result<(), Box<dyn std::error::Error>> {
    let client = Client::new();
    
    // Создаем данные формы
    let mut form = HashMap::new();
    form.insert("username", "rust_user");
    form.insert("password", "secure_password");
    
    // Отправляем форму
    let response = client.post("https://httpbin.org/post")
        .form(&form)
        .send()?;
    
    println!("Ответ на форму: {}", response.text()?);
    
    Ok(())
}
```

### Отправка JSON

```rust
use reqwest::blocking::Client;
use serde::{Deserialize, Serialize};

#[derive(Serialize, Deserialize, Debug)]
struct User {
    name: String,
    email: String,
    age: u32,
}

fn main() -> Result<(), Box<dyn std::error::Error>> {
    let client = Client::new();
    
    // Создаем данные для отправки
    let user = User {
        name: String::from("John Doe"),
        email: String::from("john@example.com"),
        age: 30,
    };
    
    // Отправляем JSON
    let response = client.post("https://httpbin.org/post")
        .json(&user)
        .send()?;
    
    println!("Ответ на JSON: {}", response.text()?);
    
    Ok(())
}
```

## Асинхронные HTTP-запросы

Асинхронные запросы позволяют эффективно выполнять несколько запросов одновременно:

```rust
use reqwest::Client;
use serde::{Deserialize, Serialize};

#[derive(Serialize, Deserialize, Debug)]
struct Todo {
    id: u32,
    title: String,
    completed: bool,
}

#[tokio::main]
async fn main() -> Result<(), Box<dyn std::error::Error>> {
    let client = Client::new();
    
    // Параллельные запросы
    let request1 = client.get("https://jsonplaceholder.typicode.com/todos/1");
    let request2 = client.get("https://jsonplaceholder.typicode.com/todos/2");
    
    // Выполняем запросы параллельно
    let (response1, response2) = tokio::join!(
        request1.send(),
        request2.send()
    );
    
    // Обрабатываем результаты
    let todo1: Todo = response1?.json().await?;
    let todo2: Todo = response2?.json().await?;
    
    println!("Todo 1: {:?}", todo1);
    println!("Todo 2: {:?}", todo2);
    
    Ok(())
}
```

### Обработка множества запросов

```rust
use futures::future::join_all;
use reqwest::Client;
use serde::{Deserialize, Serialize};

#[derive(Serialize, Deserialize, Debug)]
struct Todo {
    id: u32,
    title: String,
    completed: bool,
}

#[tokio::main]
async fn main() -> Result<(), Box<dyn std::error::Error>> {
    let client = Client::new();
    
    // Создаем множество запросов
    let mut futures = Vec::new();
    for i in 1..=10 {
        let request = client.get(&format!("https://jsonplaceholder.typicode.com/todos/{}", i))
            .send()
            .then(|res| async {
                match res {
                    Ok(resp) => resp.json::<Todo>().await.ok(),
                    Err(_) => None,
                }
            });
        futures.push(request);
    }
    
    // Выполняем все запросы параллельно
    let results: Vec<Option<Todo>> = join_all(futures).await;
    
    // Обрабатываем результаты
    for (i, todo) in results.iter().enumerate() {
        if let Some(todo) = todo {
            println!("Todo {}: {} (completed: {})", todo.id, todo.title, todo.completed);
        } else {
            println!("Failed to fetch todo {}", i + 1);
        }
    }
    
    Ok(())
}
```

## Сравнение библиотек

| Критерий | reqwest | ureq | hyper |
|----------|---------|------|-------|
| **Уровень абстракции** | Высокий | Средний | Низкий |
| **Простота использования** | Высокая | Высокая | Средняя |
| **Производительность** | Хорошая | Средняя | Отличная |
| **Асинхронность** | Да | Нет | Да |
| **Размер зависимостей** | Большой | Малый | Средний |
| **Поддержка HTTP/2** | Да | Нет | Да |
| **Поддержка JSON** | Да | Да | Нет (требуется дополнительная библиотека) |
| **Поддержка cookies** | Полная | Базовая | Нет (требуется реализация) |

### Когда использовать каждую библиотеку

- **reqwest**: Для большинства приложений, где важна простота использования и богатый функционал
- **ureq**: Для простых приложений, где важна минимизация зависимостей и не требуется асинхронность
- **hyper**: Для высокопроизводительных приложений, где требуется низкоуровневый контроль или создание собственной HTTP-библиотеки

## Заключение

Rust предоставляет несколько отличных библиотек для создания HTTP-клиентов, каждая из которых имеет свои преимущества. Выбор библиотеки зависит от конкретных требований вашего проекта: простоты использования, производительности, размера зависимостей или необходимости асинхронного API.

В следующих разделах мы рассмотрим, как создавать HTTP-серверы в Rust с использованием различных фреймворков.