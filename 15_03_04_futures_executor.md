# futures-executor

## Введение в futures-executor

[futures-executor](https://docs.rs/futures-executor/) - это минималистичный исполнитель (executor) для futures, который является частью официального крейта [futures](https://github.com/rust-lang/futures-rs). Он предоставляет базовую функциональность для запуска асинхронных задач без лишних накладных расходов и сложностей. futures-executor является самым простым и легковесным исполнителем в экосистеме Rust, что делает его идеальным для простых приложений или для понимания основ асинхронного программирования.

## История и развитие

futures-executor является частью проекта futures-rs, который был создан для стандартизации асинхронного программирования в Rust. Проект был начат Алексом Крайчиком (Alex Crichton) и другими участниками команды Rust.

Основные вехи в истории futures-executor:
- 2016: Начало разработки futures 0.1
- 2018: Переход на futures 0.3 с поддержкой async/await
- 2019: Стабилизация async/await в Rust
- 2020-настоящее время: Продолжение развития и улучшения

## Философия и принципы

futures-executor основан на нескольких ключевых принципах:

1. **Минимализм**: предоставление только базовой функциональности для запуска futures без лишних накладных расходов.

2. **Стандартизация**: соответствие стандартам и конвенциям, установленным в экосистеме Rust.

3. **Простота**: легкость в использовании и понимании, что делает его хорошим выбором для обучения асинхронному программированию.

4. **Интеграция**: хорошая интеграция с другими компонентами крейта futures.

## Архитектура futures-executor

futures-executor имеет простую архитектуру, состоящую из нескольких ключевых компонентов:

### 1. Блокирующий исполнитель (BlockingExecutor)

Блокирующий исполнитель предоставляет функцию `block_on`, которая блокирует текущий поток до завершения future. Это самый простой способ запуска асинхронного кода в синхронном контексте.

### 2. Локальный пул (LocalPool)

Локальный пул позволяет выполнять несколько futures в текущем потоке. Он предоставляет более гибкий контроль над выполнением futures по сравнению с `block_on`.

### 3. Локальный исполнитель (LocalSpawner)

Локальный исполнитель позволяет создавать новые задачи, которые будут выполняться в локальном пуле.

### 4. Многопоточный пул (ThreadPool)

Многопоточный пул позволяет выполнять futures параллельно на нескольких потоках. Это более продвинутый исполнитель, который подходит для задач, требующих параллельного выполнения.

## Основные компоненты futures-executor

### 1. Функция `block_on`

Функция `block_on` - это самый простой способ запуска асинхронного кода в синхронном контексте. Она блокирует текущий поток до завершения future и возвращает его результат.

```rust
use futures::executor::block_on;

fn main() {
    // Запуск асинхронного кода в синхронном контексте
    let result = block_on(async {
        // Асинхронный код
        println!("Выполняется асинхронный код");
        42
    });
    
    println!("Результат: {}", result);
}
```

### 2. Структура `LocalPool`

Структура `LocalPool` предоставляет более гибкий контроль над выполнением futures в текущем потоке. Она позволяет создавать новые задачи и выполнять их в порядке готовности.

```rust
use futures::executor::{LocalPool, LocalSpawner};
use futures::task::LocalSpawnExt;
use std::rc::Rc;

fn local_pool_example() {
    // Создание локального пула
    let mut pool = LocalPool::new();
    let spawner = pool.spawner();
    
    // Создание задачи
    spawner.spawn_local(async {
        // Асинхронный код
        println!("Задача 1");
    }).unwrap();
    
    // Создание еще одной задачи
    spawner.spawn_local(async {
        // Асинхронный код
        println!("Задача 2");
    }).unwrap();
    
    // Выполнение всех задач до завершения
    pool.run();
}
```

### 3. Структура `ThreadPool`

Структура `ThreadPool` позволяет выполнять futures параллельно на нескольких потоках. Она предоставляет возможность создавать новые задачи, которые будут распределены между рабочими потоками.

```rust
use futures::executor::ThreadPool;
use futures::task::SpawnExt;

fn thread_pool_example() -> Result<(), Box<dyn std::error::Error>> {
    // Создание многопоточного пула
    let pool = ThreadPool::new()?;
    
    // Создание задачи
    pool.spawn(async {
        // Асинхронный код
        println!("Задача 1");
    })?;
    
    // Создание еще одной задачи
    pool.spawn(async {
        // Асинхронный код
        println!("Задача 2");
    })?;
    
    // Ожидание завершения задач
    std::thread::sleep(std::time::Duration::from_secs(1));
    
    Ok(())
}
```

## Особенности и преимущества futures-executor

### 1. Минимальный размер и зависимости

futures-executor имеет минимальный размер и количество зависимостей, что делает его идеальным для проектов, где важен размер бинарного файла или время компиляции.

```toml
# Сравнение размеров зависимостей
# Tokio (полный)
tokio = { version = "1", features = ["full"] } # ~40 зависимостей

# async-std (полный)
async-std = { version = "1", features = ["attributes"] } # ~30 зависимостей

# smol (базовый)
smol = "1.2" # ~15 зависимостей

# futures-executor
futures-executor = "0.3" # ~5 зависимостей
```

### 2. Простота использования

API futures-executor прост и интуитивно понятен, что делает его легким для изучения и использования.

```rust
// Простой пример использования futures-executor
use futures::executor::block_on;

fn main() {
    // Запуск асинхронного кода
    let result = block_on(async {
        // Асинхронный код
        42
    });
    
    println!("Результат: {}", result);
}
```

### 3. Интеграция с крейтом futures

futures-executor хорошо интегрируется с другими компонентами крейта futures, что позволяет использовать все комбинаторы и утилиты из этого крейта.

```rust
use futures::executor::block_on;
use futures::future::{self, FutureExt};
use futures::stream::{self, StreamExt};

fn futures_integration() {
    // Запуск асинхронного кода
    block_on(async {
        // Использование комбинаторов futures
        let future1 = future::ready(1);
        let future2 = future::ready(2);
        
        // Параллельное выполнение futures
        let (result1, result2) = future::join(future1, future2).await;
        println!("Результаты: {} и {}", result1, result2);
        
        // Использование потоков
        let mut stream = stream::iter(vec![1, 2, 3, 4, 5]);
        while let Some(item) = stream.next().await {
            println!("Элемент: {}", item);
        }
    });
}
```

### 4. Стандартизация

futures-executor является частью официального крейта futures, что обеспечивает его соответствие стандартам и конвенциям, установленным в экосистеме Rust.

## Установка и настройка futures-executor

Для использования futures-executor добавьте его в зависимости вашего проекта:

```toml
# Cargo.toml
[dependencies]
futures = "0.3"
```

Или, если вы хотите использовать только futures-executor:

```toml
# Cargo.toml
[dependencies]
futures-executor = "0.3"
```

## Примеры использования futures-executor

### 1. Простой пример с `block_on`

```rust
use futures::executor::block_on;
use std::time::Duration;
use async_std::task::sleep;

fn main() {
    // Запуск асинхронного кода
    let result = block_on(async {
        println!("Начало выполнения");
        
        // Асинхронная задержка
        sleep(Duration::from_secs(1)).await;
        
        println!("Прошла 1 секунда");
        
        // Возвращаем результат
        42
    });
    
    println!("Результат: {}", result);
}
```

### 2. Пример с `LocalPool`

```rust
use futures::executor::{LocalPool, LocalSpawner};
use futures::task::LocalSpawnExt;
use std::rc::Rc;
use std::cell::RefCell;
use std::time::Duration;
use async_std::task::sleep;

fn main() {
    // Создание локального пула
    let mut pool = LocalPool::new();
    let spawner = pool.spawner();
    
    // Создание разделяемого состояния
    let counter = Rc::new(RefCell::new(0));
    
    // Создание задачи 1
    let counter_clone = counter.clone();
    spawner.spawn_local(async move {
        println!("Задача 1 начала выполнение");
        
        // Асинхронная задержка
        sleep(Duration::from_secs(1)).await;
        
        // Изменение разделяемого состояния
        *counter_clone.borrow_mut() += 1;
        
        println!("Задача 1 завершена");
    }).unwrap();
    
    // Создание задачи 2
    let counter_clone = counter.clone();
    spawner.spawn_local(async move {
        println!("Задача 2 начала выполнение");
        
        // Асинхронная задержка
        sleep(Duration::from_millis(500)).await;
        
        // Изменение разделяемого состояния
        *counter_clone.borrow_mut() += 2;
        
        println!("Задача 2 завершена");
    }).unwrap();
    
    // Выполнение всех задач до завершения
    pool.run();
    
    // Проверка результата
    println!("Конечное значение счетчика: {}", *counter.borrow());
}
```

### 3. Пример с `ThreadPool`

```rust
use futures::executor::ThreadPool;
use futures::task::SpawnExt;
use std::sync::{Arc, Mutex};
use std::time::Duration;
use async_std::task::sleep;

fn main() -> Result<(), Box<dyn std::error::Error>> {
    // Создание многопоточного пула
    let pool = ThreadPool::new()?;
    
    // Создание разделяемого состояния
    let counter = Arc::new(Mutex::new(0));
    
    // Создание задачи 1
    let counter_clone = counter.clone();
    pool.spawn(async move {
        println!("Задача 1 начала выполнение");
        
        // Асинхронная задержка
        sleep(Duration::from_secs(1)).await;
        
        // Изменение разделяемого состояния
        let mut lock = counter_clone.lock().unwrap();
        *lock += 1;
        
        println!("Задача 1 завершена");
    })?;
    
    // Создание задачи 2
    let counter_clone = counter.clone();
    pool.spawn(async move {
        println!("Задача 2 начала выполнение");
        
        // Асинхронная задержка
        sleep(Duration::from_millis(500)).await;
        
        // Изменение разделяемого состояния
        let mut lock = counter_clone.lock().unwrap();
        *lock += 2;
        
        println!("Задача 2 завершена");
    })?;
    
    // Ожидание завершения задач
    std::thread::sleep(Duration::from_secs(2));
    
    // Проверка результата
    println!("Конечное значение счетчика: {}", *counter.lock().unwrap());
    
    Ok(())
}
```

### 4. Пример с комбинаторами futures

```rust
use futures::executor::block_on;
use futures::future::{self, FutureExt};
use futures::stream::{self, StreamExt};
use std::time::Duration;
use async_std::task::sleep;

fn main() {
    // Запуск асинхронного кода
    block_on(async {
        // Создание нескольких futures
        let future1 = async {
            sleep(Duration::from_secs(1)).await;
            1
        };
        
        let future2 = async {
            sleep(Duration::from_secs(2)).await;
            2
        };
        
        // Параллельное выполнение futures
        let (result1, result2) = future::join(future1, future2).await;
        println!("Результаты: {} и {}", result1, result2);
        
        // Выполнение futures с таймаутом
        match future::timeout(Duration::from_secs(1), future2).await {
            Ok(result) => println!("Результат: {}", result),
            Err(_) => println!("Таймаут"),
        }
        
        // Использование потоков
        let mut stream = stream::iter(vec![1, 2, 3, 4, 5])
            .map(|x| async move {
                sleep(Duration::from_millis(100 * x)).await;
                x * 2
            })
            .buffer_unordered(3); // Параллельное выполнение до 3 futures
        
        while let Some(item) = stream.next().await {
            println!("Элемент: {}", item);
        }
    });
}
```

## Сравнение с другими асинхронными средами выполнения

### futures-executor vs Tokio

| Аспект | futures-executor | Tokio |
|--------|-----------------|-------|
| Размер | Минимальный | Большой |
| Зависимости | Минимальные | Многочисленные |
| API | Базовый | Полный |
| Экосистема | Минимальная | Обширная |
| Фокус | Базовая функциональность | Производительность |
| Кривая обучения | Очень пологая | Более крутая |
| Производительность | Средняя | Очень высокая |

### futures-executor vs async-std

| Аспект | futures-executor | async-std |
|--------|-----------------|-----------|
| Размер | Минимальный | Средний |
| API | Базовый | Похож на std |
| Функциональность | Минимальная | Высокая |
| Фокус | Базовая функциональность | Совместимость со std |

### futures-executor vs smol

| Аспект | futures-executor | smol |
|--------|-----------------|------|
| Размер | Минимальный | Маленький |
| API | Базовый | Минималистичный |
| Функциональность | Минимальная | Базовая |
| Фокус | Базовая функциональность | Минимализм |

## Лучшие практики использования futures-executor

### 1. Используйте `block_on` для простых случаев

```rust
use futures::executor::block_on;

fn main() {
    // Запуск асинхронного кода
    let result = block_on(async {
        // Асинхронный код
        42
    });
    
    println!("Результат: {}", result);
}
```

### 2. Используйте `LocalPool` для более сложных случаев в однопоточном контексте

```rust
use futures::executor::{LocalPool, LocalSpawner};
use futures::task::LocalSpawnExt;

fn main() {
    // Создание локального пула
    let mut pool = LocalPool::new();
    let spawner = pool.spawner();
    
    // Создание задач
    for i in 0..5 {
        spawner.spawn_local(async move {
            println!("Задача {}", i);
        }).unwrap();
    }
    
    // Выполнение всех задач до завершения
    pool.run();
}
```

### 3. Используйте `ThreadPool` для параллельного выполнения

```rust
use futures::executor::ThreadPool;
use futures::task::SpawnExt;

fn main() -> Result<(), Box<dyn std::error::Error>> {
    // Создание многопоточного пула
    let pool = ThreadPool::builder()
        .pool_size(4) // Количество потоков
        .create()?;
    
    // Создание задач
    for i in 0..10 {
        pool.spawn(async move {
            println!("Задача {}", i);
        })?;
    }
    
    // Ожидание завершения задач
    std::thread::sleep(std::time::Duration::from_secs(1));
    
    Ok(())
}
```

### 4. Используйте комбинаторы futures для композиции асинхронных операций

```rust
use futures::executor::block_on;
use futures::future::{self, FutureExt};

fn main() {
    // Запуск асинхронного кода
    block_on(async {
        // Создание нескольких futures
        let future1 = future::ready(1);
        let future2 = future::ready(2);
        
        // Последовательное выполнение
        let result = future1.then(|x| async move { x + 1 }).await;
        println!("Результат: {}", result);
        
        // Параллельное выполнение
        let (result1, result2) = future::join(future1, future2).await;
        println!("Результаты: {} и {}", result1, result2);
        
        // Выбор первого завершившегося future
        let result = future::select(future1, future2).await;
        match result {
            future::Either::Left((x, _)) => println!("Первый завершился: {}", x),
            future::Either::Right((x, _)) => println!("Второй завершился: {}", x),
        }
    });
}
```

### 5. Используйте `futures::stream` для обработки потоков данных

```rust
use futures::executor::block_on;
use futures::stream::{self, StreamExt};

fn main() {
    // Запуск асинхронного кода
    block_on(async {
        // Создание потока
        let mut stream = stream::iter(vec![1, 2, 3, 4, 5]);
        
        // Обработка элементов потока
        while let Some(item) = stream.next().await {
            println!("Элемент: {}", item);
        }
        
        // Преобразование потока
        let sum = stream::iter(vec![1, 2, 3, 4, 5])
            .map(|x| x * 2)
            .fold(0, |acc, x| async move { acc + x })
            .await;
        
        println!("Сумма: {}", sum);
    });
}
```

## Ограничения и недостатки futures-executor

### 1. Ограниченная функциональность

futures-executor предоставляет только базовую функциональность для запуска futures, и для более сложных сценариев может потребоваться использование других асинхронных сред выполнения.

### 2. Отсутствие встроенной поддержки асинхронного ввода-вывода

В отличие от Tokio, async-std и smol, futures-executor не предоставляет встроенной поддержки асинхронного ввода-вывода, и для этого требуется использование дополнительных библиотек.

### 3. Ограниченная производительность

futures-executor не оптимизирован для высокой производительности, и для высоконагруженных сценариев лучше использовать более продвинутые асинхронные среды выполнения, такие как Tokio.

### 4. Отсутствие инструментов для отладки и мониторинга

futures-executor не предоставляет встроенных инструментов для отладки и мониторинга асинхронного кода.

## Заключение

futures-executor - это минималистичный исполнитель для futures, который является частью официального крейта futures. Он предоставляет базовую функциональность для запуска асинхронных задач без лишних накладных расходов и сложностей. Благодаря своему минимализму, простоте использования и интеграции с крейтом futures, futures-executor является отличным выбором для простых приложений или для понимания основ асинхронного программирования.

Основные преимущества futures-executor:
- Минимальный размер и зависимости
- Простота использования и пологая кривая обучения
- Интеграция с крейтом futures
- Стандартизация и соответствие конвенциям Rust

futures-executor особенно подходит для:
- Простых асинхронных приложений, не требующих продвинутой функциональности
- Обучения основам асинхронного программирования
- Проектов, где важен минимальный размер бинарного файла или время компиляции
- Интеграции с другими компонентами крейта futures

Выбор между futures-executor и другими асинхронными средами выполнения зависит от конкретных требований проекта, предпочтений команды разработчиков и других факторов.