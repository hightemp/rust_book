# Практика: Создание и запуск программы "Hello, World!"

## Введение

В этом практическом разделе мы применим полученные знания и создадим нашу первую программу на Rust — классический "Hello, World!". Мы пройдем весь путь от создания проекта до запуска готовой программы, а также рассмотрим некоторые модификации и эксперименты с кодом.

## Цели практического задания

- Создать новый проект Rust
- Написать и понять код программы "Hello, World!"
- Скомпилировать и запустить программу
- Модифицировать программу для выполнения дополнительных задач
- Закрепить полученные знания на практике

## Предварительные требования

Перед началом убедитесь, что у вас установлены:
- Rust и Cargo (см. раздел "Установка Rust и Cargo")
- Текстовый редактор или IDE с поддержкой Rust (например, Visual Studio Code с расширением rust-analyzer)

## Шаг 1: Создание нового проекта

Начнем с создания нового проекта с помощью Cargo. Откройте терминал и выполните следующую команду:

```bash
cargo new hello_world
```

Эта команда создаст новую директорию `hello_world` со следующей структурой:

```
hello_world/
├── .git/
├── .gitignore
├── Cargo.toml
└── src/
    └── main.rs
```

Перейдите в директорию проекта:

```bash
cd hello_world
```

## Шаг 2: Изучение сгенерированного кода

Cargo автоматически создает файл `src/main.rs` с простой программой "Hello, world!". Давайте откроем этот файл и изучим его содержимое:

```rust
fn main() {
    println!("Hello, world!");
}
```

Этот код состоит из одной функции `main()`, которая является точкой входа в программу. Внутри функции вызывается макрос `println!`, который выводит текст "Hello, world!" в стандартный поток вывода.

Давайте разберем этот код подробнее:

- `fn main()` — объявление функции `main`, которая не принимает аргументов и ничего не возвращает. Функция `main` является точкой входа для исполняемых программ на Rust.
- `println!("Hello, world!");` — вызов макроса `println!`, который выводит текст в стандартный поток вывода. Обратите внимание на восклицательный знак после имени макроса — это отличительная черта макросов в Rust.

## Шаг 3: Компиляция и запуск программы

Теперь скомпилируем и запустим нашу программу. В терминале, находясь в директории проекта, выполните:

```bash
cargo run
```

Эта команда выполнит компиляцию проекта и запустит полученный исполняемый файл. Вы должны увидеть вывод:

```
   Compiling hello_world v0.1.0 (/path/to/hello_world)
    Finished dev [unoptimized + debuginfo] target(s) in 0.42s
     Running `target/debug/hello_world`
Hello, world!
```

Поздравляем! Вы успешно создали, скомпилировали и запустили свою первую программу на Rust.

## Шаг 4: Компиляция без запуска

Если вы хотите только скомпилировать программу без запуска, используйте:

```bash
cargo build
```

Эта команда создаст исполняемый файл в директории `target/debug/`. Вы можете запустить его напрямую:

```bash
./target/debug/hello_world
```

Для создания оптимизированной версии используйте:

```bash
cargo build --release
```

Оптимизированный исполняемый файл будет создан в директории `target/release/`:

```bash
./target/release/hello_world
```

## Шаг 5: Модификация программы

Теперь давайте модифицируем нашу программу, чтобы она делала что-то более интересное. Откройте файл `src/main.rs` в текстовом редакторе и замените его содержимое следующим кодом:

```rust
fn main() {
    // Выводим приветствие
    println!("Привет, Rust!");
    
    // Выводим информацию о языке
    println!("Rust — это системный язык программирования,");
    println!("который очень быстр и предотвращает почти все сбои.");
    
    // Выводим форматированный текст
    let version = "1.70.0";
    println!("Текущая стабильная версия Rust: {}", version);
    
    // Выводим результат выражения
    println!("Результат вычисления 5 + 7 = {}", 5 + 7);
    
    // Выводим несколько значений
    println!("Координаты: ({}, {})", 10, 20);
    
    // Выводим именованные параметры
    println!(
        "Прямоугольник: ширина = {width}, высота = {height}",
        width = 30,
        height = 40
    );
    
    // Выводим разные типы данных
    println!("Целое число: {}, Дробное число: {}, Символ: {}, Строка: {}, Булево значение: {}",
             42, 3.14, 'A', "текст", true);
}
```

Сохраните файл и запустите программу:

```bash
cargo run
```

Вы должны увидеть более сложный вывод с различными примерами использования макроса `println!`.

## Шаг 6: Добавление пользовательского ввода

Давайте сделаем нашу программу интерактивной, добавив возможность ввода имени пользователя. Измените файл `src/main.rs`:

```rust
use std::io;

fn main() {
    println!("Привет! Как тебя зовут?");
    
    let mut name = String::new();
    
    io::stdin()
        .read_line(&mut name)
        .expect("Не удалось прочитать строку");
    
    // Удаляем символ новой строки в конце
    let name = name.trim();
    
    println!("Приятно познакомиться, {}! Добро пожаловать в мир Rust!", name);
}
```

Запустите программу:

```bash
cargo run
```

Программа запросит ваше имя и выведет персонализированное приветствие.

Давайте разберем новые элементы кода:

- `use std::io;` — импортирует модуль ввода-вывода из стандартной библиотеки
- `let mut name = String::new();` — создает изменяемую переменную `name` типа `String`
- `io::stdin().read_line(&mut name)` — читает строку из стандартного ввода и сохраняет ее в переменной `name`
- `.expect("Не удалось прочитать строку");` — обрабатывает возможную ошибку
- `let name = name.trim();` — удаляет пробельные символы (включая символ новой строки) в начале и конце строки

## Шаг 7: Создание более сложной программы

Теперь давайте создадим более сложную программу, которая будет запрашивать два числа и выводить их сумму. Измените файл `src/main.rs`:

```rust
use std::io;

fn main() {
    println!("Калькулятор суммы двух чисел");
    
    // Запрашиваем первое число
    println!("Введите первое число:");
    let mut input = String::new();
    io::stdin()
        .read_line(&mut input)
        .expect("Не удалось прочитать строку");
    
    // Преобразуем строку в число
    let number1: i32 = input.trim().parse()
        .expect("Пожалуйста, введите число!");
    
    // Запрашиваем второе число
    println!("Введите второе число:");
    let mut input = String::new();
    io::stdin()
        .read_line(&mut input)
        .expect("Не удалось прочитать строку");
    
    // Преобразуем строку в число
    let number2: i32 = input.trim().parse()
        .expect("Пожалуйста, введите число!");
    
    // Вычисляем и выводим сумму
    let sum = number1 + number2;
    println!("Сумма {} и {} равна {}", number1, number2, sum);
}
```

Запустите программу:

```bash
cargo run
```

Программа запросит два числа и выведет их сумму.

Новые элементы кода:

- `let number1: i32 = input.trim().parse()` — преобразует строку в 32-битное целое число
- `.expect("Пожалуйста, введите число!");` — обрабатывает ошибку, если введенный текст нельзя преобразовать в число
- `let sum = number1 + number2;` — вычисляет сумму двух чисел

## Шаг 8: Добавление функций

Давайте улучшим нашу программу, добавив функции для повторного использования кода. Измените файл `src/main.rs`:

```rust
use std::io;

// Функция для чтения числа с консоли
fn read_number(prompt: &str) -> i32 {
    println!("{}", prompt);
    
    let mut input = String::new();
    io::stdin()
        .read_line(&mut input)
        .expect("Не удалось прочитать строку");
    
    input.trim().parse()
        .expect("Пожалуйста, введите число!")
}

// Функция для вычисления суммы двух чисел
fn add(a: i32, b: i32) -> i32 {
    a + b
}

fn main() {
    println!("Калькулятор суммы двух чисел");
    
    // Запрашиваем числа
    let number1 = read_number("Введите первое число:");
    let number2 = read_number("Введите второе число:");
    
    // Вычисляем и выводим сумму
    let sum = add(number1, number2);
    println!("Сумма {} и {} равна {}", number1, number2, sum);
}
```

Запустите программу:

```bash
cargo run
```

Функциональность осталась той же, но код стал более модульным и легче читаемым.

Новые элементы кода:

- `fn read_number(prompt: &str) -> i32 { ... }` — объявление функции, которая принимает строку и возвращает целое число
- `fn add(a: i32, b: i32) -> i32 { ... }` — объявление функции для сложения двух чисел
- `a + b` — выражение, результат которого возвращается из функции (в Rust последнее выражение в функции без точки с запятой является возвращаемым значением)

## Шаг 9: Обработка ошибок

Давайте улучшим обработку ошибок в нашей программе. Измените файл `src/main.rs`:

```rust
use std::io;

// Функция для чтения числа с консоли с обработкой ошибок
fn read_number(prompt: &str) -> i32 {
    loop {
        println!("{}", prompt);
        
        let mut input = String::new();
        io::stdin()
            .read_line(&mut input)
            .expect("Не удалось прочитать строку");
        
        match input.trim().parse() {
            Ok(number) => return number,
            Err(_) => {
                println!("Ошибка: введите корректное число!");
                continue;
            }
        }
    }
}

// Функция для вычисления суммы двух чисел
fn add(a: i32, b: i32) -> i32 {
    a + b
}

fn main() {
    println!("Калькулятор суммы двух чисел");
    
    // Запрашиваем числа
    let number1 = read_number("Введите первое число:");
    let number2 = read_number("Введите второе число:");
    
    // Вычисляем и выводим сумму
    let sum = add(number1, number2);
    println!("Сумма {} и {} равна {}", number1, number2, sum);
}
```

Запустите программу:

```bash
cargo run
```

Теперь программа будет повторно запрашивать ввод, если пользователь введет некорректное число.

Новые элементы кода:

- `loop { ... }` — бесконечный цикл
- `match input.trim().parse() { ... }` — сопоставление с образцом для обработки результата
- `Ok(number) => return number,` — если преобразование успешно, возвращаем число
- `Err(_) => { ... }` — если произошла ошибка, выводим сообщение и продолжаем цикл
- `continue;` — переходим к следующей итерации цикла

## Шаг 10: Добавление меню и нескольких операций

Давайте расширим нашу программу, добавив меню и несколько арифметических операций. Измените файл `src/main.rs`:

```rust
use std::io;

// Функция для чтения числа с консоли с обработкой ошибок
fn read_number(prompt: &str) -> i32 {
    loop {
        println!("{}", prompt);
        
        let mut input = String::new();
        io::stdin()
            .read_line(&mut input)
            .expect("Не удалось прочитать строку");
        
        match input.trim().parse() {
            Ok(number) => return number,
            Err(_) => {
                println!("Ошибка: введите корректное число!");
                continue;
            }
        }
    }
}

// Функция для чтения выбора операции
fn read_operation() -> char {
    loop {
        println!("Выберите операцию:");
        println!("1. Сложение (+)");
        println!("2. Вычитание (-)");
        println!("3. Умножение (*)");
        println!("4. Деление (/)");
        
        let mut input = String::new();
        io::stdin()
            .read_line(&mut input)
            .expect("Не удалось прочитать строку");
        
        let input = input.trim();
        if input.len() != 1 {
            println!("Ошибка: введите один символ!");
            continue;
        }
        
        let operation = input.chars().next().unwrap();
        match operation {
            '1' | '+' => return '+',
            '2' | '-' => return '-',
            '3' | '*' => return '*',
            '4' | '/' => return '/',
            _ => {
                println!("Ошибка: неизвестная операция!");
                continue;
            }
        }
    }
}

// Функция для выполнения арифметической операции
fn perform_operation(a: i32, b: i32, operation: char) -> Result<i32, String> {
    match operation {
        '+' => Ok(a + b),
        '-' => Ok(a - b),
        '*' => Ok(a * b),
        '/' => {
            if b == 0 {
                Err(String::from("Ошибка: деление на ноль!"))
            } else {
                Ok(a / b)
            }
        },
        _ => Err(String::from("Ошибка: неизвестная операция!"))
    }
}

fn main() {
    println!("Простой калькулятор");
    
    loop {
        // Запрашиваем числа
        let number1 = read_number("Введите первое число:");
        let number2 = read_number("Введите второе число:");
        
        // Запрашиваем операцию
        let operation = read_operation();
        
        // Выполняем операцию и выводим результат
        match perform_operation(number1, number2, operation) {
            Ok(result) => {
                println!("{} {} {} = {}", number1, operation, number2, result);
            },
            Err(error) => {
                println!("{}", error);
            }
        }
        
        // Спрашиваем, хочет ли пользователь продолжить
        println!("Хотите выполнить еще одну операцию? (д/н)");
        let mut input = String::new();
        io::stdin()
            .read_line(&mut input)
            .expect("Не удалось прочитать строку");
        
        let input = input.trim().to_lowercase();
        if input != "д" && input != "да" && input != "y" && input != "yes" {
            break;
        }
    }
    
    println!("Спасибо за использование калькулятора!");
}
```

Запустите программу:

```bash
cargo run
```

Теперь у вас есть полноценный калькулятор с меню, несколькими операциями и обработкой ошибок.

## Задания для самостоятельной работы

1. **Базовое задание**: Модифицируйте программу "Hello, World!" так, чтобы она выводила приветствие на нескольких языках.

2. **Среднее задание**: Расширьте калькулятор, добавив операции возведения в степень и вычисления остатка от деления.

3. **Продвинутое задание**: Создайте программу, которая запрашивает у пользователя информацию (имя, возраст, хобби) и генерирует персонализированную историю на основе этих данных.

## Заключение

В этом практическом разделе мы создали и запустили нашу первую программу на Rust, а затем постепенно расширяли ее, добавляя новые функции и возможности. Мы познакомились с основными концепциями языка, такими как функции, переменные, ввод-вывод, обработка ошибок и циклы.

Теперь у вас есть базовое понимание того, как создавать, компилировать и запускать программы на Rust. В следующих главах мы будем углубляться в изучение языка и рассматривать более сложные концепции.

## Дополнительные ресурсы

- [Официальная документация Rust](https://doc.rust-lang.org/)
- [Книга "The Rust Programming Language"](https://doc.rust-lang.org/book/)
- [Rust by Example](https://doc.rust-lang.org/rust-by-example/)
- [Rust Playground](https://play.rust-lang.org/) — онлайн-среда для экспериментов с Rust