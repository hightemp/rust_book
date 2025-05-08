# HTTP-серверы в Rust

Rust предлагает несколько мощных библиотек и фреймворков для создания HTTP-серверов. В этой главе мы рассмотрим основные инструменты для разработки веб-серверов и их особенности.

## Содержание
- [Обзор HTTP-серверных фреймворков](#обзор-http-серверных-фреймворков)
- [Простой HTTP-сервер с hyper](#простой-http-сервер-с-hyper)
- [Веб-фреймворк Actix Web](#веб-фреймворк-actix-web)
- [Веб-фреймворк Rocket](#веб-фреймворк-rocket)
- [Веб-фреймворк Warp](#веб-фреймворк-warp)
- [Маршрутизация и обработка запросов](#маршрутизация-и-обработка-запросов)
- [Middleware и фильтры](#middleware-и-фильтры)
- [Сравнение фреймворков](#сравнение-фреймворков)

## Обзор HTTP-серверных фреймворков

В экосистеме Rust существует несколько HTTP-серверных фреймворков, каждый со своими особенностями:

| Фреймворк | Описание | Особенности |
|-----------|----------|-------------|
| **hyper** | Низкоуровневая HTTP-библиотека | Высокая производительность, гибкость, минимальная абстракция |
| **Actix Web** | Высокопроизводительный веб-фреймворк | Основан на актерской модели, полнофункциональный, высокая производительность |
| **Rocket** | Интуитивный веб-фреймворк | Фокус на удобстве использования, безопасности типов и хорошей документации |
| **Warp** | Легковесный веб-фреймворк | Основан на фильтрах, композиционный подход, высокая производительность |
| **Tide** | Минималистичный веб-фреймворк | Простота, модульность, асинхронность |
| **Axum** | Эргономичный веб-фреймворк | Основан на Tower, модульный, типобезопасный |

## Простой HTTP-сервер с hyper

`hyper` - это низкоуровневая HTTP-библиотека, которая обеспечивает высокую производительность и гибкость. Она является основой для многих других веб-фреймворков.

### Установка

```toml
[dependencies]
hyper = { version = "0.14", features = ["full"] }
tokio = { version = "1", features = ["full"] }
```

### Простой HTTP-сервер

```rust
use hyper::service::{make_service_fn, service_fn};
use hyper::{Body, Request, Response, Server};
use std::convert::Infallible;
use std::net::SocketAddr;

// Обработчик запросов
async fn handle(_req: Request<Body>) -> Result<Response<Body>, Infallible> {
    Ok(Response::new(Body::from("Привет, мир!")))
}

#[tokio::main]
async fn main() {
    // Адрес для прослушивания
    let addr = SocketAddr::from(([127, 0, 0, 1], 3000));
    
    // Создаем сервисную функцию
    let make_svc = make_service_fn(|_conn| async {
        Ok::<_, Infallible>(service_fn(handle))
    });
    
    // Создаем и запускаем сервер
    let server = Server::bind(&addr).serve(make_svc);
    
    println!("Сервер запущен на http://{}", addr);
    
    // Запускаем сервер
    if let Err(e) = server.await {
        eprintln!("Ошибка сервера: {}", e);
    }
}
```

### Обработка различных маршрутов

```rust
use hyper::{Body, Method, Request, Response, Server, StatusCode};
use hyper::service::{make_service_fn, service_fn};
use std::convert::Infallible;
use std::net::SocketAddr;

async fn handle(req: Request<Body>) -> Result<Response<Body>, Infallible> {
    let mut response = Response::new(Body::empty());
    
    match (req.method(), req.uri().path()) {
        // GET /
        (&Method::GET, "/") => {
            *response.body_mut() = Body::from("Привет, мир!");
        },
        
        // GET /about
        (&Method::GET, "/about") => {
            *response.body_mut() = Body::from("О нас");
        },
        
        // POST /echo
        (&Method::POST, "/echo") => {
            *response.body_mut() = req.into_body();
        },
        
        // Обработка 404 Not Found
        _ => {
            *response.status_mut() = StatusCode::NOT_FOUND;
            *response.body_mut() = Body::from("404 Not Found");
        },
    };
    
    Ok(response)
}

#[tokio::main]
async fn main() {
    let addr = SocketAddr::from(([127, 0, 0, 1], 3000));
    
    let make_svc = make_service_fn(|_conn| async {
        Ok::<_, Infallible>(service_fn(handle))
    });
    
    let server = Server::bind(&addr).serve(make_svc);
    
    println!("Сервер запущен на http://{}", addr);
    
    if let Err(e) = server.await {
        eprintln!("Ошибка сервера: {}", e);
    }
}
```

## Веб-фреймворк Actix Web

Actix Web - высокопроизводительный веб-фреймворк, построенный на основе актерской модели. Он предоставляет богатый набор функций и является одним из самых быстрых веб-фреймворков в мире.

### Установка

```toml
[dependencies]
actix-web = "4.3"
serde = { version = "1.0", features = ["derive"] }
serde_json = "1.0"
```

### Простой сервер

```rust
use actix_web::{web, App, HttpResponse, HttpServer, Responder};

// Обработчик GET-запроса
async fn hello() -> impl Responder {
    HttpResponse::Ok().body("Привет, мир!")
}

// Обработчик GET-запроса с параметрами пути
async fn hello_name(path: web::Path<String>) -> impl Responder {
    let name = path.into_inner();
    HttpResponse::Ok().body(format!("Привет, {}!", name))
}

#[actix_web::main]
async fn main() -> std::io::Result<()> {
    HttpServer::new(|| {
        App::new()
            .route("/", web::get().to(hello))
            .route("/hello/{name}", web::get().to(hello_name))
    })
    .bind("127.0.0.1:8080")?
    .run()
    .await
}
```

### Работа с JSON

```rust
use actix_web::{web, App, HttpResponse, HttpServer, Responder};
use serde::{Deserialize, Serialize};

#[derive(Serialize, Deserialize)]
struct User {
    name: String,
    age: u32,
}

// Обработчик POST-запроса с JSON-телом
async fn create_user(user: web::Json<User>) -> impl Responder {
    println!("Создан пользователь: {} ({})", user.name, user.age);
    HttpResponse::Created().json(user.0)
}

#[actix_web::main]
async fn main() -> std::io::Result<()> {
    HttpServer::new(|| {
        App::new()
            .route("/users", web::post().to(create_user))
    })
    .bind("127.0.0.1:8080")?
    .run()
    .await
}
```

### Структурирование приложения

```rust
use actix_web::{web, App, HttpResponse, HttpServer, Responder};

// Обработчики
async fn index() -> impl Responder {
    HttpResponse::Ok().body("Главная страница")
}

async fn about() -> impl Responder {
    HttpResponse::Ok().body("О нас")
}

async fn user_profile(path: web::Path<u32>) -> impl Responder {
    let user_id = path.into_inner();
    HttpResponse::Ok().body(format!("Профиль пользователя {}", user_id))
}

// Конфигурация маршрутов
fn config_app(cfg: &mut web::ServiceConfig) {
    cfg.service(
        web::scope("/api")
            .route("/users/{id}", web::get().to(user_profile))
    );
}

#[actix_web::main]
async fn main() -> std::io::Result<()> {
    HttpServer::new(|| {
        App::new()
            .route("/", web::get().to(index))
            .route("/about", web::get().to(about))
            .configure(config_app)
    })
    .bind("127.0.0.1:8080")?
    .run()
    .await
}
```

## Веб-фреймворк Rocket

Rocket - интуитивно понятный веб-фреймворк с фокусом на удобство использования и безопасность типов.

### Установка

```toml
[dependencies]
rocket = "0.5.0-rc.2"
serde = { version = "1.0", features = ["derive"] }
```

### Простой сервер

```rust
#[macro_use] extern crate rocket;

#[get("/")]
fn index() -> &'static str {
    "Привет, мир!"
}

#[get("/hello/<name>")]
fn hello(name: &str) -> String {
    format!("Привет, {}!", name)
}

#[launch]
fn rocket() -> _ {
    rocket::build()
        .mount("/", routes![index, hello])
}
```

### Работа с JSON

```rust
#[macro_use] extern crate rocket;

use rocket::serde::{Deserialize, Serialize, json::Json};

#[derive(Serialize, Deserialize)]
struct User {
    name: String,
    age: u32,
}

#[get("/")]
fn index() -> &'static str {
    "Привет, мир!"
}

#[post("/users", format = "json", data = "<user>")]
fn create_user(user: Json<User>) -> Json<User> {
    println!("Создан пользователь: {} ({})", user.name, user.age);
    user
}

#[launch]
fn rocket() -> _ {
    rocket::build()
        .mount("/", routes![index, create_user])
}
```

### Обработка форм

```rust
#[macro_use] extern crate rocket;

use rocket::form::Form;
use rocket::response::Redirect;

#[derive(FromForm)]
struct LoginForm {
    username: String,
    password: String,
}

#[get("/login")]
fn login_page() -> &'static str {
    "Пожалуйста, войдите в систему"
}

#[post("/login", data = "<login_form>")]
fn login(login_form: Form<LoginForm>) -> Redirect {
    println!("Попытка входа: {}", login_form.username);
    // В реальном приложении здесь была бы проверка учетных данных
    Redirect::to(uri!(dashboard(login_form.username.clone())))
}

#[get("/dashboard/<username>")]
fn dashboard(username: String) -> String {
    format!("Добро пожаловать в панель управления, {}!", username)
}

#[launch]
fn rocket() -> _ {
    rocket::build()
        .mount("/", routes![login_page, login, dashboard])
}
```

## Веб-фреймворк Warp

Warp - современный, легковесный веб-фреймворк, основанный на фильтрах.

### Установка

```toml
[dependencies]
warp = "0.3"
tokio = { version = "1", features = ["full"] }
serde = { version = "1.0", features = ["derive"] }
```

### Простой сервер

```rust
use warp::Filter;

#[tokio::main]
async fn main() {
    // Маршрут для корневого пути
    let hello = warp::path::end().map(|| "Привет, мир!");
    
    // Маршрут с параметром пути
    let hello_name = warp::path!("hello" / String)
        .map(|name| format!("Привет, {}!", name));
    
    // Объединяем маршруты
    let routes = hello.or(hello_name);
    
    // Запускаем сервер
    println!("Сервер запущен на http://127.0.0.1:8080");
    warp::serve(routes).run(([127, 0, 0, 1], 8080)).await;
}
```

### Работа с JSON

```rust
use warp::{Filter, Reply};
use serde::{Deserialize, Serialize};

#[derive(Deserialize, Serialize)]
struct User {
    name: String,
    age: u32,
}

#[tokio::main]
async fn main() {
    // Маршрут для корневого пути
    let hello = warp::path::end().map(|| "Привет, мир!");
    
    // Маршрут для создания пользователя
    let create_user = warp::path("users")
        .and(warp::post())
        .and(warp::body::json())
        .map(|user: User| {
            println!("Создан пользователь: {} ({})", user.name, user.age);
            warp::reply::json(&user)
        });
    
    // Объединяем маршруты
    let routes = hello.or(create_user);
    
    // Запускаем сервер
    println!("Сервер запущен на http://127.0.0.1:8080");
    warp::serve(routes).run(([127, 0, 0, 1], 8080)).await;
}
```

### Фильтры и композиция

```rust
use warp::{Filter, Rejection, Reply};
use serde::{Deserialize, Serialize};
use std::sync::{Arc, Mutex};
use std::collections::HashMap;

type Users = Arc<Mutex<HashMap<u32, User>>>;

#[derive(Clone, Deserialize, Serialize)]
struct User {
    id: u32,
    name: String,
    age: u32,
}

// Фильтр для извлечения хранилища пользователей
fn with_users(users: Users) -> impl Filter<Extract = (Users,), Error = std::convert::Infallible> + Clone {
    warp::any().map(move || users.clone())
}

// Обработчик для получения всех пользователей
async fn get_users(users: Users) -> Result<impl Reply, Rejection> {
    let users = users.lock().unwrap();
    let users_vec: Vec<User> = users.values().cloned().collect();
    Ok(warp::reply::json(&users_vec))
}

// Обработчик для создания пользователя
async fn create_user(new_user: User, users: Users) -> Result<impl Reply, Rejection> {
    let mut users = users.lock().unwrap();
    users.insert(new_user.id, new_user.clone());
    Ok(warp::reply::json(&new_user))
}

#[tokio::main]
async fn main() {
    // Создаем хранилище пользователей
    let users: Users = Arc::new(Mutex::new(HashMap::new()));
    
    // Маршрут для получения всех пользователей
    let get_users_route = warp::path("users")
        .and(warp::get())
        .and(with_users(users.clone()))
        .and_then(get_users);
    
    // Маршрут для создания пользователя
    let create_user_route = warp::path("users")
        .and(warp::post())
        .and(warp::body::json())
        .and(with_users(users.clone()))
        .and_then(create_user);
    
    // Объединяем маршруты
    let routes = get_users_route.or(create_user_route);
    
    // Запускаем сервер
    println!("Сервер запущен на http://127.0.0.1:8080");
    warp::serve(routes).run(([127, 0, 0, 1], 8080)).await;
}
```

## Маршрутизация и обработка запросов

Маршрутизация - это процесс определения, как запросы должны обрабатываться в зависимости от их URL, метода и других параметров.

### Пример сложной маршрутизации с Actix Web

```rust
use actix_web::{web, App, HttpResponse, HttpServer, Responder};
use serde::{Deserialize, Serialize};

#[derive(Serialize, Deserialize)]
struct User {
    id: Option<u32>,
    name: String,
    email: String,
}

// Хранилище пользователей (в реальном приложении это была бы база данных)
struct AppState {
    users: std::sync::Mutex<Vec<User>>,
}

// Получение всех пользователей
async fn get_users(data: web::Data<AppState>) -> impl Responder {
    let users = data.users.lock().unwrap();
    HttpResponse::Ok().json(&*users)
}

// Получение пользователя по ID
async fn get_user(path: web::Path<u32>, data: web::Data<AppState>) -> impl Responder {
    let user_id = path.into_inner();
    let users = data.users.lock().unwrap();
    
    match users.iter().find(|u| u.id == Some(user_id)) {
        Some(user) => HttpResponse::Ok().json(user),
        None => HttpResponse::NotFound().body(format!("Пользователь с ID {} не найден", user_id)),
    }
}

// Создание нового пользователя
async fn create_user(user: web::Json<User>, data: web::Data<AppState>) -> impl Responder {
    let mut users = data.users.lock().unwrap();
    
    // Генерируем новый ID
    let new_id = users.iter().map(|u| u.id.unwrap_or(0)).max().unwrap_or(0) + 1;
    
    // Создаем нового пользователя
    let mut new_user = user.into_inner();
    new_user.id = Some(new_id);
    
    // Добавляем пользователя
    users.push(new_user.clone());
    
    HttpResponse::Created().json(new_user)
}

// Обновление пользователя
async fn update_user(path: web::Path<u32>, user: web::Json<User>, data: web::Data<AppState>) -> impl Responder {
    let user_id = path.into_inner();
    let mut users = data.users.lock().unwrap();
    
    if let Some(index) = users.iter().position(|u| u.id == Some(user_id)) {
        // Обновляем пользователя, сохраняя его ID
        let mut updated_user = user.into_inner();
        updated_user.id = Some(user_id);
        users[index] = updated_user.clone();
        
        HttpResponse::Ok().json(updated_user)
    } else {
        HttpResponse::NotFound().body(format!("Пользователь с ID {} не найден", user_id))
    }
}

// Удаление пользователя
async fn delete_user(path: web::Path<u32>, data: web::Data<AppState>) -> impl Responder {
    let user_id = path.into_inner();
    let mut users = data.users.lock().unwrap();
    
    if let Some(index) = users.iter().position(|u| u.id == Some(user_id)) {
        users.remove(index);
        HttpResponse::NoContent().finish()
    } else {
        HttpResponse::NotFound().body(format!("Пользователь с ID {} не найден", user_id))
    }
}

#[actix_web::main]
async fn main() -> std::io::Result<()> {
    // Инициализируем состояние приложения
    let app_state = web::Data::new(AppState {
        users: std::sync::Mutex::new(vec![
            User { id: Some(1), name: "Alice".to_string(), email: "alice@example.com".to_string() },
            User { id: Some(2), name: "Bob".to_string(), email: "bob@example.com".to_string() },
        ]),
    });
    
    HttpServer::new(move || {
        App::new()
            .app_data(app_state.clone())
            .service(
                web::scope("/api")
                    .route("/users", web::get().to(get_users))
                    .route("/users", web::post().to(create_user))
                    .route("/users/{id}", web::get().to(get_user))
                    .route("/users/{id}", web::put().to(update_user))
                    .route("/users/{id}", web::delete().to(delete_user))
            )
    })
    .bind("127.0.0.1:8080")?
    .run()
    .await
}
```

## Middleware и фильтры

Middleware (промежуточное ПО) позволяет обрабатывать запросы до или после основных обработчиков. Это полезно для таких задач, как аутентификация, логирование, сжатие и т.д.

### Middleware в Actix Web

```rust
use actix_web::{dev::{self, Service, ServiceRequest, ServiceResponse, Transform}, Error, HttpResponse};
use futures::future::{ok, Ready};
use std::future::Future;
use std::pin::Pin;
use std::time::Instant;

// Структура middleware
pub struct Logger;

// Фабрика middleware
impl<S, B> Transform<S, ServiceRequest> for Logger
where
    S: Service<ServiceRequest, Response = ServiceResponse<B>, Error = Error>,
    S::Future: 'static,
    B: 'static,
{
    type Response = ServiceResponse<B>;
    type Error = Error;
    type InitError = ();
    type Transform = LoggerMiddleware<S>;
    type Future = Ready<Result<Self::Transform, Self::InitError>>;

    fn new_transform(&self, service: S) -> Self::Future {
        ok(LoggerMiddleware { service })
    }
}

pub struct LoggerMiddleware<S> {
    service: S,
}

impl<S, B> Service<ServiceRequest> for LoggerMiddleware<S>
where
    S: Service<ServiceRequest, Response = ServiceResponse<B>, Error = Error>,
    S::Future: 'static,
    B: 'static,
{
    type Response = ServiceResponse<B>;
    type Error = Error;
    type Future = Pin<Box<dyn Future<Output = Result<Self::Response, Self::Error>>>>;

    dev::forward_ready!(service);

    fn call(&self, req: ServiceRequest) -> Self::Future {
        let start = Instant::now();
        let method = req.method().clone();
        let path = req.path().to_owned();

        let fut = self.service.call(req);

        Box::pin(async move {
            let res = fut.await?;
            let duration = start.elapsed();
            println!("{} {} - {} - {:?}", method, path, res.status(), duration);
            Ok(res)
        })
    }
}

// Использование middleware в приложении
use actix_web::{web, App, HttpServer};

#[actix_web::main]
async fn main() -> std::io::Result<()> {
    HttpServer::new(|| {
        App::new()
            .wrap(Logger) // Добавляем middleware
            .route("/", web::get().to(|| async { HttpResponse::Ok().body("Привет, мир!") }))
    })
    .bind("127.0.0.1:8080")?
    .run()
    .await
}
```

### Фильтры в Warp

В Warp фильтры - это основной строительный блок для создания маршрутов:

```rust
use warp::Filter;
use std::time::Instant;

// Фильтр для логирования
fn log(name: &'static str) -> impl Filter<Extract = (), Error = std::convert::Infallible> + Copy {
    warp::any()
        .map(move || {
            let start = Instant::now();
            (start, name)
        })
        .and_then(|(start, name)| async move {
            println!("{}: начало обработки", name);
            Ok::<_, std::convert::Infallible>((start, name))
        })
        .untuple_one()
        .map(move |start| {
            let elapsed = start.elapsed();
            println!("{}: завершено за {:?}", name, elapsed);
        })
}

#[tokio::main]
async fn main() {
    // Маршрут с логированием
    let hello = warp::path::end()
        .and(log("GET /"))
        .map(|| "Привет, мир!");
    
    // Запускаем сервер
    println!("Сервер запущен на http://127.0.0.1:8080");
    warp::serve(hello).run(([127, 0, 0, 1], 8080)).await;
}
```

## Сравнение фреймворков

| Критерий | hyper | Actix Web | Rocket | Warp |
|----------|-------|-----------|--------|------|
| **Уровень абстракции** | Низкий | Высокий | Высокий | Средний |
| **Простота использования** | Низкая | Средняя | Высокая | Средняя |
| **Производительность** | Отличная | Отличная | Хорошая | Отличная |
| **Типобезопасность** | Средняя | Хорошая | Отличная | Отличная |
| **Зрелость** | Высокая | Высокая | Средняя | Средняя |
| **Экосистема** | Средняя | Большая | Средняя | Средняя |
| **Асинхронность** | Да | Да | Да | Да |
| **Документация** | Хорошая | Отличная | Отличная | Хорошая |

### Когда использовать каждый фреймворк

- **hyper**: Когда требуется низкоуровневый контроль или создание собственного фреймворка
- **Actix Web**: Для высокопроизводительных приложений с богатым функционалом
- **Rocket**: Когда важна простота использования и безопасность типов
- **Warp**: Для приложений, требующих гибкой композиции фильтров

## Заключение

Rust предлагает разнообразные инструменты для создания HTTP-серверов, от низкоуровневых библиотек до высокоуровневых фреймворков. Выбор конкретного инструмента зависит от требований вашего проекта: производительности, простоты использования, типобезопасности или гибкости.

В следующих разделах мы рассмотрим работу с HTTPS, WebSockets и другие продвинутые темы веб-разработки в Rust.