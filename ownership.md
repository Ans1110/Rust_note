# Rust Ownership

## rust ownership rules

**1.每個值都有一個所有者(owner)**

```rust
    let s1 = String::from("hello");
```

**2.一個值在同一時間只能有一個所有者**

```rust
    let s1 = String::from("hello");
    let s2 = s1;
```

**3.當所有者離開作用域時，值會被釋放(drop)**

rust中的作用域不只是語法塊{}，也是"最後一次使用的位置"

```rust
    {
        let s = String::from("hello");
    } // s離開作用域，值會被釋放(drop)
```

*move vs copy*

```rust
    let s1 = String::from("hello");
    let s2 = s1; //move(所有權轉移)
    println!("{}", s2); //ok
    println!("{}", s1); //error s1的所有權已經轉移到s2
```

**copy**

```rust
    let s1 = String::from("hello");
    let s2 = s1.clone(); //copy
    println!("{}", s2); //ok
    println!("{}", s1); //ok
```

**哪些型別會被move**

    1. String
    2. Vec<T>
    3. Box<T>

**如何避免move**

1. 使用引用(reference)
2. 使用深拷貝(clone)

```rust
    let s1 = String::from("hello");
    let s2 = s1.clone();
```

**借用(borrowing)及引用(reference):**

1. 借用是概念，引用是語法
2. 借用是透過 & 取得一個值的引用
3. 借用不會奪走值的所有權
4. 借用有三種形式:
    
| 類型    | 說明                  | 用法               |
|---------|----------------------|-------------------|
| &T      | 不可變引用（只能讀）   | let r = &x        |
| &mut T  | 可變引用（可讀寫）     | let r = &mut x    |
| &&T     | 引用的引用（不常用）   | let r = &&x       |

5. 借用規則:
    1. 同一時間只能有一個可變借用
    2. 同一時間只能有多个不可變借用
    3. 不可變借用可以和可變借用共存
6. 解引用(dereference):
    1. 解引用是透過 *取得一個值的引用
    2. 解引用可以透過解引用運算子(*)或解引用方法(deref)來實現
    3. 解引用後更改的是"被引用的原始資料"
  
```rust
    let mut s = String::from("hello");
    let r1 = &mut s;
    *r1 = String::from("world");
    println!("{}", s);//output: world
```

**那改變引用變量本身呢？ans: 你不能“修改引用”去改變它指向什麼（引用一旦建立，它就固定指向那個對象），只能通過變量重新綁定到新的引用**

```rust
    let x = 5;
    let y = 10;
    
    let mut r1 = &x;
    r1 = &y;//correct 這是“重新綁定 r”，不是“修改引用”
```

**借用的應用:**

1. 借用可以避免值的所有權轉移
2. 借用可以避免值的複製(傳遞大型資料時)
3. 借用可以避免值的釋放(drop)

**& 背後是什麼？（底層結構）**

```rust
    let x = 42;
    let r: &i32 = &x;
```

**代表:**

1. r是一個"read-only"的pointer
2. compiler會保證x活著的時候，r不會成為悬垂引用(dangling references)
    - dangling references:
    - 指向已經釋放(drop)或不存在的記憶體
    - rust會防止dangling references的產生

```rust
    fn dangle() -> &String {
        let s = String::from("hello");
        &s //error: s will be dropped at the end of the function
    }
        
    fn main() {
        let r = dangle();
    }
        
    //correct version
    fn no_dangle() -> String {
        let s = String::from("hello");
        s //return ownership
    }
```

**3. 透過生命週期(lifetime)保證**

**與C的指標差別：**

| 面向 | Rust 的 & | C 的指標 * |
|------|-----------|-----------|
| 安全性 | 編譯期檢查 | 不檢查 |
| Null 可能性 | 不可能為 null | 指標可能為 null |
| 自動記憶體管理 | 有（RAII） | 無 |
| 所有權概念 | 有 | 無 |

# 與stack和heap的關連:
**值儲存在哪裡，會影響所有權行為**

1. Stack 資料：Copy，不會 Move

```rust
    let x = 5;
    let y = x;
```

- x和y儲存在stack
- 這類型實現了Copy trait
- 沒有所有權問題，不會觸發move

2. Heap 資料：Move

```rust
    let s1 = String::from("hello");
    let s2 = s1;
```

- String本身儲存在stack，但它指向的資料(真正的字串)儲存在heap
- Move是為了避免雙重釋放(double free)問題

**與 Borrowing / Reference 的關係:**

借用就是傳遞「stack 上的指標」，不會 move 或 clone 整個 heap 資料

```rust
    let r = String::from("hi");// String: stack上的指標 -> heap上的資料
    let s = &r;// r是stack上的指標，指向heap上的資料
    println!("{}", s); //ok
```

    Stack:               Heap:
    +----------+         +-----+-----+
    | s        | ----->  | 'h' | 'i' |
    +----------+         +-----+-----+
    |         
    +-- stack 上是三個欄位(type: usize): 指標 + 長度 + 容量(pointer + length + capacity)
    |         
    +-- heap 上是真正的資料
    
