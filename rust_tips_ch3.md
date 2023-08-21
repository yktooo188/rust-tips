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

### Iterator
オブジェクトを順番に取り扱うことができる
nextの実装が必要
```rust
pub trait Iterator {
    type Item;

    // Required method
    fn next(&mut self) -> Option<Self::Item>;
}
```
```rust
fn main() {
    let iterator = Iter { current: 0, max: 5 };

    for num in iterator {
        println!("{}", num); // 0, 1, 2, 3, 4
    }
}

// 自作の構造体
struct Iter {
    current: usize,
    max: usize,
}

// 構造体Iterに対して、Iteratorを適用
impl Iterator for Iter {
    type Item = usize; // イテレータが返す型が必要。usizeは符号なし整数

    // 復習：Option型は、取得できないかもしれない値を表現
    // イテレーションを進め、次の要素を取得するためにnextが必要
    fn next(&mut self) -> Option<usize> {
        self.current += 1; // まずは、現在の値を1増やす
        if self.current - 1 < self.max {
            // maxより小さいとき
            Some(self.current - 1) // イテレータの次の要素を返すにはSome, Noneが一般的
        } else {
            None
        }
    }
}
```

## 関数
### fn
```rust
fn main() {
    let x1 = multiply(10, 20);
    println!("{}", x1); // 30

    let x2 = abs(-100);
    println!("{}", x2); // 100
}

fn multiply(val1: i32, val2: i32) -> i32 {
    // 関数内の最後にセミコロン無しで記述された値が戻り値
    val1 + val2
}

fn abs(val: i32) -> i32 {
    if val < 0 {
        // 処理の途中で戻り値を返したい時はreturnを使用
        return -val;
    }
    val
}
```

### impl
* implによって構造体にメソッドを加えられる
* メソッドの戻り値に自分自身の方を指定すると、メソッドチェーンが使える
* 関連関数はコンストラクタや、同じ構造体の関数をまとめられる
```rust
struct Vehicle {
    name: String,
    price: f32,
    unit: String,
}

// implで構造体にメソッドを加える
// データに関係のある関数を紐付けておけば、オブジェクト指向のクラスのように扱える
impl Vehicle {
    fn output_name(&self) {
        println!("Thic vehicle is {}.", self.name);
    }

    // 戻り値をselfにすることで、メソッドチェーンが使用可能
    fn calc_price(&self) -> &Self {
        print!("The price is {} ", self.price * 1.10);
        self
    }

    fn output_unit(&self) -> &Self {
        println!("{}.", self.unit);
        self
    }

    // 関連関数
    // implの中でselfを引数に取らない関数を定義
    fn new(name: &str, price: f32, unit: &str) -> Vehicle {
        Vehicle {
            name: String::from(name),
            price: price,
            unit: String::from(unit),
        }
    }
}

fn main() {
    let prius = Vehicle {
        name: String::from("Prius"),
        price: 3000000.0,
        unit: String::from("Yen"),
    };

    prius.output_name(); // Thic vehicle is Prius.
    prius.calc_price().output_unit(); // The price is 3300000 Yen.

    // Vehicle構造体に関連付いていることを明示的に示す
    let rav4 = Vehicle::new("Rav4", 4500000.0, "Yen");
    rav4.output_name(); // Thic vehicle is Rav4.
    rav4.calc_price().output_unit(); // The price is 4950000 Yen.
}
```

## マクロ
### 文字列操作
* concat!: リテラルを結合する
* format!: フォーマット文字列と値から文字列を作成する
```rust
fn main() {
    let str1 = concat!("Text1", "Text2", "Text3");
    println!("{}", str1); // Text1Text2Text3

    let str2 = format!("{}-{:?}", str1, ("Text4", 5));
    println!("{}", str2); // Text1Text2Text3-("Text4", 5)

    let str3 = format!("{}{}", "ABC", "DEF");
    println!("{}", str3); // ABCDEF
}
```

### データ出力
* print!, println!: 標準出力に出力する
* eprint!, eprintln!: 標準エラー出力に出力する
* write!, writeln": バッファにバイト列を出力する
* dbg!: ファイル名、行数、式、式の値を標準エラー出力に出力する

```rust
use std::io::Write;

fn main() {
    print!("hello"); // hello
    println!("hello {}", "world"); // hello world

    let x: Result<i32, String> = Err(200.to_string());

    match x {
        Ok(value) => println!("Standart Output! {}", value),
        Err(error) => eprintln!("Standart Error! {}", error), // Standart print! 200 at standard error
    };

    let mut w = Vec::new();
    write!(&mut w, "{}", "abc"); // 97, 98, 99 (UTF-8)
    writeln!(&mut w, "def"); // 100, 101, 102, 10 (UTF-8)
    dbg!(w); // 上記を出力
    dbg!(1 + 2 + 3); // 1 + 2 + 3 = 6
}
```

### 異常終了
* panic!: 異常終了させる
```rust
fn main() {
    let x: i32 = 10;

    if x == 10 {
        panic!("finish program"); // thread 'main' panicked at 'finish program', src/main.rs:5:9
    }
}
```

### ベクタの初期化
* vec!: Vec<T>を初期化する
```rust
fn main() {
    let mut vec: Vec<i32> = Vec::new();
    vec.push(1);
    vec.push(2);
    vec.push(3);
    println!("{:?}", vec); // [1, 2, 3]

    let vec_macro = vec![1, 2, 3];
    println!("{:?}", vec_macro); // [1, 2, 3]
}
```

### リソースへのアクセス
* file!: ファイル名を取得する
* line!: 行番号を取得する
* cfg!: コンパイラの情報のboolを返す
* env!: 環境変数を取得する
```rust
fn main() {
    println!("this file is defined in {}", file!()); // this file is defined in src/main.rs
    println!("this file is defined on line {}", line!()); // this file is defined on line 3
    println!("target of is {}", cfg!(target_os = "linux")); // is test true
    println!("CARGO_HOME {}", env!("CARGO_HOME")); // CARGO_HOME /playground/.cargo
    println!("CARGO {}", env!("CARGO")); // CARGO /playground/.rustup/toolchains/stable-x86_64-unknown-linux-gnu/bin/cargo
}
```
### アサーション
* assert!: 引数がtrueであることを表明する
* assert_eq!, debug_assert_eq!: 2つの引数が等しいことを表明する
* assert_ne!, debug_assert_ne!: 2つの引数が等しくないことを表明する
```rust
fn main() {
    assert!(true);
    assert_eq!(1, 1);
    assert_ne!(1, 0);
    println!("debug"); // debug

    // 行か、cargo run --release時は無視される
    debug_assert!(false); // thread 'main' panicked at 'assertion failed: false', src/main.rs:7:5
    debug_assert_eq!(1, 1);
    debug_assert_ne!(1, 0);
}
```

### 実装の補助
* unimplemented!: 実装されていないことを示す
* todo!: 今後実装されることを示す
* unreachable!: 処理が到達しないことを示す
```rust
// ShipStatusはDebugトレイトが実装されていないため指定必要
#[derive(Debug)]
enum ShipStatus {
    ShipNg, // SHIP_NGはダメ
    ShipOk,
}

trait Vehicle {
    fn get_price(&mut self) -> i32;
    fn get_ship_status(&mut self) -> String;
    fn get_discount_rate(&self) -> i32;
}

struct Prius {
    price: i32,
    ship_status: ShipStatus,
}

impl Vehicle for Prius {
    fn get_price(&mut self) -> i32 {
        // i32をreturnせずとも型検査を通過
        unimplemented!()
    }

    fn get_ship_status(&mut self) -> String {
        // format!: フォーマット後の文字列を取得
        // 列挙型ShipStatusの内容を文字列として出力
        format!("{:?}", self.ship_status)
    }

    fn get_discount_rate(&self) -> i32 {
        // i32をreturnせずとも型検査を通過
        todo!()
    }
}

fn main() {
    let mut prius = Prius {
        price: 3000000,
        ship_status: ShipStatus::ShipOk,
    };

    println!("Ship Status: {}", prius.get_ship_status()); // Ship Status: ShipOk
}
```

### トレイト
* Rustでは`derive`を使って構造体や列挙型に振る舞いを追加できる
    * PartialEq: オブジェクト同士が等価であることを比較する振る舞い
    * Eq: 同じ値での比較が全て真のときに付与できる振る舞い
        * Eqを実装するには、PartialEqも実装が必要
        * 浮動小数だとEqではダメ
    * PartialOrd: 大小を比較するための振る舞い
        * PartialEqも一緒に実装が必要
    * Ord: 順序付けできないものだけに付与できる
    * Copy: =演算子を使った時に、所有権の移動ではなくクローンを行う振る舞い
        * Cloneも一緒に実装が必要
    * Clone: クローンできる振る舞い
    * Debug: デバッグ出力できる振る舞いの追加
    * Default: 構造体のデフォルト値を決められる
```rust
// --- Eq, PartialEq ---
#[derive(PartialEq)]
struct A(i32);

#[derive(Debug)]
struct A1(i32);

// --- PartialOrd ---
#[derive(PartialEq, PartialOrd)]
struct B(f32);

#[derive(PartialEq)]
struct B1(f32);

// --- Copy, Clone ---
#[derive(Copy, Clone, Debug)]
struct C;

#[derive(Debug)]
struct C1;

#[derive(Clone, Debug)]
struct D;

// --- Debug ---
#[derive(Debug)]
struct E;

// --- Default ---
#[derive(Default, Debug)]
struct F {
    name: String,
    price: f32,
}

#[derive(Debug)]
struct F1 {
    name: String,
    price: f32,
}

fn main() {
    // --- Eq, PartialEq ---
    println!("{:?}", A(0) == A(0)); // false

    // println!("{:?}", A1(0) == A1(0)); // an implementation of `PartialEq` might be missing for `A1`

    // --- PartialOrd ---
    println!("{:?}", B(1.0) > B(0.0)); // true

    // println!("{:?}", B1(1.0) > B1(0.0)); // an implementation of `PartialOrd` might be missing for `B1`

    // --- Copy ---
    // 所有権の移動ではなく、クローンが行われている
    let c = C;
    let clone1 = c;
    let clone2 = c;

    let clone1_pointer = &clone1 as *const C;
    let clone2_pointer = &clone1 as *const C;
    println!("{:?}", clone1_pointer); // 0x7ffcf39b5b38
    println!("{:?}", clone2_pointer); // 0x7ffcf39b5b40

    let c1 = C1;
    let clone1_1 = c1; // 所有権が移動

    // let clone1_2 = c1; // move occurs because `c1` has type `C1`, which does not implement the `Copy` trait

    // --- Clone ---
    let d = D;
    let clone_d = d.clone();
    println!("{:?}", clone_d); // D

    // --- Debug ---
    println!("{:?}", E); // E

    // --- Default ---
    let f = F::default();
    println!("{:?}", f); // F { name: "", price: 0.0 }

    // let f1 = F1::default();
    // the following trait defines an item `default`, perhaps you need to implement it:
}
```