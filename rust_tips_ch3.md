# Rust tips - #2
### Reference
* 実践Rustプログラミング入門
  * 初田直也【著】/山口聖弘【著】/吉川哲史【著】/豊田優貴【著】/松本健太郎【著】/原将己【著】

## 基本的な型
### 文字列型
strは変更できないプリミティブ型、Stringは変更できる型
```rust
fn main() {
    // str: 文字列スライス＝変更できない文字列
    // 変数s0は、テキストが存在するメモリ上の位置を指す
    // つまり、定義したテキストが入る大きさしか割当られていないため変更不可
    let s0: &str = "this is str";
    println!("{}", s0); // this is str

    // String: 変更できる文字列
    let mut s1: String = String::from("this is string");
    println!("{}", s1); // this is string
    let s2: String = String::from(", added text");
    s1 += &s2;
    println!("{}", s1); // this is string, added text
    println!("{}", s2); // , added text

    // strとStringの変換
    let s3: &str = &s1; // String => &strに変換
    println!("{}", s3); // this is string, added text

    let s4: String = s1.to_string(); // &str -> Stringに変換
    println!("{}", s4); // this is string, added text
}
```

### タプル
```rust
fn main() {
    // タプル：異なる型を収められる集合
    // タブル内の方は後から変えることは不可
    let mut t =(1, "No.2");
    t.0 = 2;
    t.1 = "No.3";
    println!("{:?}", t); // (2, "No.3")
}
```

### 配列
Rustの配列は固定長であるため、宣言後にサイズの伸縮は不可
```rust
fn main() {
    // 配列は、全要素が同じ型である必要あり
    let mut array1: [i32; 3] = [0, 1, 2];
    let mut array2: [i32; 3] = [0; 3];
    println!("{:?}{:?}", array1, array2); // [0, 1, 2][0, 0, 0]

    array1[1] = array2[1] + 10;
    array1[2] = array2[2] + 10;
    println!("{:?}", &array1[1..3]); // [10, 10]
}
```

### 構造体、列挙型
```rust
fn main() {
    // 構造体
    struct Vehicle {
        name: String,
        price: u32,
    }
    let v = Vehicle {
        name: String::from("Prius"),
        price: 3500000,
    };
    println!("{:?}, {:?}", v.name, v.price); // "Prius", 3500000

    // 列挙型
    // enum型は、デフォルトではDebugトレイトが実装されていないため必要
    #[derive(Debug)]
    enum Event {
        Push,
        KeyDown(u8),
        MouseDown { x: i32, y: i32 },
    }

    let enum1 = Event::Push;
    let enum2 = Event::KeyDown(2);
    let enum3 = Event::MouseDown { x: 10, y: 20 };

    // Push, KeyDown(2), MouseDown { x: 10, y: 20 }
    println!("{:?}, {:?}, {:?}", enum1, enum2, enum3);
}
```

### Option
Option型は、取得できないかもしれない値を表現
値がないNone（エラーという意味ではない）と、何か値があることを示すSomeのいずかの値を取る
エラーを表す場合はResultの方が適切
```rust
pub enum Option<T> {
    None,
    Some(T),
}
```
```rust
fn matcher(x: Option<usize>) {
    // Option型はSome, Noneの両方の定義が必要
    match x {
        Some(value) => println!("{}", value),
        None => println!("None value"),
    };
}

fn main() {
    let x: Option<usize> = Some(10);
    matcher(x); // 10

    let none: Option<usize> = None;
    matcher(none); // None value

    // if let: 値が1つのパターンにマッチしたときの動作を書ける
    if let Some(value) = x {
        println!("{}", value); // 10
    }
}
```

### Result
Result型は、処理の結果が成功orエラーを表現
```rust
pub enum Result<T, E> {
    Ok(T),
    Err(E),
}
```
```rust
fn matcher(x: &Result<i32, String>) {
    // Result型はOk, Errの両方の定義が必要
    match x {
        Ok(value) => println!("{}", value),
        Err(error) => println!("{}", error),
    };
}

fn main() {
    // --- metcher ---
    let x: Result<i32, String> = Ok(200);
    matcher(&x); // 200

    let err: Result<i32, String> = Err("error".to_string());
    matcher(&err); // error

    // --- unwrap_or ---
    // unwrap_or: Okの場合は値を取り出し、Errの場合はデフォルト値を返す
    // matcher関数の引数を参照(&)とすることで、所有権のエラーを回避
    println!("{}", x.unwrap_or(200));

    // --- and_then ---
    // valueを受け取って、Result型で常にOk(100)を返す関数
    fn func(value: i32) -> Result<i32, String> {
        println!("code: {}", value);
        Ok(100)
    }
    let x2: Result<i32, String> = Ok(100);

    // and_then: Result型の値に関数を適用し、その結果を新しいResult型として返す
    // x2がOkの場合、funcが呼び出される。Errの場合は呼び出されない
    let next_result = x2.and_then(func);
}
```

### Vec
配列と違い、内部に収められる要素の数を増減できる
```rust
fn main() {
    // println!同様、!はマクロ呼び出しを意味する
    // 可変個の引数を取るマクロ
    let mut vec1 = vec![1, 2, 3, 4, 5];

    println!("{:?}", vec1); // [1, 2, 3, 4, 5]
    println!("{:?}", vec1[2]); // 3

    // Vecはpop, pushがサポート
    vec1.pop();
    println!("{:?}", vec1); // [1, 2, 3, 4]
    vec1.push(5);
    println!("{:?}", vec1); // [1, 2, 3, 4, 5]

    let mut vec2 = vec![0; 5];
    println!("{:?}", vec2); // [0, 0, 0, 0, 0]

    for element in &vec2 {
        println!("{}", element); // [0, 0, 0, 0, 0]
    }
}
```

### スタック領域とヒープ領域
* スタック領域
    * コンパイラやOSが自動で割当・解放を実施
    * 関数内の変数やアドレスに対して割当
    * スタックのように積み上げでメモリを確保し、開放時は上から順番に実施
    * **動作がシンプルで高速であるが、固定サイズである必要あり**
* ヒープ領域
    * 動的にメモリ領域の確保・解放を実施
    * **コンパイル時にサイズが分かっていなくても確保したいタイミングで必要な分を確保**

### Box
ヒープ領域に確保される形式
そのため、サイズがわからない型を格納したり、ポインタでのデータ渡しが可能
```rust
fn print(num: Box<[i32]>) {
    println!("{:?}", num);
}

fn main() {
    let array1 = [1, 2, 3];
    // printへポインタを渡しているため、どのようなサイズでも渡せる
    print(Box::new(array1)); // [1, 2, 3]

    let array1 = [4, 5, 6, 7, 8];
    print(Box::new(array1)); // [4, 5, 6, 7, 8]
}
```

## 変数宣言
### letとmut
```rust
fn main() {
    // 数字の方の場合は、型を指定することが可能
    let immutable_value: i32 = 10;
    let mut mutable_value = 100;

    mutable_value += immutable_value;
    println!("{}", mutable_value); // 110
}
```

### if
if文で評価した値は、変数の束縛や関数の引数にできる
```rust
fn main() {
    let value = 10;
    if 0 > value {
        println!("under 0");
    } else if 0 < value {
        println!("over 0"); // pass
    } else {
        println!("0");
    }

    // if文を式として使用
    let result = if 0 < value {
        value * 1000
    } else {
        value * -1000
    };

    println!("{}", result); // 10000
}
```

// loop, while, for
```rust
fn main() {
    // --- loop ---
    let mut loop_cnt = 0;

    // loopも式なので値を返せる
    let result = loop {
        println!("{:?}", loop_cnt); // 0, 1, 2, 3, 4
        loop_cnt += 1;

        // loopを抜け出すときはbreak
        if loop_cnt == 5 {
            break loop_cnt;
        }
    };

    // --- while ---
    let mut while_cnt = 0;

    while while_cnt < 5 {
        println!("{}", while_cnt); // 0, 1, 2, 3, 4
        while_cnt += 1;
    }

    // --- for ---
    let for_cnt: i32;

    for for_cnt in 0..5 {
        println!("{}", for_cnt); // 0, 1, 2, 3, 4
    }

    let nums = [100, 101, 102];

    for element in &nums {
        println!("{}", element); // 100, 101, 102
    }
}
```

### match
matchはif文よりも高度なパターンマッチが可能
数字や文字列の比較だけではなく、列挙型やタプル、構造体などの比較も可能
```rust
fn main() {
    let value: i32 = 100;

    match value {
        100 => println!("100"),
        _ => println!("Not 100"),
    }
}
```

列挙型での例
また、結果を変数に格納もできる
```rust
enum Vehicle {
    Prius,
    LandCruiser,
}

fn main() {
    let vehicle = Vehicle::Prius;

    let res = match vehicle {
        Vehicle::Prius => "Prius!",
        Vehicle::LandCruiser => "Land Cruiser!",
    };

    println!("{}", res); // Prius!
}

```

### Range
特定範囲の数字の指定
```rust
fn main() {
    for num in 1..3 {
        println!("{}", num); // 1, 2
    }
}
```