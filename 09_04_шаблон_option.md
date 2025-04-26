# Шаблон Option<T>

В предыдущем разделе мы кратко упомянули перечисление `Option<T>` из стандартной библиотеки Rust. В этом разделе мы рассмотрим его более подробно и узнаем, почему оно является одним из наиболее важных и часто используемых типов в Rust.

## Проблема null-значений

Во многих языках программирования существует концепция `null` или `nil` - специальное значение, которое указывает на отсутствие значения или ссылки. Хотя эта концепция кажется полезной, она может привести к множеству ошибок, если программист забудет проверить значение на `null` перед его использованием.

Тони Хоар, изобретатель концепции null-ссылки, назвал это своей "ошибкой на миллиард долларов". Проблема в том, что когда `null` может появиться в любом месте программы, очень легко забыть о его проверке, что приводит к ошибкам времени выполнения.

Rust решает эту проблему, не имея значения `null`. Вместо этого в стандартной библиотеке есть перечисление `Option<T>`, которое может быть использовано для представления наличия или отсутствия значения.

## Определение Option<T>

Перечисление `Option<T>` определено в стандартной библиотеке следующим образом:

```rust
enum Option<T> {
    None,    // Значение отсутствует
    Some(T), // Значение присутствует
}
```

Здесь `T` - это обобщенный тип, который может быть любым типом данных. Это означает, что `Option<T>` может представлять отсутствие или наличие значения любого типа.

Перечисление `Option<T>` настолько часто используется, что оно включено в прелюдию (prelude) Rust, поэтому его не нужно явно импортировать. Кроме того, варианты `Some` и `None` также доступны без префикса `Option::`.

## Использование Option<T>

Вот несколько примеров использования `Option<T>`:

```rust
fn main() {
    // Option с целым числом
    let some_number = Some(5);
    
    // Option со строкой
    let some_string = Some("строка");
    
    // Option без значения (нужно указать тип, так как компилятор не может его вывести)
    let absent_number: Option<i32> = None;
    
    // Использование match для обработки Option
    match some_number {
        Some(n) => println!("Число: {}", n),
        None => println!("Число отсутствует"),
    }
}
```

## Безопасность типов с Option<T>

Одним из ключевых преимуществ `Option<T>` является то, что компилятор Rust заставляет вас явно обрабатывать случай отсутствия значения. Вы не можете использовать значение типа `Option<T>` напрямую, как если бы это было значение типа `T`:

```rust
fn main() {
    let x: i8 = 5;
    let y: Option<i8> = Some(5);
    
    // Это не скомпилируется, потому что i8 и Option<i8> - разные типы
    // let sum = x + y;
    
    // Нужно явно обработать случай None
    let sum = x + match y {
        Some(value) => value,
        None => 0, // Значение по умолчанию, если y равно None
    };
    
    println!("Сумма: {}", sum);
}
```

Это заставляет программиста явно обрабатывать случай отсутствия значения, что предотвращает ошибки, связанные с null-значениями.

## Методы Option<T>

Перечисление `Option<T>` имеет множество полезных методов, которые упрощают работу с ним. Вот некоторые из наиболее часто используемых:

### is_some() и is_none()

Методы для проверки, содержит ли `Option` значение или нет:

```rust
fn main() {
    let some_number = Some(5);
    let no_number: Option<i32> = None;
    
    if some_number.is_some() {
        println!("some_number содержит значение");
    }
    
    if no_number.is_none() {
        println!("no_number не содержит значения");
    }
}
```

### unwrap()

Метод `unwrap()` извлекает значение из `Option<T>`. Если `Option` равен `None`, `unwrap()` вызывает панику:

```rust
fn main() {
    let some_number = Some(5);
    let value = some_number.unwrap(); // Вернет 5
    
    println!("Значение: {}", value);
    
    // Это вызовет панику!
    // let no_number: Option<i32> = None;
    // let value = no_number.unwrap();
}
```

Из-за возможности паники `unwrap()` обычно используется только в прототипах, тестах или когда вы абсолютно уверены, что `Option` содержит значение.

### expect()

Метод `expect()` похож на `unwrap()`, но позволяет указать сообщение об ошибке, которое будет показано в случае паники:

```rust
fn main() {
    let some_number = Some(5);
    let value = some_number.expect("Ожидалось число, но получен None");
    
    println!("Значение: {}", value);
}
```

### unwrap_or()

Метод `unwrap_or()` извлекает значение из `Option<T>` или возвращает значение по умолчанию, если `Option` равен `None`:

```rust
fn main() {
    let some_number = Some(5);
    let no_number: Option<i32> = None;
    
    let value1 = some_number.unwrap_or(0); // Вернет 5
    let value2 = no_number.unwrap_or(0);   // Вернет 0
    
    println!("value1: {}, value2: {}", value1, value2);
}
```

### unwrap_or_else()

Метод `unwrap_or_else()` похож на `unwrap_or()`, но вместо значения по умолчанию принимает функцию, которая будет вызвана для получения значения по умолчанию:

```rust
fn main() {
    let some_number = Some(5);
    let no_number: Option<i32> = None;
    
    let value1 = some_number.unwrap_or_else(|| 0);          // Вернет 5
    let value2 = no_number.unwrap_or_else(|| {
        println!("Вычисляем значение по умолчанию...");
        42
    }); // Вернет 42
    
    println!("value1: {}, value2: {}", value1, value2);
}
```

### map()

Метод `map()` применяет функцию к значению внутри `Some` и возвращает новый `Option` с результатом. Если `Option` равен `None`, `map()` просто возвращает `None`:

```rust
fn main() {
    let some_number = Some(5);
    let no_number: Option<i32> = None;
    
    let doubled = some_number.map(|x| x * 2); // Вернет Some(10)
    let still_none = no_number.map(|x| x * 2); // Вернет None
    
    println!("doubled: {:?}, still_none: {:?}", doubled, still_none);
}
```

### and_then()

Метод `and_then()` (также известный как `flatmap` в других языках) применяет функцию, которая сама возвращает `Option`, к значению внутри `Some` и возвращает результат. Если исходный `Option` равен `None`, `and_then()` просто возвращает `None`:

```rust
fn square_root(x: i32) -> Option<f64> {
    if x >= 0 {
        Some((x as f64).sqrt())
    } else {
        None
    }
}

fn main() {
    let some_number = Some(16);
    let negative_number = Some(-4);
    let no_number: Option<i32> = None;
    
    let root1 = some_number.and_then(square_root);      // Вернет Some(4.0)
    let root2 = negative_number.and_then(square_root);  // Вернет None
    let root3 = no_number.and_then(square_root);        // Вернет None
    
    println!("root1: {:?}, root2: {:?}, root3: {:?}", root1, root2, root3);
}
```

### filter()

Метод `filter()` возвращает `None`, если `Option` равен `None` или если предикат, примененный к значению внутри `Some`, возвращает `false`. В противном случае возвращает исходный `Option`:

```rust
fn main() {
    let some_number = Some(5);
    let no_number: Option<i32> = None;
    
    let even = some_number.filter(|x| x % 2 == 0); // Вернет None, так как 5 нечетное
    let odd = some_number.filter(|x| x % 2 != 0);  // Вернет Some(5), так как 5 нечетное
    let still_none = no_number.filter(|x| x % 2 == 0); // Вернет None
    
    println!("even: {:?}, odd: {:?}, still_none: {:?}", even, odd, still_none);
}
```

### or() и or_else()

Методы `or()` и `or_else()` возвращают исходный `Option`, если он равен `Some`, или альтернативный `Option`, если исходный равен `None`:

```rust
fn main() {
    let some_number = Some(5);
    let no_number: Option<i32> = None;
    
    let result1 = some_number.or(Some(10)); // Вернет Some(5)
    let result2 = no_number.or(Some(10));   // Вернет Some(10)
    
    let result3 = some_number.or_else(|| Some(10)); // Вернет Some(5)
    let result4 = no_number.or_else(|| {
        println!("Вычисляем альтернативное значение...");
        Some(42)
    }); // Вернет Some(42)
    
    println!("result1: {:?}, result2: {:?}", result1, result2);
    println!("result3: {:?}, result4: {:?}", result3, result4);
}
```

## Комбинирование методов Option<T>

Методы `Option<T>` можно комбинировать для создания сложных цепочек обработки:

```rust
fn main() {
    let numbers = vec![Some(1), None, Some(3), None, Some(5)];
    
    let sum: i32 = numbers
        .iter()
        .filter_map(|&x| x)  // Отфильтровывает None и извлекает значения из Some
        .filter(|&x| x % 2 != 0)  // Оставляет только нечетные числа
        .map(|x| x * x)  // Возводит каждое число в квадрат
        .sum();  // Суммирует все числа
    
    println!("Сумма квадратов нечетных чисел: {}", sum);  // Выведет 35 (1^2 + 3^2 + 5^2)
}
```

## Примеры использования Option<T>

### Пример 1: Поиск элемента в коллекции

```rust
fn find_user_by_id(id: u32, users: &[User]) -> Option<&User> {
    for user in users {
        if user.id == id {
            return Some(user);
        }
    }
    None
}

struct User {
    id: u32,
    name: String,
    email: String,
}

fn main() {
    let users = vec![
        User { id: 1, name: String::from("Алиса"), email: String::from("alice@example.com") },
        User { id: 2, name: String::from("Боб"), email: String::from("bob@example.com") },
        User { id: 3, name: String::from("Чарли"), email: String::from("charlie@example.com") },
    ];
    
    let user_id = 2;
    match find_user_by_id(user_id, &users) {
        Some(user) => println!("Пользователь найден: {} ({})", user.name, user.email),
        None => println!("Пользователь с ID {} не найден", user_id),
    }
}
```

### Пример 2: Парсинг строки в число

```rust
fn parse_age(age_str: &str) -> Option<u32> {
    match age_str.parse::<u32>() {
        Ok(age) => Some(age),
        Err(_) => None,
    }
}

fn main() {
    let inputs = vec!["25", "тридцать", "42", "-5"];
    
    for input in inputs {
        match parse_age(input) {
            Some(age) => println!("Возраст: {} лет", age),
            None => println!("'{}' не является допустимым возрастом", input),
        }
    }
}
```

### Пример 3: Цепочка вычислений с возможными ошибками

```rust
fn divide(a: f64, b: f64) -> Option<f64> {
    if b == 0.0 {
        None
    } else {
        Some(a / b)
    }
}

fn square_root(x: f64) -> Option<f64> {
    if x < 0.0 {
        None
    } else {
        Some(x.sqrt())
    }
}

fn compute(a: f64, b: f64, c: f64) -> Option<f64> {
    // Вычисляем (a / b) + sqrt(c)
    divide(a, b).and_then(|result| {
        square_root(c).map(|sqrt_c| result + sqrt_c)
    })
}

fn main() {
    let cases = vec![
        (10.0, 2.0, 16.0),    // Должно вернуть Some(9.0)
        (10.0, 0.0, 16.0),    // Должно вернуть None (деление на ноль)
        (10.0, 2.0, -16.0),   // Должно вернуть None (отрицательный корень)
    ];
    
    for (a, b, c) in cases {
        match compute(a, b, c) {
            Some(result) => println!("({} / {}) + sqrt({}) = {}", a, b, c, result),
            None => println!("Невозможно вычислить ({} / {}) + sqrt({})", a, b, c),
        }
    }
}
```

## Заключение

Перечисление `Option<T>` - это мощный инструмент в Rust для работы с возможным отсутствием значения. Оно заставляет программиста явно обрабатывать случай отсутствия значения, что предотвращает ошибки, связанные с null-значениями.

Ключевые преимущества использования `Option<T>`:

1. **Безопасность типов**: Компилятор гарантирует, что вы обработали случай отсутствия значения.
2. **Выразительность**: Код явно показывает, что функция может не вернуть значение.
3. **Богатый API**: Множество методов для удобной работы с возможным отсутствием значения.
4. **Композиция**: Возможность создавать сложные цепочки обработки с помощью комбинирования методов.

В следующем разделе мы рассмотрим практическое применение структур, методов, перечислений и шаблона `Option<T>` для моделирования предметной области.

## Упражнения

1. Напишите функцию `find_max`, которая принимает срез целых чисел и возвращает `Option<i32>` - максимальное значение или `None`, если срез пуст.

2. Реализуйте функцию `safe_division`, которая принимает два числа с плавающей точкой и возвращает `Option<f64>` - результат деления или `None`, если делитель равен нулю.

3. Создайте структуру `Person` с полями `name` и `age`, где `age` имеет тип `Option<u32>`. Реализуйте метод `is_adult`, который возвращает `Option<bool>` - `Some(true)`, если возраст известен и больше или равен 18, `Some(false)`, если возраст известен и меньше 18, или `None`, если возраст неизвестен.

4. Напишите функцию `find_by_key`, которая принимает вектор пар `(K, V)` и ключ типа `K`, и возвращает `Option<&V>` - ссылку на значение, соответствующее ключу, или `None`, если ключ не найден.

5. Реализуйте функцию `chain_operations`, которая принимает число и выполняет следующие операции: умножает на 2, вычитает 10, берет квадратный корень и делит на 3. Каждая операция может завершиться неудачей (например, отрицательный корень), поэтому функция должна возвращать `Option<f64>`. Используйте методы `map` и `and_then` для цепочки операций.