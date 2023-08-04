# Rust tips - #2
### Reference
* 実践Rustプログラミング入門
  * 初田直也【著】/山口聖弘【著】/吉川哲史【著】/豊田優貴【著】/松本健太郎【著】/原将己【著】

### 文字列型
```
fn main() {
    let s1: String = String::from("Hello world");
    let s2: &str = &s1;
    let s3: String = s2.to_string();
}
```