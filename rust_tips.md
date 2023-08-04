# Rust tips - #1
### Reference
* 実践Rustプログラミング入門
  * 初田直也【著】/山口聖弘【著】/吉川哲史【著】/豊田優貴【著】/松本健太郎【著】/原将己【著】

### 変数の再代入
```rust
fn main() {
    // letで変数の割当（束縛）
    // mutを付けることで可変
    let mut x = 10;
    println!("number: {}", x); // 10

    x = 100;
    println!("number: {}", x); // 100
}
```

### デバッグトレイト
```rust
fn main() {
    let num = vec![1, 2, 3, 4, 5];

    // Displayトレイト。構造体や列挙型では使えない
    // println!("{}", num); // error

    // Debugトレイト
    println!("{:?}", num); // [1, 2, 3, 4, 5]
}
```

### イテレータ
最後のcollectがないと、下記のような出力になる
`Map { iter: Filter { iter: IntoIter([1, 2, 3, 4, 5]) } }`
```rust
fn main() {
    let nums = vec![1, 2, 3, 4, 5];
    let result = nums
        .into_iter() // イテレータの作成
        .filter(|n| n%2 == 0) // filterは当たられたクロージャによるフィルタリング
        .map(|n| n.to_string()) // 各要素に対してクロージャを適用
        .collect::<Vec<String>>(); // イテレータをベクタに変換

    println!("{:?}", result); // ["2", "4"]
}

```

### パターンマッチング
Noneを消すと、全て網羅していないのでエラーが出る
```rust
fn main() {
    // Option型
    let objective: Option<i32> = Some(1);

    // matchは他言語でいうswitch, caseのようなもの
    match objective {
        Some(x) if x % 2 == 0 => println!("even: {}", x),
        Some(x) => println!("odd: {}", x),
        None => println!("not a number"),
    }
}

```
Option型は、無効な値を取れる列挙型
```rust
enum Option<T> {
    Some(T), // 値が存在
    None, // 値が存在しない
}
```

### 型推論
変数を宣言した時点では何が入るか分からなくても、逆算的に型を推論してくれる
```rust
// 型を調べるためのtype_nameを表示するための関数
fn print_type_name<T>(_: T) {
    println!("{}", std::any::type_name::<T>());
}

fn main() {
    let mut v = vec![];

    v.push(100);
    print_type_name(v); // alloc::vec::Vec<i32>
}

```

### トレイト
他言語でいうインタフェース
構造体に共通メソッドを実装したいときに利用
```rust
fn main() {
    let prius = Prius {};
    let rav4 = Rav4 {};

    show_vehicle_data(prius);
    show_vehicle_data(rav4);
}

trait Vehicle {
    // self, &self, &mut selfのいずれかが付く
    // self: メソッド内でデータを所有し、変更する場合
    // &self: メソッド内でデータを読み取るだけで変更しない場合
    // &mut self: メソッド内でデータを変更する場合
    fn price(&self) -> u32;
    fn ranking(&self) -> String;
}

struct Prius;

impl Vehicle for Prius {
    fn price(&self) -> u32 {
        3000000
    }

    fn ranking(&self) -> String {
        "No.1".to_string()
    }
}

struct Rav4;

impl Vehicle for Rav4 {
    fn price(&self) -> u32 {
        4500000
    }

    fn ranking(&self) -> String {
        "No.2".to_string()
    }
}

// <T: Vehicle> Vehicleトレイトを実装する任意の型
// (vehicle: T) 関数の引数で、Vehicleトレイトを実装する型のインスタンスを受け取る
fn show_vehicle_data<T: Vehicle>(vehicle: T) {
    println!("price: {}", vehicle.price());
    println!("ranking: {}", vehicle.ranking());
}
```

###コマンド
* `cargo init`: パッケージの作成環境を構築
* `cargo build`: ビルド
  * cargoは外部パッケージを自動でダウンロードしてビルド
  * 依存する外部パッケージはCargo.tomlに記述
  * ダウンロードしたパッケージのバージョンはCargo.lockに書き込まれる
* `cargo run`: 作成した実行可能ファイルを起動
* `cargo check`: コンパイルチェックのみ
* `cargo add`: Cargo.tomlのdependenciesへの追加
* `cargo tree`: パッケージ構成の確認