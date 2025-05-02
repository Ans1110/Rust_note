# Rust Ownership

rust ownership rules:
    1.每個值都有一個所有者(owner)

    ```rust
    let s1 = String::from("hello");
    ```

    2.一個值在同一時間只能有一個所有者

    ```rust
    let s1 = String::from("hello");
    let s2 = s1;
    ```

    3.當所有者離開作用域時，值會被釋放(drop) #rust中的作用域不只是語法塊{}，也是"最後一次使用的位置"

    ```rust
    {
        let s = String::from("hello");
    } // s離開作用域，值會被釋放(drop)
    ```

    move vs copy

    ```rust
    let s1 = String::from("hello");
    let s2 = s1; //move(所有權轉移)
    println!("{}", s2); //ok
    println!("{}", s1); //error s1的所有權已經轉移到s2
    ```

    copy

    ```rust
    let s1 = String::from("hello");
    let s2 = s1.clone(); //copy
    println!("{}", s2); //ok
    println!("{}", s1); //ok
    ```

    哪些型別會被move
    1. String
    2. Vec<T>
    3. Box<T>

借用(borrowing)及引用(reference):
    1.借用是概念，引用是語法
    2.借用是透過 & 取得一個值的引用
    3.借用不會奪走值的所有權
    4.借用有三種形式:
        1.不可變借用(&T)
        2.可變借用(&mut T)
        3.引用(&T)
    5.借用規則:
        1.同一時間只能有一個可變借用
        2.同一時間只能有多个不可變借用
        3.不可變借用可以和可變借用共存
