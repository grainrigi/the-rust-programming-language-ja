% ドキュメント
<!--  % Documentation  -->

<!-- Documentation is an important part of any software project, and it's -->
<!-- first-class in Rust. Let's talk about the tooling Rust gives you to -->
<!-- document your project. -->
ドキュメントはどんなソフトウェアプロジェクトにとっても重要な部分であり、Rustにおいてはファーストクラスです。
プロジェクトのドキュメントを作成するために、Rustが提供するツールについて話しましょう。

<!-- ## About `rustdoc` -->
## `rustdoc` について

<!-- The Rust distribution includes a tool, `rustdoc`, that generates documentation. -->
<!-- `rustdoc` is also used by Cargo through `cargo doc`. -->
Rustの配布物には `rustdoc` というドキュメントを生成するツールが含まれています。
`rustdoc` は `cargo doc` によってCargoでも使われます。

<!-- Documentation can be generated in two ways: from source code, and from -->
<!-- standalone Markdown files. -->
ドキュメントは2通りの方法で生成することができます。ソースコードから、そして単体のMarkdownファイルからです。

<!-- ## Documenting source code -->
## ソースコードのドキュメントの作成

<!-- The primary way of documenting a Rust project is through annotating the source -->
<!-- code. You can use documentation comments for this purpose: -->
Rustのプロジェクトでドキュメントを書く1つ目の方法は、ソースコードに注釈を付けることで行います。
ドキュメンテーションコメントはこの目的のために使うことができます。

```rust,ignore
# /// Constructs a new `Rc<T>`.
/// 新しい`Rc<T>`の生成
///
/// # Examples
///
/// ```
/// use std::rc::Rc;
///
/// let five = Rc::new(5);
/// ```
pub fn new(value: T) -> Rc<T> {
#   // implementation goes here
    // 実装が続く
}
```

<!-- This code generates documentation that looks [like this][rc-new]. I've left the -->
<!-- implementation out, with a regular comment in its place. -->
このコードは[このような][rc-new]見た目のドキュメントを生成します。
実装についてはそこにある普通のコメントのとおり、省略しています。

<!-- The first thing to notice about this annotation is that it uses -->
<!-- `///` instead of `//`. The triple slash -->
<!-- indicates a documentation comment. -->
この注釈について注意すべき1つ目のことは、 `//` の代わりに `///` が使われていることです。
3連スラッシュはドキュメンテーションコメントを示します。

<!-- Documentation comments are written in Markdown. -->
ドキュメンテーションコメントはMarkdownで書きます。

<!-- Rust keeps track of these comments, and uses them when generating -->
<!-- documentation. This is important when documenting things like enums: -->
Rustはそれらのコメントを把握し、ドキュメントを生成するときにそれらを使います。
このことは次のように列挙型のようなもののドキュメントを作成するときに重要です。

```rust
# /// The `Option` type. See [the module level documentation](index.html) for more.
/// `Option`型。詳細は[モジュールレベルドキュメント](index.html)を参照
enum Option<T> {
#   /// No value
    /// 値なし
    None,
#   /// Some value `T`
    /// `T`型の何らかの値
    Some(T),
}
```

<!-- The above works, but this does not: -->
上記の例は動きますが、これは動きません。

```rust,ignore
# /// The `Option` type. See [the module level documentation](index.html) for more.
/// `Option`型。詳細は[モジュールレベルドキュメント](index.html)を参照
enum Option<T> {
#   /// None, /// No value
    None, /// 値なし
#   /// Some(T), /// Some value `T`
    Some(T), /// `T`型の何らかの値
}
```

<!-- You'll get an error: -->
次のようにエラーが発生します。

```text
hello.rs:4:1: 4:2 error: expected ident, found `}`
hello.rs:4 }
           ^
```

<!-- This [unfortunate error](https://github.com/rust-lang/rust/issues/22547) is -->
<!-- correct; documentation comments apply to the thing after them, and there's -->
<!-- nothing after that last comment. -->
この [残念なエラー](https://github.com/rust-lang/rust/issues/22547) は正しいのです。ドキュメンテーションコメントはそれらの後のものに適用されるところ、その最後のコメントの後には何もないからです。

[rc-new]: https://doc.rust-lang.org/nightly/std/rc/struct.Rc.html#method.new

<!-- ### Writing documentation comments -->
### ドキュメンテーションコメントの記述

<!-- Anyway, let's cover each part of this comment in detail: -->
とりあえず、このコメントの各部分を詳細にカバーしましょう。

```rust
# /// Constructs a new `Rc<T>`.
/// 新しい`Rc<T>`の生成
# fn foo() {}
```

<!-- The first line of a documentation comment should be a short summary of its -->
<!-- functionality. One sentence. Just the basics. High level. -->
ドキュメンテーションコメントの最初の行は、その機能の短いサマリにすべきです。
一文で。
基本だけを。
高レベルから。

```rust
///
# /// Other details about constructing `Rc<T>`s, maybe describing complicated
# /// semantics, maybe additional options, all kinds of stuff.
/// `Rc<T>`の生成についてのその他の詳細。例えば、複雑なセマンティクスの説明、
/// 追加のオプションなどあらゆる種類のもの
///
# fn foo() {}
```

<!-- Our original example had just a summary line, but if we had more things to say, -->
<!-- we could have added more explanation in a new paragraph. -->
この例にはサマリしかありませんが、もしもっと書くべきことがあれば、新しい段落にもっと多くの説明を追加することができます。

<!-- #### Special sections -->
#### 特別なセクション

<!-- Next, are special sections. These are indicated with a header, `#`. There -->
<!-- are four kinds of headers that are commonly used. They aren't special syntax, -->
<!-- just convention, for now. -->
次は特別なセクションです。
それらには `#` が付いていて、ヘッダであることを示しています。
一般的には、4種類のヘッダが使われます。
今のところそれらは特別な構文ではなく、単なる慣習です。

```rust
/// # Panics
# fn foo() {}
```

<!-- Unrecoverable misuses of a function (i.e. programming errors) in Rust are -->
<!-- usually indicated by panics, which kill the whole current thread at the very -->
<!-- least. If your function has a non-trivial contract like this, that is -->
<!-- detected/enforced by panics, documenting it is very important. -->
Rustにおいて、関数の回復不可能な誤用（つまり、プログラミングエラー）は普通、パニックによって表現されます。パニックは、少なくとも現在のスレッド全体の息の根を止めてしまいます。
もし関数にこのような、パニックによって検出されたり強制されたりするような自明でない取決めがあるときには、ドキュメントを作成することは非常に重要です。

```rust
/// # Errors
# fn foo() {}
```

<!-- If your function or method returns a `Result<T, E>`, then describing the -->
<!-- conditions under which it returns `Err(E)` is a nice thing to do. This is -->
<!-- slightly less important than `Panics`, because failure is encoded into the type -->
<!-- system, but it's still a good thing to do. -->
もし関数やメソッドが `Result<T, E>` を戻すのであれば、それが `Err(E)` を戻したときの状況をドキュメントで説明するのはよいことです。
これは `Panics` のときに比べると重要性は少し下です。失敗は型システムによってコード化されますが、それでもまだそうすることはよいことだからです。

```rust
/// # Safety
# fn foo() {}
```

<!-- If your function is `unsafe`, you should explain which invariants the caller is -->
<!-- responsible for upholding. -->
もし関数が `unsafe` であれば、呼出元が動作を続けるためにはどの不変条件について責任を持つべきなのかを説明すべきです。

```rust
/// # Examples
///
/// ```
/// use std::rc::Rc;
///
/// let five = Rc::new(5);
/// ```
# fn foo() {}
```

<!-- Fourth, `Examples`. Include one or more examples of using your function or -->
<!-- method, and your users will love you for it. These examples go inside of -->
<!-- code block annotations, which we'll talk about in a moment, and can have -->
<!-- more than one section: -->
4つ目は `Examples` です。
関数やメソッドの使い方の例を1つ以上含めてください。そうすればユーザから愛されることでしょう。
それらの例はコードブロック注釈内に入れます。コードブロック注釈についてはすぐ後で話しますが、それらは1つ以上のセクションを持つことができます。

```rust
/// # Examples
///
# /// Simple `&str` patterns:
/// 単純な`&str`パターン
///
/// ```
/// let v: Vec<&str> = "Mary had a little lamb".split(' ').collect();
/// assert_eq!(v, vec!["Mary", "had", "a", "little", "lamb"]);
/// ```
///
# /// More complex patterns with a lambda:
/// ラムダを使ったもっと複雑なパターン
///
/// ```
/// let v: Vec<&str> = "abc1def2ghi".split(|c: char| c.is_numeric()).collect();
/// assert_eq!(v, vec!["abc", "def", "ghi"]);
/// ```
# fn foo() {}
```

<!-- Let's discuss the details of these code blocks. -->
それらのコードブロックの詳細について議論しましょう。

<!-- #### Code block annotations -->
#### コードブロック注釈

<!-- To write some Rust code in a comment, use the triple graves: -->
コメント内にRustのコードを書くためには、3連バッククオートを使います。

```rust
/// ```
/// println!("Hello, world");
/// ```
# fn foo() {}
```

<!-- If you want something that's not Rust code, you can add an annotation: -->
もしRustのコードではないものを書きたいのであれば、注釈を追加することができます。

```rust
/// ```c
/// printf("Hello, world\n");
/// ```
# fn foo() {}
```

<!-- This will highlight according to whatever language you're showing off. -->
<!-- If you're only showing plain text, choose `text`. -->
これは、使われている言語が何であるかに応じてハイライトされます。
もし単なるプレーンテキストを書いているのであれば、 `text` を選択してください。

<!-- It's important to choose the correct annotation here, because `rustdoc` uses it -->
<!-- in an interesting way: It can be used to actually test your examples in a -->
<!-- library crate, so that they don't get out of date. If you have some C code but -->
<!-- `rustdoc` thinks it's Rust because you left off the annotation, `rustdoc` will -->
<!-- complain when trying to generate the documentation. -->
ここでは正しい注釈を選ぶことが重要です。なぜなら、 `rustdoc` はそれを興味深い方法で使うからです。それらが実際のコードと不整合を起こさないように、ライブラリクレート内で実際にあなたの例をテストするために使うのです。
もし例の中にCのコードが含まれているのに、あなたが注釈を付けるのを忘れてしまい、 `rustdoc` がそれをRustのコードだと考えてしまえば、 `rustdoc` はドキュメントを生成しようとするときに怒るでしょう。

<!-- ## Documentation as tests -->
## テストとしてのドキュメント

<!-- Let's discuss our sample example documentation: -->
次のようなドキュメントにおける例について議論しましょう。

```rust
/// ```
/// println!("Hello, world");
/// ```
# fn foo() {}
```

<!-- You'll notice that you don't need a `fn main()` or anything here. `rustdoc` will -->
<!-- automatically add a `main()` wrapper around your code, using heuristics to attempt -->
<!-- to put it in the right place. For example: -->
`fn main()` とかがここでは不要だということに気が付くでしょう。
`rustdoc` は自動的に `main()` ラッパをコードの周りに、正しい場所へ配置するためのヒューリスティクスを使って追加します。
例えば、こうです。

```rust
/// ```
/// use std::rc::Rc;
///
/// let five = Rc::new(5);
/// ```
# fn foo() {}
```

<!-- This will end up testing: -->
これが、テストのときには結局こうなります。

```rust
fn main() {
    use std::rc::Rc;
    let five = Rc::new(5);
}
```

<!-- Here's the full algorithm rustdoc uses to preprocess examples: -->
これがrustdocが例の前処理に使うアルゴリズムの全てです。

<!-- 1. Any leading `#![foo]` attributes are left intact as crate attributes. -->
<!-- 2. Some common `allow` attributes are inserted, including -->
<!--    `unused_variables`, `unused_assignments`, `unused_mut`, -->
<!--    `unused_attributes`, and `dead_code`. Small examples often trigger -->
<!--    these lints. -->
<!-- 3. If the example does not contain `extern crate`, then `extern crate -->
<!--    <mycrate>;` is inserted (note the lack of `#[macro_use]`). -->
<!-- 4. Finally, if the example does not contain `fn main`, the remainder of the -->
<!--    text is wrapped in `fn main() { your_code }`. -->
1. 前の方にある全ての `#![foo]` アトリビュートは、そのままクレートのアトリビュートとして置いておく
2. `unused_variables` 、 `unused_assignments` 、 `unused_mut` 、 `unused_attributes` 、 `dead_code` などのいくつかの一般的な `allow` アトリビュートを追加する。
   小さな例はしばしばこれらのリントに引っ掛かる
3. もしその例が `extern crate` を含んでいなければ、 `extern crate <mycrate>;` を挿入する（ `#[macro_use]` がないことに注意する）
4. 最後に、もし例が `fn main` を含んでいなければ、テキストの残りの部分を `fn main() { your_code }` で囲む

<!-- This generated `fn main` can be a problem! If you have `extern crate` or a `mod` -->
<!-- statements in the example code that are referred to by `use` statements, they will -->
<!-- fail to resolve unless you include at least `fn main() {}` to inhibit step 4. -->
<!-- `#[macro_use] extern crate` also does not work except at the crate root, so when -->
<!-- testing macros an explicit `main` is always required. It doesn't have to clutter -->
<!-- up your docs, though -- keep reading! -->
こうして生成された `fn main` は問題になり得ます!
もし `use` 文によって参照される例のコードに `extern crate` 文や `mod` 文が入っていれば、それらはステップ4を抑制するために少なくとも `fn main() {}` を含んでいない限り失敗します。
`#[macro_use] extern crate` も同様に、クレートのルート以外では動作しません。そのため、マクロのテストには明示的な `main` が常に必要なのです。
しかし、ドキュメントを散らかす必要はありません……続きを読みましょう!

<!-- Sometimes this algorithm isn't enough, though. For example, all of these code samples -->
<!-- with `///` we've been talking about? The raw text: -->
しかし、これでは不十分なことがときどきあります。
例えば、今まで話してきた全ての `///` の付いたコード例はどうだったでしょうか。
生のテキストはこうなっています。

```text
/// 何らかのドキュメント
# fn foo() {}
```

<!-- looks different than the output: -->
それは出力とは違って見えます。

```rust
# /// Some documentation.
/// 何らかのドキュメント
# fn foo() {}
```

<!-- Yes, that's right: you can add lines that start with `# `, and they will -->
<!-- be hidden from the output, but will be used when compiling your code. You -->
<!-- can use this to your advantage. In this case, documentation comments need -->
<!-- to apply to some kind of function, so if I want to show you just a -->
<!-- documentation comment, I need to add a little function definition below -->
<!-- it. At the same time, it's only there to satisfy the compiler, so hiding -->
<!-- it makes the example more clear. You can use this technique to explain -->
<!-- longer examples in detail, while still preserving the testability of your -->
<!-- documentation. -->
そうです。正解です。 `# ` で始まる行を追加することで、コードをコンパイルするときには使われるけれども、出力はされないというようにすることができます。
これは都合のよいように使うことができます。
この場合、ドキュメンテーションコメントそのものを見せたいので、ドキュメンテーションコメントを何らかの関数に適用する必要があります。そのため、その後に小さい関数定義を追加する必要があります。
同時に、それは単にコンパイラを満足させるためだけのものなので、それを隠すことで、例がすっきりするのです。
長い例を詳細に説明する一方、テスト可能性を維持するためにこのテクニックを使うことができます。

<!-- For example, imagine that we wanted to document this code: -->
例えば、このコードをドキュメントに書きたいとします。

```rust
let x = 5;
let y = 6;
println!("{}", x + y);
```

<!-- We might want the documentation to end up looking like this: -->
最終的にはこのように見えるドキュメントが欲しいのかもしれません。

<!-- > First, we set `x` to five: -->
> まず、`x`に5をセットする
>
> ```rust
> let x = 5;
> # let y = 6;
> # println!("{}", x + y);
> ```
>
<!-- > Next, we set `y` to six: -->
> 次に、`y`に6をセットする
>
> ```rust
> # let x = 5;
> let y = 6;
> # println!("{}", x + y);
> ```
>
<!-- > Finally, we print the sum of `x` and `y`: -->
> 最後に、`x`と`y`との合計を出力する
>
> ```rust
> # let x = 5;
> # let y = 6;
> println!("{}", x + y);
> ```

<!-- To keep each code block testable, we want the whole program in each block, but -->
<!-- we don't want the reader to see every line every time.  Here's what we put in -->
<!-- our source code: -->
各コードブロックをテスト可能な状態にしておくために、各ブロックにはプログラム全体が必要です。しかし、読者には全ての行を毎回見せたくはありません。
ソースコードに挿入するものはこれです。

```text
    まず、`x`に5をセットする

    ```rust
    let x = 5;
    # let y = 6;
    # println!("{}", x + y);
    ```

    次に、`y`に6をセットする

    ```rust
    # let x = 5;
    let y = 6;
    # println!("{}", x + y);
    ```

    最後に、`x`と`y`との合計を出力する

    ```rust
    # let x = 5;
    # let y = 6;
    println!("{}", x + y);
    ```
```

<!-- By repeating all parts of the example, you can ensure that your example still -->
<!-- compiles, while only showing the parts that are relevant to that part of your -->
<!-- explanation. -->
例の全体を繰り返すことで、例がちゃんとコンパイルされることを保証する一方、説明に関係する部分だけを見せることができます。

<!-- ### Documenting macros -->
### マクロのドキュメントの作成

<!-- Here’s an example of documenting a macro: -->
これはマクロのドキュメントの例です。

```rust
# /// Panic with a given message unless an expression evaluates to true.
/// 式がtrueと評価されない限り、与えられたメッセージとともにパニックする
///
/// # Examples
///
/// ```
/// # #[macro_use] extern crate foo;
/// # fn main() {
/// panic_unless!(1 + 1 == 2, “Math is broken.”);
/// # }
/// ```
///
/// ```should_panic
/// # #[macro_use] extern crate foo;
/// # fn main() {
/// panic_unless!(true == false, “I’m broken.”);
/// # }
/// ```
#[macro_export]
macro_rules! panic_unless {
    ($condition:expr, $($rest:expr),+) => ({ if ! $condition { panic!($($rest),+); } });
}
# fn main() {}
```

<!-- You’ll note three things: we need to add our own `extern crate` line, so that -->
<!-- we can add the `#[macro_use]` attribute. Second, we’ll need to add our own -->
<!-- `main()` as well (for reasons discussed above). Finally, a judicious use of -->
<!-- `#` to comment out those two things, so they don’t show up in the output. -->
3つのことに気が付くでしょう。 `#[macro_use]` アトリビュートを追加するために、自分で `extern crate` 行を追加しなければなりません。
2つ目に、 `main()` も自分で追加する必要があります（理由は前述しました）。
最後に、それらの2つが出力されないようにコメントアウトするという `#` の賢い使い方です。

<!-- Another case where the use of `#` is handy is when you want to ignore -->
<!-- error handling. Lets say you want the following, -->
`#` の使うと便利な場所のもう1つのケースは、エラーハンドリングを無視したいときです。
次のようにしたいとしましょう。

```rust,ignore
/// use std::io;
/// let mut input = String::new();
/// try!(io::stdin().read_line(&mut input));
```

<!-- The problem is that `try!` returns a `Result<T, E>` and test functions -->
<!-- don't return anything so this will give a mismatched types error. -->
問題は `try!` が `Result<T, E>` を返すところ、テスト関数は何も返さないことで、これは型のミスマッチエラーを起こします。

```rust,ignore
# /// A doc test using try!
/// try!を使ったドキュメンテーションテスト
///
/// ```
/// use std::io;
/// # fn foo() -> io::Result<()> {
/// let mut input = String::new();
/// try!(io::stdin().read_line(&mut input));
/// # Ok(())
/// # }
/// ```
# fn foo() {}
```

<!-- You can get around this by wrapping the code in a function. This catches -->
<!-- and swallows the `Result<T, E>` when running tests on the docs. This -->
<!-- pattern appears regularly in the standard library. -->
これは関数内のコードをラッピングすることで回避できます。
これはドキュメント上のテストが実行されるときに `Result<T, E>` を捕まえて飲み込みます。
このパターンは標準ライブラリ内でよく現れます。

<!-- ### Running documentation tests -->
### ドキュメンテーションテストの実行

<!-- To run the tests, either: -->
テストを実行するには、次のどちらかを使います。

```bash
$ rustdoc --test path/to/my/crate/root.rs
# or
$ cargo test
```

<!-- That's right, `cargo test` tests embedded documentation too. **However, -->
<!-- `cargo test` will not test binary crates, only library ones.** This is -->
<!-- due to the way `rustdoc` works: it links against the library to be tested, -->
<!-- but with a binary, there’s nothing to link to. -->
正解です。 `cargo test` は組み込まれたドキュメントもテストします。
**しかし、`cargo test`がテストするのはライブラリクレートだけで、バイナリクレートはテストしません。**
これは `rustdoc` の動き方によるものです。それはテストするためにライブラリをリンクしますが、バイナリには何もリンクするものがないからです。

<!-- There are a few more annotations that are useful to help `rustdoc` do the right -->
<!-- thing when testing your code: -->
`rustdoc` がコードをテストするときに正しく動作するのを助けるために便利な注釈があと少しあります。

```rust
/// ```ignore
/// fn foo() {
/// ```
# fn foo() {}
```

<!-- The `ignore` directive tells Rust to ignore your code. This is almost never -->
<!-- what you want, as it's the most generic. Instead, consider annotating it -->
<!-- with `text` if it's not code, or using `#`s to get a working example that -->
<!-- only shows the part you care about. -->
`ignore` ディレクティブはRustにコードを無視するよう指示します。
これはあまりに汎用的なので、必要になることはほとんどありません。
もしそれがコードではなければ、代わりに `text` の注釈を付けること、又は問題となる部分だけが表示された、動作する例を作るために `#` を使うことを検討してください。

```rust
/// ```should_panic
/// assert!(false);
/// ```
# fn foo() {}
```

<!-- `should_panic` tells `rustdoc` that the code should compile correctly, but -->
<!-- not actually pass as a test. -->
`should_panic` は、そのコードは正しくコンパイルされるべきではあるが、実際にテストとして成功する必要まではないということを `rustdoc` に教えます。

```rust
/// ```no_run
/// loop {
///     println!("Hello, world");
/// }
/// ```
# fn foo() {}
```

<!-- The `no_run` attribute will compile your code, but not run it. This is -->
<!-- important for examples such as "Here's how to start up a network service," -->
<!-- which you would want to make sure compile, but might run in an infinite loop! -->
`no_run` アトリビュートはコードをコンパイルしますが、実行はしません。
これは「これはネットワークサービスを開始する方法です」というような例や、コンパイルされることは保証したいけれども、無限ループになってしまうような例にとって重要です!

<!-- ### Documenting modules -->
### モジュールのドキュメントの作成

<!-- Rust has another kind of doc comment, `//!`. This comment doesn't document the next item, but the enclosing item. In other words: -->
Rustには別の種類のドキュメンテーションコメント、 `//!` があります。
このコメントは次に続く要素のドキュメントではなく、それを囲っている要素のドキュメントです。
言い換えると、こうです。

```rust
mod foo {
#   //! This is documentation for the `foo` module.
    //! これは`foo`モジュールのドキュメントである
    //!
    //! # Examples

    // ...
}
```

<!-- This is where you'll see `//!` used most often: for module documentation. If -->
<!-- you have a module in `foo.rs`, you'll often open its code and see this: -->
`//!` を頻繁に見る場所がここ、モジュールドキュメントです。
もし `foo.rs` 内にモジュールを持っていれば、しばしばそのコードを開くとこれを見るでしょう。

```rust
# //! A module for using `foo`s.
//! `foo`で使われるモジュール
//!
# //! The `foo` module contains a lot of useful functionality blah blah blah
//! `foo`モジュールに含まれているたくさんの便利な関数などなど
```

<!-- ### Documentation comment style -->
### ドキュメンテーションコメントのスタイル

<!-- Check out [RFC 505][rfc505] for full conventions around the style and format of -->
<!-- documentation. -->
ドキュメントのスタイルや書式についての全ての慣習を知るには [RFC 505][rfc505] をチェックしてください。

[rfc505]: https://github.com/rust-lang/rfcs/blob/master/text/0505-api-comment-conventions.md

<!-- ## Other documentation -->
## その他のドキュメント

<!-- All of this behavior works in non-Rust source files too. Because comments -->
<!-- are written in Markdown, they're often `.md` files. -->
ここにある振舞いは全て、Rust以外のソースコードファイルでも働きます。
コメントはMarkdownで書かれるので、しばしば `.md` ファイルになります。

<!-- When you write documentation in Markdown files, you don't need to prefix -->
<!-- the documentation with comments. For example: -->
ドキュメントをMarkdownファイルに書くとき、ドキュメントにコメントのプレフィックスを付ける必要はありません。
例えば、こうする必要はありません。

```rust
/// # Examples
///
/// ```
/// use std::rc::Rc;
///
/// let five = Rc::new(5);
/// ```
# fn foo() {}
```

<!-- is -->
これは、こうします。

~~~markdown
# Examples

```
use std::rc::Rc;

let five = Rc::new(5);
```
~~~

<!-- when it's in a Markdown file. There is one wrinkle though: Markdown files need -->
<!-- to have a title like this: -->
Markdownファイルの中ではこうします。
ただし、1つだけ新しいものがあります。Markdownファイルではこのように題名を付けなければなりません。

```markdown
# % The title
% タイトル

# This is the example documentation.
これはサンプルのドキュメントです。
```

<!-- This `%` line needs to be the very first line of the file. -->
この `%` 行はそのファイルの一番先頭の行に書く必要があります。

<!-- ## `doc` attributes -->
## `doc` アトリビュート

<!-- At a deeper level, documentation comments are syntactic sugar for documentation -->
<!-- attributes: -->
もっと深いレベルで言えは、ドキュメンテーションコメントはドキュメントアトリビュートの糖衣構文です。

```rust
/// this
# fn foo() {}

#[doc="this"]
# fn bar() {}
```

<!-- are the same, as are these: -->
これらは同じもので、次のものも同じものです。

```rust
//! this

#![doc="this"]
```

<!-- You won't often see this attribute used for writing documentation, but it -->
<!-- can be useful when changing some options, or when writing a macro. -->
このアトリビュートがドキュメントを書くために使われているのを見ることはそんなにないでしょう。しかし、これは何らかのオプションを変更したり、マクロを書いたりするときに便利です。

<!-- ### Re-exports -->
### 再エクスポート

<!-- `rustdoc` will show the documentation for a public re-export in both places: -->
`rustdoc` はパブリックな再エクスポートがなされた場合に、両方の場所にドキュメントを表示します。

```ignore
extern crate foo;

pub use foo::bar;
```

<!-- This will create documentation for `bar` both inside the documentation for the -->
<!-- crate `foo`, as well as the documentation for your crate. It will use the same -->
<!-- documentation in both places. -->
これは `bar` のドキュメントをクレートのドキュメントの中に生成するのと同様に、 `foo` クレートのドキュメントの中にも生成します。
同じドキュメントが両方の場所で使われます。

<!-- This behavior can be suppressed with `no_inline`: -->
この振舞いは `no_inline` で抑制することができます。

```ignore
extern crate foo;

#[doc(no_inline)]
pub use foo::bar;
```

<!-- ## Missing documentation -->
## ドキュメントの不存在

<!-- Sometimes you want to make sure that every single public thing in your project -->
<!-- is documented, especially when you are working on a library. Rust allows you to -->
<!-- to generate warnings or errors, when an item is missing documentation. -->
<!-- To generate warnings you use `warn`: -->
ときどき、プロジェクト内の公開されている全てのものについて、ドキュメントが作成されていることを確認したいことがあります。これは特にライブラリについて作業をしているときにあります。
Rustでは、要素にドキュメントがないときに警告やエラーを生成することができます。
警告を生成するためには、 `warn` を使います。

```rust
#![warn(missing_docs)]
```

<!-- And to generate errors you use `deny`: -->
そしてエラーを生成するとき、 `deny` を使います。

```rust,ignore
#![deny(missing_docs)]
```

<!-- There are cases where you want to disable these warnings/errors to explicitly -->
<!-- leave something undocumented. This is done by using `allow`: -->
何かを明示的にドキュメント化されていないままにするため、それらの警告やエラーを無効にしたい場合があります。
これは `allow` を使えば可能です。

```rust
#[allow(missing_docs)]
struct Undocumented;
```

<!-- You might even want to hide items from the documentation completely: -->
ドキュメントから要素を完全に見えなくしたいこともあるかもしれません。

```rust
#[doc(hidden)]
struct Hidden;
```

<!-- ### Controlling HTML -->
### HTMLの制御

<!-- You can control a few aspects of the HTML that `rustdoc` generates through the -->
<!-- `#![doc]` version of the attribute: -->
`rustdoc` の生成するHTMLのいくつかの外見は、 `#![doc]` アトリビュートを通じて制御することができます。

```rust
#![doc(html_logo_url = "https://www.rust-lang.org/logos/rust-logo-128x128-blk-v2.png",
       html_favicon_url = "https://www.rust-lang.org/favicon.ico",
       html_root_url = "https://doc.rust-lang.org/")]
```

<!-- This sets a few different options, with a logo, favicon, and a root URL. -->
これは、複数の異なったオプション、つまりロゴ、お気に入りアイコン、ルートのURLをセットします。

<!-- ### Configuring documentation tests -->
### ドキュメンテーションテストの設定

<!-- You can also configure the way that `rustdoc` tests your documentation examples -->
<!-- through the `#![doc(test(..))]` attribute. -->
`rustdoc` がドキュメントの例をテストする方法は、 `#[doc(test(..))]` アトリビュートを通じて設定することができます。

```rust
#![doc(test(attr(allow(unused_variables), deny(warnings))))]
```

<!-- This allows unused variables within the examples, but will fail the test for any -->
<!-- other lint warning thrown. -->
これによって例の中の使われていない値は許されるようになりますが、その他の全てのリントの警告に対してテストは失敗するようになるでしょう。

<!-- ## Generation options -->
## 生成オプション

<!-- `rustdoc` also contains a few other options on the command line, for further customization: -->
`rustdoc` はさらなるカスタマイズのために、その他にもコマンドラインのオプションをいくつか持っています。

<!-- - `--html-in-header FILE`: includes the contents of FILE at the end of the -->
<!--   `<head>...</head>` section. -->
<!-- - `--html-before-content FILE`: includes the contents of FILE directly after -->
<!--   `<body>`, before the rendered content (including the search bar). -->
<!-- - `--html-after-content FILE`: includes the contents of FILE after all the rendered content. -->
- `--html-in-header FILE` : FILEの内容を `<head>...</head>` セクションの末尾に加える
- `--html-before-content FILE` : FILEの内容を `<body>` の直後、レンダリングされた内容（検索バーを含む）の直前に加える
- `--html-after-content FILE` : FILEの内容を全てのレンダリングされた内容の後に加える

<!-- ## Security note -->
## セキュリティ上の注意

<!-- The Markdown in documentation comments is placed without processing into -->
<!-- the final webpage. Be careful with literal HTML: -->
ドキュメンテーションコメント内のMarkdownは最終的なウェブページの中に無修正で挿入されます。
リテラルのHTMLには注意してください。

```rust
/// <script>alert(document.cookie)</script>
# fn foo() {}
```
