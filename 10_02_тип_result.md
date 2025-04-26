# Тип Result&lt;T, E&gt;

В предыдущем разделе мы рассмотрели механизм паники в Rust, который используется для обработки критических, невосстановимых ошибок. Однако в большинстве случаев ошибки можно предвидеть и обработать более элегантно, не прерывая выполнение программы. Для этого в Rust используется тип `Result<T, E>`.

## Что такое Result&lt;T, E&gt;?

`Result<T, E>` - это перечисление (enum), определенное в стандартной библиотеке Rust, которое представляет результат операции, которая может завершиться успешно или с ошибкой:

```rust
enum Result<T, E> {
    Ok(T),    // Операция завершилась успешно, содержит результат типа T
    Err(E),   // Операция завершилась с ошибкой, содержит ошибку типа E
}
```

Где:
- `T` - тип значения, которое возвращается при успешном выполнении
- `E` - тип ошибки, которая возвращается при неудачном выполнении

Тип `Result` является обобщенным (generic), что позволяет использовать его с любыми типами данных для успешного результата и ошибки.

## Использование Result&lt;T, E&gt;

Рассмотрим пример функции, которая пытается преобразовать строку в целое число:

```rust
fn parse_number(input: &str) -> Result<i32, std::num::ParseIntError> {
    input.parse::<i32>()
}

fn main() {
    let result = parse_number("42");
    
    match result {
        Ok(number) => println!("Успешно распарсили число: {}", number),
        Err(error) => println!("Ошибка при парсинге: {}", error),
    }
    
    let result = parse_number("не число");
    
    match result {
        Ok(number) => println!("Успешно распарсили число: {}", number),
        Err(error) => println!("Ошибка при парсинге: {}", error),
    }
}
```

Вывод:
```
Успешно распарсили число: 42
Ошибка при парсинге: invalid digit found in string
```

В этом примере:
1. Функция `parse_number` возвращает `Result<i32, std::num::ParseIntError>`
2. Если парсинг успешен, возвращается `Ok` с числом внутри
3. Если парсинг не удался, возвращается `Err` с объектом ошибки
4. В `main` мы используем `match` для обработки обоих возможных результатов

## Методы типа Result

Тип `Result` предоставляет множество полезных методов для работы с результатами операций:

### Методы для извлечения значений

#### `unwrap()`

Метод `unwrap()` извлекает значение из `Ok` или вызывает панику, если результат - `Err`:

```rust
fn main() {
    let ok_result: Result<i32, &str> = Ok(42);
    let value = ok_result.unwrap(); // Вернет 42
    
    let err_result: Result<i32, &str> = Err("ошибка");
    // let value = err_result.unwrap(); // Вызовет панику с сообщением "ошибка"
}
```

#### `expect()`

Метод `expect()` похож на `unwrap()`, но позволяет указать собственное сообщение для паники:

```rust
fn main() {
    let err_result: Result<i32, &str> = Err("ошибка");
    // let value = err_result.expect("Произошла критическая ошибка"); 
    // Вызовет панику с сообщением "Произошла критическая ошибка: ошибка"
}
```

#### `unwrap_or()`

Метод `unwrap_or()` извлекает значение из `Ok` или возвращает указанное значение по умолчанию, если результат - `Err`:

```rust
fn main() {
    let ok_result: Result<i32, &str> = Ok(42);
    let value = ok_result.unwrap_or(0); // Вернет 42
    
    let err_result: Result<i32, &str> = Err("ошибка");
    let value = err_result.unwrap_or(0); // Вернет 0
    
    println!("Значение: {}", value);
}
```

#### `unwrap_or_else()`

Метод `unwrap_or_else()` извлекает значение из `Ok` или вычисляет значение по умолчанию с помощью переданной функции, если результат - `Err`:

```rust
fn main() {
    let err_result: Result<i32, &str> = Err("ошибка");
    let value = err_result.unwrap_or_else(|error| {
        println!("Произошла ошибка: {}", error);
        -1 // Значение по умолчанию
    });
    
    println!("Значение: {}", value);
}
```

### Методы для преобразования Result

#### `map()`

Метод `map()` преобразует значение внутри `Ok`, не изменяя `Err`:

```rust
fn main() {
    let ok_result: Result<i32, &str> = Ok(42);
    let mapped = ok_result.map(|x| x * 2); // Result<i32, &str> = Ok(84)
    
    let err_result: Result<i32, &str> = Err("ошибка");
    let mapped = err_result.map(|x| x * 2); // Result<i32, &str> = Err("ошибка")
}
```

#### `map_err()`

Метод `map_err()` преобразует ошибку внутри `Err`, не изменяя `Ok`:

```rust
fn main() {
    let ok_result: Result<i32, &str> = Ok(42);
    let mapped = ok_result.map_err(|e| format!("Ошибка: {}", e)); // Result<i32, String> = Ok(42)
    
    let err_result: Result<i32, &str> = Err("ошибка");
    let mapped = err_result.map_err(|e| format!("Ошибка: {}", e)); // Result<i32, String> = Err("Ошибка: ошибка")
}
```

#### `and_then()`

Метод `and_then()` (также известный как `flatMap` в других языках) позволяет выполнить цепочку операций, которые возвращают `Result`:

```rust
fn double(x: i32) -> Result<i32, &'static str> {
    Ok(x * 2)
}

fn main() {
    let ok_result: Result<i32, &str> = Ok(42);
    let chained = ok_result.and_then(double); // Result<i32, &str> = Ok(84)
    
    let err_result: Result<i32, &str> = Err("ошибка");
    let chained = err_result.and_then(double); // Result<i32, &str> = Err("ошибка")
}
```

#### `or_else()`

Метод `or_else()` позволяет обработать ошибку и вернуть новый `Result`:

```rust
fn handle_error(error: &str) -> Result<i32, &'static str> {
    println!("Обрабатываем ошибку: {}", error);
    Ok(0) // Восстанавливаемся после ошибки
}

fn main() {
    let ok_result: Result<i32, &str> = Ok(42);
    let recovered = ok_result.or_else(handle_error); // Result<i32, &str> = Ok(42)
    
    let err_result: Result<i32, &str> = Err("ошибка");
    let recovered = err_result.or_else(handle_error); // Result<i32, &str> = Ok(0)
}
```

### Комбинаторы для работы с несколькими Result

#### `and()`

Метод `and()` возвращает второй `Result`, если первый - `Ok`, иначе возвращает первую ошибку:

```rust
fn main() {
    let first: Result<i32, &str> = Ok(42);
    let second: Result<&str, &str> = Ok("успех");
    let result = first.and(second); // Result<&str, &str> = Ok("успех")
    
    let first: Result<i32, &str> = Err("ошибка 1");
    let second: Result<&str, &str> = Ok("успех");
    let result = first.and(second); // Result<&str, &str> = Err("ошибка 1")
    
    let first: Result<i32, &str> = Ok(42);
    let second: Result<&str, &str> = Err("ошибка 2");
    let result = first.and(second); // Result<&str, &str> = Err("ошибка 2")
}
```

#### `or()`

Метод `or()` возвращает первый `Result`, если он - `Ok`, иначе возвращает второй `Result`:

```rust
fn main() {
    let first: Result<i32, &str> = Ok(42);
    let second: Result<i32, &str> = Ok(24);
    let result = first.or(second); // Result<i32, &str> = Ok(42)
    
    let first: Result<i32, &str> = Err("ошибка 1");
    let second: Result<i32, &str> = Ok(24);
    let result = first.or(second); // Result<i32, &str> = Ok(24)
    
    let first: Result<i32, &str> = Err("ошибка 1");
    let second: Result<i32, &str> = Err("ошибка 2");
    let result = first.or(second); // Result<i32, &str> = Err("ошибка 2")
}
```

## Обработка нескольких операций, возвращающих Result

При работе с несколькими операциями, которые могут завершиться ошибкой, можно использовать различные подходы:

### Использование match

```rust
fn process_data(data: &str) -> Result<i32, String> {
    match data.parse::<i32>() {
        Ok(number) => {
            match number.checked_mul(2) {
                Some(doubled) => Ok(doubled),
                None => Err(String::from("Переполнение при умножении"))
            }
        },
        Err(e) => Err(format!("Ошибка парсинга: {}", e))
    }
}
```

### Использование комбинаторов

```rust
fn process_data(data: &str) -> Result<i32, String> {
    data.parse::<i32>()
        .map_err(|e| format!("Ошибка парсинга: {}", e))
        .and_then(|number| {
            number.checked_mul(2)
                .ok_or(String::from("Переполнение при умножении"))
        })
}
```

## Преобразование Option в Result

Часто требуется преобразовать `Option<T>` в `Result<T, E>`. Для этого можно использовать методы:

### `ok_or()`

```rust
fn find_user(id: u32) -> Option<String> {
    if id == 42 {
        Some(String::from("Alice"))
    } else {
        None
    }
}

fn main() {
    let user = find_user(42)
        .ok_or("Пользователь не найден");
    // Result<String, &str> = Ok("Alice")
    
    let user = find_user(1)
        .ok_or("Пользователь не найден");
    // Result<String, &str> = Err("Пользователь не найден")
}
```

### `ok_or_else()`

```rust
fn find_user(id: u32) -> Option<String> {
    if id == 42 {
        Some(String::from("Alice"))
    } else {
        None
    }
}

fn main() {
    let user = find_user(1)
        .ok_or_else(|| format!("Пользователь с ID {} не найден", 1));
    // Result<String, String> = Err("Пользователь с ID 1 не найден")
}
```

## Преобразование Result в Option

Иногда требуется преобразовать `Result<T, E>` в `Option<T>`, игнорируя информацию об ошибке:

### `ok()`

```rust
fn parse_number(s: &str) -> Result<i32, std::num::ParseIntError> {
    s.parse::<i32>()
}

fn main() {
    let number = parse_number("42").ok();
    // Option<i32> = Some(42)
    
    let number = parse_number("не число").ok();
    // Option<i32> = None
}
```

## Использование Result в main

Функция `main` в Rust может возвращать `Result`, что упрощает обработку ошибок в программах:

```rust
use std::fs::File;
use std::io::{self, Read};

fn main() -> Result<(), io::Error> {
    let mut file = File::open("config.txt")?;
    let mut content = String::new();
    file.read_to_string(&mut content)?;
    println!("Содержимое файла: {}", content);
    Ok(())
}
```

В этом примере:
1. Функция `main` возвращает `Result<(), io::Error>`
2. Если любая операция завершится с ошибкой, функция вернет эту ошибку
3. Если все операции успешны, функция вернет `Ok(())`

## Когда использовать Result вместо паники

`Result` следует использовать в следующих случаях:
1. Когда ошибка является ожидаемой частью нормальной работы программы
2. Когда вызывающий код должен иметь возможность обработать ошибку
3. Когда функция может завершиться неудачно по причинам, не связанным с программной ошибкой

Примеры:
- Чтение файла (файл может не существовать)
- Сетевые запросы (сервер может быть недоступен)
- Парсинг данных (данные могут быть некорректными)

## Заключение

Тип `Result<T, E>` является основным механизмом обработки ошибок в Rust. Он позволяет:
1. Явно указать, что функция может завершиться с ошибкой
2. Передать информацию об ошибке вызывающему коду
3. Заставить программиста явно обработать возможные ошибки

В следующем разделе мы рассмотрим оператор `?`, который значительно упрощает работу с `Result` и делает код более читаемым.