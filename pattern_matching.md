# Pattern Matching

## match

```rust
    //基本結構
    match value {
        pattern if condition => expression,
        pattern => expression,
        _ => expression,//_是"catch-all"模式(like default in switch-case)
    }
```

**match的特性:**

1. match是"exhaustive"的，必須處理所有可能的值
2. match的每個分支都必須有"expression"
3. match的每個分支的"expression"的型別必須相同

**match的應用:**

在enum中使用match

```rust
    enum Message {
        Hello(i32),
    }

    fn main(){
        let msg = Message::Hello(5);
        match msg {
            Message::Hello(n) => println!("Hello {}", n),
            _ => ()
        }
    }
```

**match guard:**

match guard 是用來在 match 的基礎上加「額外條件判斷」的工具。

```rust
    enum Message {
        Hello(i32),
    }

    fn main(){
        let msg = Message::Hello(5);
        match msg {
            Message::Hello(n) if n > 5 => println!("Hello {}", n),
            _ => ()
        }
    }
```

**什麼時候需要match guard?**

1. 當pattern不能表達所有可能的值時
2. 某個 match 分支在語法上可以匹配成功（比如 Some(x)）
3. 但你只想處理 其中某些值的情況
4. 這時就可以用 if 搭配條件，做更細的分類

```rust
    let num = Some(4);

    match num {
        Some(x) if x < 5 => println!("smaller than 5"),
        Some(x) => println!("greater than 5"),
        None => println!("None"),
    }
```

*如果沒有match guard*

```rust
    match num {
        Some(x) => {
            if x < 5 {
                println!("smaller than 5");
            }
        } else {
            println!("greater than 5");
        },
        None => println!("None"),
    }
```

**match + reference:**

```rust
    let mut x = String::from("hello");
    let r = &mut x;
    
    match r {
        value => value.push_str("world"),//value是&mut String
        /*
        rust 會自動解引用
        等同於:
        value => (*value).push_str("world"),
        */
        _ => ()
    }
```

## if let

if let是match的簡化版本，用於處理簡單的模式匹配(只處理某一個pattern)

```rust
    //match
    let value = Some(5);
    match value {
        Some(5) => println!("5"),
        _ => ()
    }

    //if let
    let value = Some(5);
    if let Some(5) = value {//發生variable shadowing
        println!("5");
    }
```

**if let跟match的variable shadowing:**
無論match還是if let，都會發生variable shadowing

```rust
    //variable shadowing
    let x = 5;
    let x = 6;//發生variable shadowing
    println!("{}", x); //output: 6
```

*match跟if let的variable shadowing*

```rust
    //match
    fn main() {
        let x = Some(5);
        println!("{:?}", x);//Some(5)
        match x {//發生variable shadowing
            Some(x) => println!("{}", x),//5
            _ => ()
        }
        println!("{:?}", x);//Some(5)
    }
```

```rust
    //if let
    fn main() {
        let x = Some(5);
        println!("{:?}", x);//Some(5)
        if let Some(x) = x {//發生variable shadowing
            println!("{}", x);//5
        }
        println!("{:?}", x);//Some(5)
    }
```

## while let

只要pattern匹配成功，就會一直執行

```rust
    let mut stack = Vec::new();
    stack.push(1);
    stack.push(2);
    stack.push(3);
    
    while let Some(x) = stack.pop() {
        println!("{}", x);
    }
```

## @(at) pattern

@　允許我們在模式中綁定一個值，並同時進行模式匹配

```rust
    enum Message {
        Hello { id: i32 }
    }
    
    let msg = Message::Hello { id: 5 };
    match msg {
        Message::Hello { id } @ 1..=5 => println!("id: {}", id),//output: id: 5
        _ => ()
    }
```

```rust
    match 1 {
        num @ (1 | 2) => {
            println!("{}", num);
        },
        _ => ()
    }
```

@前綁定後解構

```rust
    #[derive(Debug)]
    struct Point { x: i32, y: i32 }
    
    fn main() {
        //綁定一個值，並同時解構
        let p @ Point { x: px, y: py } = Point { x: 1, y: 2 };
        println!("{:?}", p);
        println!("x: {}, y: {}", px, py);

        let point = Point { x:1, y:2 };
        if let p @ Point { x: 10, y } = point {
            println!("x is 10, y is {} in {:?}", y, p);
        } else {
            println!("x is not 10, y is {} in {:?}", y, p);
        }
    }
```

## All pattern matching

| 類型    | 範例               | 說明                  |
|---------|--------------------|-----------------------|
| 字面值  | 1, "abc"           | 比對具體值            |
| 變數綁定| x, value           | 將值綁定到變數        |
| 通配符  | _                  | 匹配任何值但忽略      |
| 或模式  | 1 \| 2             | 匹配多個選項          |
| 範圍模式| 1..=5              | 匹配範圍值            |
| 解構結構體| Point { x, y }     | 拆開 struct 成員      |
| 解構列舉| Enum::Variant(data) | 拆開 enum 成員        |
| 解構元組| (x, y)             | 拆開 tuple 成員       |
| 陣列／切片| [a, b], [head, ..] | 部分比對固定或變長陣列 |
| 忽略模式| .., _              | 忽略某些資料          |
| 嵌套模式| Some(Some(x))      | 結構內的結構          |
| 守衛條件| if x > 5           | 加上條件限制          |
| @ 綁定模式| id @ 1..=5         | 同時比對並保留原始值   |
| 參考模式| &Some(x)           | 比對 reference 資料   |

*example*

1. Literal pattern

```rust
    let x = 5;
    match x {
        5 => println!("5"),
        _ => ()
    }
```

2. Variable binding pattern

```rust
    let x = 5;
    match x {
        val => println!("{}", val),
        _ => ()
    }
```

3. Wildcard pattern

```rust
    let x = 5;
    match x {
        _ => println!("5"),
    }
```

4. Or pattern

```rust
    let x = 5;
    match x {
        1 | 2 => println!("1 or 2"),
        _ => ()
    }
```

5. Range pattern

```rust
    let x = 5;
    match x {
        1..=5 => println!("1 to 5"),
        _ => ()
    }
```

6. Destructuring pattern

```rust
    struct Point { x: i32, y: i32 }
    
    match p {
        Point { x, y: 0 } => println!("x: {}", x),
        Point { x: 0, y } => println!("y: {}", y),
        _ => ()
    }
```

7. Destructuring enum pattern

```rust
    enum Message {
        Quit,
        Move { x: i32, y: i32 },
        Write(String),
        ChangeColor(i32, i32, i32),
    }
    
    match msg {
        Message::Quit => println!("Quit"),
        Message::Move { x, y } => println!("Move ({}, {})", x, y),
        Message::Write(msg) => println!("Write ({})", msg),
        Message::ChangeColor(r, g, b) => println!("ChangeColor ({}, {}, {})", r, g, b),
    }
```

8. Tuple pattern

```rust
    let x = (1, 2);
    match x {
        (1, 2) => println!("1, 2"),
        _ => ()
    }
```

9. Array/slice pattern

```rust
    let arr = [1, 2, 3];
    match arr {
        [1, _, _] => println!("head is 1"),
        [a, b, c] => println!("{} {} {}", a, b, c),
        _ => ()
    }
```

10. Ignore pattern

```rust
    match arr {
        [first, .., last] => println!("{} {}", first, last),
        _ => ()
    }
```

11. Nested pattern

```rust
    enum Option {
        Some(i32),
        None,
    }
    let x = Some(Some(5));
    match x {
        Some(Some(y)) => println!("{}", y),
        _ => ()
    }
```

12. Guard pattern

```rust
    let x = 5;
    match x {
        1 => println!("1"),
        2 => println!("2"),
        _ if x > 2 => println!("greater than 2"),
        _ => ()
    }
```

13. @ pattern

```rust
    let msg = Message::Hello(5);
    match msg {
        Message::Hello(id @ 1..=5) => println!("id: {}", id),
        _ => ()
    }
```

14. Reference pattern

```rust
    let x = &Some(5);
    match x {
        &Some(y) => println!("{}", y),
        _ => ()
    }
```

## refutable pattern(可辯駁) and irrefutable pattern(不可辯駁)

*refutable pattern: 可能失敗的match*

```rust
    if let Some(y) = x {   // OK：可能成功也可能失敗
        println!("{}", y);
    }
```

*irrefutable pattern: 一定成功的match*

```rust
    let x = Some(5);      // OK：`let` 只接受不可辯駁模式
    let Some(y) = x;      // error！因為 `Some(y)` 是可辯駁的
```

**refutable pattern vs irrefutable pattern**

| 語法 | 接受模式類型 |
|------|------------|
| let | 不可辯駁 |
| if let | 可辯駁 |
| while let | 可辯駁 |
| match | 可辯駁 + 多分支 |
| 函式參數 | 不可辯駁 |
| for 解構 | 不可辯駁 |
