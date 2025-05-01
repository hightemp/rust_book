# Futures и async/await

## Введение в Futures

Futures (будущие значения) - это основа асинхронного программирования в Rust. Future представляет собой значение, которое может быть недоступно сейчас, но станет доступно в будущем. Это абстракция, которая позволяет писать код, ожидающий завершения асинхронных операций, таких как чтение из файла, сетевые запросы или таймеры.

## Трейт Future

В основе асинхронного программирования в Rust лежит трейт `Future`, определенный в стандартной библиотеке:

```rust
pub trait Future {
    type Output;
    
    fn poll(self: Pin<&mut Self>, cx: &mut Context<'_>) -> Poll<Self::Output>;
}
```

Где:
- `Output` - ассоциированный тип, представляющий результат, который будет получен при завершении future
- `poll` - метод, который проверяет, готов ли результат future
- `Pin<&mut Self>` - закрепленная ссылка на future, предотвращающая перемещение данных в памяти
- `Context` - контекст, содержащий "пробуждающий" (waker), который уведомляет исполнителя о готовности future
- `Poll` - перечисление с двумя вариантами: `Ready(T)` (future завершен с результатом T) и `Pending` (future еще не готов)

## Ручная реализация Future

Хотя обычно мы не реализуем трейт `Future` вручную, понимание его внутреннего устройства полезно. Вот пример простой реализации:

```rust
use std::future::Future;
use std::pin::Pin;
use std::task::{Context, Poll};
use std::time::{Duration, Instant};

struct Delay {
    when: Instant,
}

impl Future for Delay {
    type Output = ();
    
    fn poll(self: Pin<&mut Self>, cx: &mut Context<'_>) -> Poll<Self::Output> {
        if Instant::now() >= self.when {
            Poll::Ready(())
        } else {
            // Запланировать пробуждение в будущем
            let waker = cx.waker().clone();
            let when = self.when;
            
            std::thread::spawn(move || {
                let now = Instant::now();
                if now < when {
                    std::thread::sleep(when - now);
                }
                waker.wake();
            });
            
            Poll::Pending
        }
    }
}

// Создание future, который завершится через указанное время
fn delay(duration: Duration) -> Delay {
    Delay {
        when: Instant::now() + duration,
    }
}
```

## Комбинаторы Futures

До появления синтаксиса `async`/`await`, работа с futures осуществлялась с помощью комбинаторов - методов, которые преобразуют futures или комбинируют их. Некоторые из них:

- `map` - преобразует результат future
- `then` - выполняет один future после другого
- `and_then` - выполняет второй future, если первый успешно завершился
- `or_else` - выполняет второй future, если первый завершился с ошибкой
- `select` - ожидает завершения любого из двух futures
- `join` - ожидает завершения обоих futures

Пример использования комбинаторов:

```rust
use futures::future::{self, Future};

// Пример с комбинаторами (устаревший подход)
fn fetch_and_process() -> impl Future<Output = Result<String, Error>> {
    fetch_data()
        .and_then(|data| {
            process_data(data)
        })
        .map(|result| {
            format!("Processed: {}", result)
        })
}
```

## Синтаксис async/await

Синтаксис `async`/`await`, введенный в Rust 1.39, значительно упростил работу с futures:

- Ключевое слово `async` превращает функцию или блок кода в future
- Оператор `.await` приостанавливает выполнение до завершения future

### Асинхронные функции

Асинхронная функция объявляется с ключевым словом `async`:

```rust
async fn fetch_data() -> Result<String, Error> {
    // Асинхронный код
    Ok("data".to_string())
}
```

Это эквивалентно следующему коду:

```rust
fn fetch_data() -> impl Future<Output = Result<String, Error>> {
    async {
        // Асинхронный код
        Ok("data".to_string())
    }
}
```

### Асинхронные блоки

Можно создавать анонимные асинхронные блоки:

```rust
let future = async {
    // Асинхронный код
    "результат"
};
```

### Оператор await

Оператор `.await` используется для ожидания завершения future:

```rust
async fn process() -> Result<(), Error> {
    let data = fetch_data().await?;
    let result = process_data(data).await?;
    println!("Результат: {}", result);
    Ok(())
}
```

Оператор `.await` можно использовать только внутри асинхронных функций или блоков.

## Преимущества async/await

1. **Читаемость**: код выглядит почти как синхронный, что упрощает его понимание
2. **Композиция**: легко комбинировать асинхронные операции
3. **Обработка ошибок**: поддержка оператора `?` для распространения ошибок
4. **Эффективность**: компилятор генерирует оптимизированные конечные автоматы

## Внутреннее устройство async/await

Когда вы пишете асинхронную функцию, компилятор Rust преобразует ее в конечный автомат (state machine), где каждая точка `.await` представляет собой состояние. Это позволяет приостанавливать и возобновлять выполнение функции.

Упрощенно, асинхронная функция:

```rust
async fn example() {
    println!("Начало");
    delay(Duration::from_secs(1)).await;
    println!("Прошла 1 секунда");
    delay(Duration::from_secs(1)).await;
    println!("Прошло 2 секунды");
}
```

Преобразуется в нечто похожее на:

```rust
enum ExampleState {
    Start,
    WaitingFirstDelay(/* сохраненное состояние */),
    WaitingSecondDelay(/* сохраненное состояние */),
    Completed,
}

struct ExampleFuture {
    state: ExampleState,
}

impl Future for ExampleFuture {
    type Output = ();
    
    fn poll(mut self: Pin<&mut Self>, cx: &mut Context<'_>) -> Poll<Self::Output> {
        match self.state {
            ExampleState::Start => {
                println!("Начало");
                let delay_future = delay(Duration::from_secs(1));
                self.state = ExampleState::WaitingFirstDelay(delay_future);
                self.poll(cx) // Повторный опрос в новом состоянии
            }
            ExampleState::WaitingFirstDelay(ref mut delay_future) => {
                match Pin::new(delay_future).poll(cx) {
                    Poll::Ready(()) => {
                        println!("Прошла 1 секунда");
                        let delay_future = delay(Duration::from_secs(1));
                        self.state = ExampleState::WaitingSecondDelay(delay_future);
                        self.poll(cx) // Повторный опрос в новом состоянии
                    }
                    Poll::Pending => Poll::Pending,
                }
            }
            ExampleState::WaitingSecondDelay(ref mut delay_future) => {
                match Pin::new(delay_future).poll(cx) {
                    Poll::Ready(()) => {
                        println!("Прошло 2 секунды");
                        self.state = ExampleState::Completed;
                        Poll::Ready(())
                    }
                    Poll::Pending => Poll::Pending,
                }
            }
            ExampleState::Completed => Poll::Ready(()),
        }
    }
}
```

## Практические примеры использования async/await

### Параллельное выполнение futures

Для параллельного выполнения нескольких futures можно использовать `join!` или `try_join!`:

```rust
use futures::future::{self, join, try_join};

async fn parallel_tasks() -> Result<(), Error> {
    // Запуск двух задач параллельно
    let (result1, result2) = join!(
        async_task1(),
        async_task2()
    );
    
    // Запуск двух задач параллельно с обработкой ошибок
    let (data1, data2) = try_join!(
        fetch_data_1(),
        fetch_data_2()
    )?;
    
    process_results(data1, data2).await?;
    Ok(())
}
```

### Таймауты

Добавление таймаута к асинхронной операции:

```rust
use futures::future::{self, select};
use std::time::Duration;
use tokio::time::timeout;

async fn operation_with_timeout() -> Result<Data, Error> {
    // Вариант 1: с использованием timeout из tokio
    let result = timeout(Duration::from_secs(5), fetch_data()).await??;
    
    // Вариант 2: с использованием select и race
    let fetch_future = fetch_data();
    let timeout_future = tokio::time::sleep(Duration::from_secs(5));
    
    match future::select(fetch_future, timeout_future).await {
        Either::Left((result, _)) => result,
        Either::Right((_, _)) => Err(Error::Timeout),
    }
}
```

### Обработка потока данных

Асинхронная обработка потока данных с использованием `Stream`:

```rust
use futures::stream::{self, StreamExt};

async fn process_stream() {
    let mut stream = stream::iter(vec![1, 2, 3, 4, 5])
        .map(|x| async move {
            // Асинхронная обработка каждого элемента
            tokio::time::sleep(Duration::from_millis(100)).await;
            x * 2
        })
        .buffer_unordered(3) // Параллельная обработка до 3 элементов
        .collect::<Vec<_>>()
        .await;
        
    println!("Результаты: {:?}", stream);
}
```

## Распространенные проблемы и их решения

### 1. "Async блок в синхронном контексте"

Проблема: нельзя вызвать `.await` вне асинхронного контекста.

```rust
fn main() {
    // Ошибка: нельзя использовать .await здесь
    let result = fetch_data().await;
}
```

Решение: использовать исполнитель для запуска асинхронного кода:

```rust
fn main() {
    // С tokio
    tokio::runtime::Runtime::new()
        .unwrap()
        .block_on(async {
            let result = fetch_data().await;
            println!("Результат: {:?}", result);
        });
        
    // Или с помощью #[tokio::main]
    // #[tokio::main]
    // async fn main() {
    //     let result = fetch_data().await;
    //     println!("Результат: {:?}", result);
    // }
}
```

### 2. Проблема "?Sized"

Проблема: трейт `Future` требует, чтобы `Self` был типом известного размера, что может вызвать проблемы при использовании с обобщенными типами.

Решение: использовать `Box<dyn Future<Output = T>>` или `Pin<Box<dyn Future<Output = T>>>`:

```rust
fn returns_future(condition: bool) -> Pin<Box<dyn Future<Output = i32>>> {
    if condition {
        Box::pin(async { 42 })
    } else {
        Box::pin(async { 0 })
    }
}
```

### 3. Проблема "не реализован трейт Send"

Проблема: некоторые futures не реализуют трейт `Send`, что не позволяет отправлять их между потоками.

Решение: убедиться, что все данные, захваченные future, реализуют `Send`, или использовать исполнитель в текущем потоке:

```rust
// Проблема: захват не-Send данных
let rc = Rc::new(42);
let future = async move {
    println!("значение: {}", rc);
};
// Ошибка: future не реализует Send

// Решение: использовать локальный исполнитель
tokio::task::LocalSet::new().block_on(&rt, async move {
    let rc = Rc::new(42);
    let future = async move {
        println!("значение: {}", rc);
    };
    tokio::task::spawn_local(future).await.unwrap();
});
```

## Заключение

Futures и синтаксис `async`/`await` являются фундаментальными компонентами асинхронного программирования в Rust. Они позволяют писать эффективный, неблокирующий код, который может обрабатывать тысячи параллельных операций с минимальными накладными расходами.

Хотя внутреннее устройство futures может быть сложным, синтаксис `async`/`await` делает асинхронный код почти таким же читаемым, как и синхронный, сохраняя при этом все преимущества асинхронного выполнения.

В следующем разделе мы рассмотрим асинхронные среды выполнения, такие как Tokio, которые предоставляют исполнители для запуска futures и дополнительные инструменты для асинхронного программирования.