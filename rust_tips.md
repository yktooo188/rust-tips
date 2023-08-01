# Rust tips

## Chapter 1
### 変数の再代入
```
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
```
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
```
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
```
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
```
enum Option<T> {
    Some(T), // 値が存在
    None, // 値が存在しない
}
```

### 型推論
変数を宣言した時点では何が入るか分からなくても、逆算的に型を推論してくれる
```
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