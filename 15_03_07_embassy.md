# Embassy

## Введение в Embassy

[Embassy](https://github.com/embassy-rs/embassy) - это асинхронная среда выполнения для Rust, специально разработанная для встраиваемых систем и микроконтроллеров. В отличие от других асинхронных сред выполнения, которые ориентированы на серверные или десктопные приложения, Embassy оптимизирован для работы в условиях ограниченных ресурсов, характерных для встраиваемых систем.

Embassy предоставляет не только асинхронную среду выполнения, но и целую экосистему для разработки встраиваемых приложений, включая драйверы для различных периферийных устройств, поддержку сетевых протоколов и интеграцию с различными микроконтроллерами.

## История и развитие

Embassy был создан Дирком-Яном Кройсом (Dirk-Jan Kruis) и сообществом разработчиков встраиваемых систем на Rust в 2020 году. Проект возник из необходимости иметь эффективную асинхронную среду выполнения для микроконтроллеров, которая бы использовала преимущества системы типов Rust и модели владения для обеспечения безопасности и надежности.

Основные вехи в истории Embassy:
- 2020: Начало разработки Embassy
- 2021: Первый стабильный релиз
- 2022: Расширение поддержки микроконтроллеров и периферийных устройств
- 2023-настоящее время: Продолжение развития и оптимизации

## Философия и принципы

Embassy основан на нескольких ключевых принципах:

1. **Оптимизация для встраиваемых систем**: минимальное использование памяти и процессорного времени, отсутствие динамической аллокации памяти.

2. **Безопасность**: использование системы типов Rust и модели владения для обеспечения безопасности и предотвращения ошибок.

3. **Эффективность**: минимальные накладные расходы и максимальная производительность.

4. **Модульность**: разделение на независимые компоненты, которые можно использовать по отдельности.

5. **Интеграция с экосистемой**: тесная интеграция с другими библиотеками и инструментами для встраиваемых систем.

## Архитектура Embassy

Embassy имеет модульную архитектуру, состоящую из нескольких ключевых компонентов:

### 1. Исполнитель (Executor)

Исполнитель Embassy отвечает за выполнение асинхронных задач. Он оптимизирован для работы на микроконтроллерах и не требует динамической аллокации памяти.

### 2. HAL (Hardware Abstraction Layer)

HAL предоставляет абстракции для работы с различными микроконтроллерами и периферийными устройствами. Он обеспечивает единый интерфейс для доступа к аппаратным возможностям различных платформ.

### 3. Драйверы

Embassy включает в себя драйверы для различных периферийных устройств, таких как датчики, дисплеи, сетевые интерфейсы и т.д. Эти драйверы используют асинхронный API для эффективной работы с устройствами.

### 4. Сетевой стек

Embassy предоставляет асинхронный сетевой стек, который поддерживает различные протоколы, такие как TCP/IP, UDP, MQTT и т.д. Он оптимизирован для работы на микроконтроллерах с ограниченными ресурсами.

## Основные компоненты Embassy

### 1. embassy-executor

`embassy-executor` - это ядро асинхронной среды выполнения Embassy. Он предоставляет исполнитель для асинхронных задач, оптимизированный для встраиваемых систем:

```rust
use embassy_executor::Executor;
use embassy_executor::task;
use embassy_time::{Duration, Timer};

// Определение асинхронной задачи
#[task]
async fn my_task() {
    loop {
        // Асинхронный код
        println!("Привет, Embassy!");
        Timer::after(Duration::from_secs(1)).await;
    }
}

fn main() {
    // Создание исполнителя
    let mut executor = Executor::new();
    
    // Запуск задачи
    executor.spawn(my_task());
    
    // Запуск исполнителя
    executor.run();
}
```

### 2. embassy-time

`embassy-time` предоставляет функциональность для работы с временем, включая таймеры и задержки:

```rust
use embassy_executor::Executor;
use embassy_executor::task;
use embassy_time::{Duration, Timer, Instant};

#[task]
async fn timing_example() {
    // Получение текущего времени
    let start = Instant::now();
    
    // Задержка
    Timer::after(Duration::from_millis(100)).await;
    
    // Вычисление прошедшего времени
    let elapsed = start.elapsed();
    println!("Прошло: {:?}", elapsed);
    
    // Периодическое выполнение
    let mut interval = Timer::interval(Duration::from_secs(1));
    for _ in 0..5 {
        interval.next().await;
        println!("Тик");
    }
}
```

### 3. embassy-sync

`embassy-sync` предоставляет примитивы синхронизации для асинхронного кода:

```rust
use embassy_executor::Executor;
use embassy_executor::task;
use embassy_sync::channel::Channel;
use embassy_sync::signal::Signal;
use embassy_sync::mutex::Mutex;
use embassy_time::{Duration, Timer};

// Канал для передачи данных между задачами
static CHANNEL: Channel<u32, 10> = Channel::new();

// Сигнал для уведомления о событии
static SIGNAL: Signal<bool> = Signal::new();

// Мьютекс для защиты разделяемых данных
static MUTEX: Mutex<u32> = Mutex::new(0);

#[task]
async fn producer() {
    for i in 0..5 {
        // Отправка данных в канал
        CHANNEL.send(i).await;
        
        // Задержка
        Timer::after(Duration::from_secs(1)).await;
    }
    
    // Отправка сигнала о завершении
    SIGNAL.signal(true);
}

#[task]
async fn consumer() {
    loop {
        // Проверка сигнала
        if SIGNAL.signaled() {
            break;
        }
        
        // Получение данных из канала
        if let Ok(value) = CHANNEL.try_receive() {
            println!("Получено: {}", value);
            
            // Обновление разделяемых данных
            let mut lock = MUTEX.lock().await;
            *lock += value;
        }
        
        // Задержка
        Timer::after(Duration::from_millis(500)).await;
    }
    
    // Вывод итогового значения
    let lock = MUTEX.lock().await;
    println!("Итоговое значение: {}", *lock);
}
```

### 4. embassy-hal

`embassy-hal` предоставляет абстракции для работы с аппаратными возможностями микроконтроллеров:

```rust
use embassy_executor::Executor;
use embassy_executor::task;
use embassy_stm32::gpio::{Level, Output, Speed};
use embassy_stm32::Peripherals;
use embassy_time::{Duration, Timer};

#[task]
async fn blink(mut led: Output<'static>) {
    loop {
        // Включение светодиода
        led.set_high();
        
        // Задержка
        Timer::after(Duration::from_millis(500)).await;
        
        // Выключение светодиода
        led.set_low();
        
        // Задержка
        Timer::after(Duration::from_millis(500)).await;
    }
}

#[embassy_executor::main]
async fn main(spawner: Spawner) {
    // Инициализация периферии
    let p = embassy_stm32::init(Default::default());
    
    // Настройка светодиода
    let led = Output::new(p.PC13, Level::Low, Speed::Low);
    
    // Запуск задачи мигания светодиодом
    spawner.spawn(blink(led)).unwrap();
}
```

### 5. embassy-net

`embassy-net` предоставляет сетевой стек для встраиваемых систем:

```rust
use embassy_executor::Executor;
use embassy_executor::task;
use embassy_net::{Stack, StackResources, Config};
use embassy_net::tcp::TcpSocket;
use embassy_net::dns::DnsSocket;
use embassy_time::{Duration, Timer};
use embassy_stm32::eth::EthernetDevice;

// Буферы для сетевого стека
static STACK_RESOURCES: StackResources<2> = StackResources::new();

#[task]
async fn net_task(stack: &'static Stack<EthernetDevice>) {
    // Запуск сетевого стека
    stack.run().await;
}

#[task]
async fn http_client(stack: &'static Stack<EthernetDevice>) {
    // Ожидание подключения к сети
    loop {
        if stack.is_link_up() {
            break;
        }
        Timer::after(Duration::from_millis(500)).await;
    }
    
    // Создание DNS-сокета
    let mut dns = DnsSocket::new(stack);
    
    // Разрешение доменного имени
    let addr = dns.lookup("example.com").await.unwrap();
    
    // Создание TCP-сокета
    let mut socket = TcpSocket::new(stack, &mut rx_buffer, &mut tx_buffer);
    
    // Подключение к серверу
    socket.connect((addr, 80)).await.unwrap();
    
    // Отправка HTTP-запроса
    socket.write_all(b"GET / HTTP/1.0\r\nHost: example.com\r\n\r\n").await.unwrap();
    
    // Чтение ответа
    let mut buffer = [0; 1024];
    let n = socket.read(&mut buffer).await.unwrap();
    
    // Вывод ответа
    let response = core::str::from_utf8(&buffer[..n]).unwrap();
    println!("Ответ: {}", response);
    
    // Закрытие соединения
    socket.close();
}
```

## Особенности и преимущества Embassy

### 1. Оптимизация для встраиваемых систем

Embassy оптимизирован для работы на микроконтроллерах с ограниченными ресурсами. Он минимизирует использование памяти и процессорного времени, не требует динамической аллокации памяти и обеспечивает предсказуемое поведение.

### 2. Интеграция с экосистемой встраиваемых систем

Embassy тесно интегрирован с другими библиотеками и инструментами для встраиваемых систем, такими как `embedded-hal`, `defmt` и т.д. Это позволяет легко использовать его в существующих проектах.

### 3. Безопасность

Embassy использует систему типов Rust и модель владения для обеспечения безопасности и предотвращения ошибок. Это особенно важно для встраиваемых систем, где ошибки могут иметь серьезные последствия.

### 4. Модульность

Embassy имеет модульную архитектуру, которая позволяет использовать только те компоненты, которые необходимы для конкретного проекта. Это минимизирует размер итогового бинарного файла и использование ресурсов.

### 5. Поддержка различных микроконтроллеров

Embassy поддерживает различные микроконтроллеры, включая STM32, nRF, RP2040 и другие. Это позволяет использовать его на различных платформах без изменения кода.

## Установка и настройка Embassy

Для использования Embassy добавьте необходимые компоненты в зависимости вашего проекта:

```toml
# Cargo.toml
[dependencies]
embassy-executor = { version = "0.1.0", features = ["nightly", "arch-cortex-m"] }
embassy-time = { version = "0.1.0", features = ["nightly", "tick-hz-32_768"] }
embassy-sync = { version = "0.1.0" }
embassy-stm32 = { version = "0.1.0", features = ["stm32f411ce", "time-driver-tim2"] }
```

Для использования Embassy с конкретным микроконтроллером необходимо включить соответствующие функции:

```toml
# Для STM32F4
embassy-stm32 = { version = "0.1.0", features = ["stm32f411ce", "time-driver-tim2"] }

# Для nRF52
embassy-nrf = { version = "0.1.0", features = ["nrf52840", "time-driver-rtc1"] }

# Для RP2040
embassy-rp = { version = "0.1.0", features = ["time-driver"] }
```

## Примеры использования Embassy

### 1. Мигание светодиодом

```rust
#![no_std]
#![no_main]
#![feature(type_alias_impl_trait)]

use embassy_executor::Spawner;
use embassy_stm32::gpio::{Level, Output, Speed};
use embassy_stm32::Peripherals;
use embassy_time::{Duration, Timer};
use panic_halt as _;

#[embassy_executor::main]
async fn main(spawner: Spawner) {
    // Инициализация периферии
    let p = embassy_stm32::init(Default::default());
    
    // Настройка светодиода
    let mut led = Output::new(p.PC13, Level::Low, Speed::Low);
    
    // Мигание светодиодом
    loop {
        // Включение светодиода
        led.set_high();
        
        // Задержка
        Timer::after(Duration::from_millis(500)).await;
        
        // Выключение светодиода
        led.set_low();
        
        // Задержка
        Timer::after(Duration::from_millis(500)).await;
    }
}
```

### 2. Чтение датчика температуры

```rust
#![no_std]
#![no_main]
#![feature(type_alias_impl_trait)]

use embassy_executor::Spawner;
use embassy_stm32::adc::{Adc, SampleTime};
use embassy_stm32::Peripherals;
use embassy_time::{Duration, Timer};
use panic_halt as _;

#[embassy_executor::main]
async fn main(spawner: Spawner) {
    // Инициализация периферии
    let p = embassy_stm32::init(Default::default());
    
    // Настройка АЦП
    let mut adc = Adc::new(p.ADC1, &mut embassy_time::Delay);
    
    // Чтение температуры
    loop {
        // Чтение значения АЦП
        let value = adc.read(p.PA0, SampleTime::Cycles_480);
        
        // Преобразование в температуру
        let temperature = (value as f32) * 0.1 - 50.0;
        
        // Вывод температуры
        defmt::info!("Температура: {:.1} °C", temperature);
        
        // Задержка
        Timer::after(Duration::from_secs(1)).await;
    }
}
```

### 3. Веб-сервер

```rust
#![no_std]
#![no_main]
#![feature(type_alias_impl_trait)]

use embassy_executor::Spawner;
use embassy_net::{Stack, StackResources, Config};
use embassy_net::tcp::TcpSocket;
use embassy_stm32::eth::EthernetDevice;
use embassy_stm32::Peripherals;
use embassy_time::{Duration, Timer};
use panic_halt as _;

// Буферы для сетевого стека
static STACK_RESOURCES: StackResources<2> = StackResources::new();

// Буферы для TCP-сокета
static mut RX_BUFFER: [u8; 1024] = [0; 1024];
static mut TX_BUFFER: [u8; 1024] = [0; 1024];

#[embassy_executor::task]
async fn net_task(stack: &'static Stack<EthernetDevice>) {
    // Запуск сетевого стека
    stack.run().await;
}

#[embassy_executor::task]
async fn web_server(stack: &'static Stack<EthernetDevice>) {
    // Ожидание подключения к сети
    loop {
        if stack.is_link_up() {
            break;
        }
        Timer::after(Duration::from_millis(500)).await;
    }
    
    // Создание TCP-сокета
    let mut socket = TcpSocket::new(stack, unsafe { &mut RX_BUFFER }, unsafe { &mut TX_BUFFER });
    
    // Привязка к порту
    socket.bind(80).unwrap();
    
    // Ожидание входящих соединений
    socket.listen().unwrap();
    
    loop {
        // Принятие соединения
        let mut client = socket.accept().await.unwrap();
        
        // Чтение запроса
        let mut buffer = [0; 1024];
        let n = client.read(&mut buffer).await.unwrap();
        
        // Формирование ответа
        let response = "HTTP/1.0 200 OK\r\n\
                       Content-Type: text/html\r\n\
                       \r\n\
                       <html><body><h1>Hello, Embassy!</h1></body></html>";
        
        // Отправка ответа
        client.write_all(response.as_bytes()).await.unwrap();
        
        // Закрытие соединения
        client.close();
    }
}

#[embassy_executor::main]
async fn main(spawner: Spawner) {
    // Инициализация периферии
    let p = embassy_stm32::init(Default::default());
    
    // Настройка Ethernet
    let eth_device = EthernetDevice::new(
        p.ETH,
        p.PA1,
        p.PA2,
        p.PC1,
        p.PA7,
        p.PC4,
        p.PC5,
        p.PG11,
        p.PG13,
    );
    
    // Настройка сетевого стека
    let config = Config::dhcp();
    let stack = Stack::new(eth_device, config, &STACK_RESOURCES);
    
    // Запуск сетевых задач
    spawner.spawn(net_task(&stack)).unwrap();
    spawner.spawn(web_server(&stack)).unwrap();
}
```

## Сравнение с другими асинхронными средами выполнения

### Embassy vs Tokio

| Аспект | Embassy | Tokio |
|--------|---------|-------|
| Целевая платформа | Встраиваемые системы | Серверные и десктопные приложения |
| Использование памяти | Минимальное | Высокое |
| Динамическая аллокация | Нет | Да |
| Поддержка `no_std` | Да | Нет |
| Экосистема | Специализированная для встраиваемых систем | Обширная для серверных приложений |
| Фокус | Эффективность и предсказуемость | Производительность и масштабируемость |

### Embassy vs RTIC

| Аспект | Embassy | RTIC |
|--------|---------|------|
| Модель программирования | Асинхронная | Событийно-ориентированная |
| Приоритеты задач | Нет | Да |
| Анализ времени выполнения | Нет | Да |
| Асинхронный код | Да | Нет |
| Интеграция с `async`/`await` | Да | Нет |

### Embassy vs smol

| Аспект | Embassy | smol |
|--------|---------|------|
| Целевая платформа | Встраиваемые системы | Серверные и десктопные приложения |
| Использование памяти | Минимальное | Среднее |
| Поддержка `no_std` | Да | Нет |
| Размер | Маленький | Маленький |
| Фокус | Встраиваемые системы | Минимализм |

## Лучшие практики использования Embassy

### 1. Используйте статическое выделение памяти

```rust
// Плохо: динамическое выделение памяти
let buffer = vec![0; 1024];

// Хорошо: статическое выделение памяти
static mut BUFFER: [u8; 1024] = [0; 1024];
```

### 2. Избегайте блокирующих операций

```rust
// Плохо: блокирующая операция
std::thread::sleep(std::time::Duration::from_secs(1));

// Хорошо: асинхронная задержка
embassy_time::Timer::after(embassy_time::Duration::from_secs(1)).await;
```

### 3. Используйте задачи для параллельных операций

```rust
use embassy_executor::Spawner;

#[embassy_executor::task]
async fn task1() {
    // Асинхронный код
}

#[embassy_executor::task]
async fn task2() {
    // Асинхронный код
}

#[embassy_executor::main]
async fn main(spawner: Spawner) {
    // Запуск задач
    spawner.spawn(task1()).unwrap();
    spawner.spawn(task2()).unwrap();
}
```

### 4. Используйте примитивы синхронизации для обмена данными

```rust
use embassy_sync::channel::Channel;
use embassy_sync::signal::Signal;
use embassy_sync::mutex::Mutex;

// Статические примитивы синхронизации
static CHANNEL: Channel<u32, 10> = Channel::new();
static SIGNAL: Signal<bool> = Signal::new();
static MUTEX: Mutex<u32> = Mutex::new(0);
```

## Заключение

Embassy - это мощная асинхронная среда выполнения для Rust, специально разработанная для встраиваемых систем и микроконтроллеров. Она предоставляет эффективный и безопасный способ написания асинхронного кода для устройств с ограниченными ресурсами.

Основные преимущества Embassy:
- Оптимизация для встраиваемых систем
- Интеграция с экосистемой встраиваемых систем
- Безопасность
- Модульность
- Поддержка различных микроконтроллеров

Embassy особенно подходит для:
- Встраиваемых систем и микроконтроллеров
- Устройств Интернета вещей (IoT)
- Систем реального времени
- Приложений с ограниченными ресурсами

Благодаря своей эффективности, безопасности и модульности, Embassy становится все более популярным выбором для разработки встраиваемых систем на Rust.