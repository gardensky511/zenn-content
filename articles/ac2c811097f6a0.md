---
title: "Javascript における Hoisting を理解したい"
emoji: "⛓️"
type: "tech"
topics: ["javascript"]
published: true
published_at: "2022-12-06 23:28"
---

:::message
この記事は [JavaScript Advent Calendar 2022](https://qiita.com/advent-calendar/2022/javascript) の7日目の記事です₍₍◝(°꒳°*)◜₎₎
:::

# 🌼 はじめに

Javascript の Hoisting をなんとなくは知ってたけど正確にどういう動きをするのかをあまり把握してなくて、一度ちゃんと理解したいという気持ちは昔からありました。

が、なかなか実行できず、、（^_^）だったので今年アドベントカレンダーを機会にちゃんと整理したいと思います！！

# 1. Hoisting とは

Javascript における Hoisting は「**変数や関数などの宣言をスコープの先頭に巻き上げること**」です。

「hoist」という英単語自体が「持ち上げる、巻き上げる」という意味を持っているので、日本語では Hoisting のことを「巻き上げ」とも言うらしいですね。

まあ言葉だけではピンとこないかもしれないので、これから変数と関数の具体例で説明します。

# 2. 変数の Hoisting

## 2-1. `var`

まず`var`の Hoisting による事象の一つを紹介します。

```js
console.log(name); // undeinfed
var name = 'みんちゃん';
```

ぱっと見「なんでこれエラーにならないんだ？」と思っちゃいますね。だって変数宣言の前に呼び出してるので、参照エラーになるのが自然に感じます。

エラーにはならない理由は、変数の宣言が巻き上げられたからです。上のコードが動作するイメージを表現するとこういう感じでしょう。

```js
var name;
console.log(name);
name = 'みんちゃん';
```
注意）実際のコードがこのように変更されたりコンパイルされたりはしません。 あくまでイメージするために書いたコードです。

ここで大事なことは、**宣言だけ Hoisting が発生する**ということです。試しに変数の宣言と初期化を分離してみましょう。

```js
console.log(num); // undefined を返す。宣言のみが巻き上げられ、この段階では初期化が行われないため

var num; // 宣言
num = 6; // 初期化
```

初期化は Hoisting が起こらず、巻き上げられることもないから`console.log`で`undefined`が出力されるということですね。

変数宣言せず初期化だけしてみるともっとはっきりした挙動が見られます。

```js
console.log(num); // ReferenceError
num = 6; // 初期化
```

変数宣言がない、従って Hoisting も起こらないので参照エラーが発生します。

なるほど、宣言のみ巻き上げられることは理解しました。次にその宣言が**スコープの先頭**に巻き上げられることを確認してみましょう。

```js
console.log(number) // Uncaught ReferenceError: number is not defined

function printNumber () {
  console.log(number) // undefined
  var number = 100;
}
```

`var`の有効スコープは関数スコープなので、変数宣言が関数の先頭には巻き上げられるます。関数の外はスコープ外なので巻き上げられません。

まあ言い換えると関数以外のスコープでは色々と巻き上げられてしまうので色々怖いですね。例えばfor文で宣言した変数とかも Hoisting が発生します。

```js
console.log(i) // undefined

for (var i = 0; i < 5; i++) {
  // ...
}
```

for文で宣言した`i`がfor文の外まで巻き上げられて`console.log`が参照エラーになっていません。

なんかやばい匂いを感じますね（^_^）。`var`は Hoisting 以外にも色々危ない仕様が多かったので、ES6から`const`と`let`が登場しました。

## 2-2. `const`、`let`

では`const`と`let`での Hoisting 挙動も見てみましょう。
`const`と`let`の有効スコープはブロックスコープなので、ブロックの中で実行してみました。

```js
{
  console.log(constVar) // Cannot access 'constVar' before initialization
  const constVar = "constVar"
}

{
  console.log(letVar) // Cannot access 'letVar' before initialization
  let letVar = "letVar"
}
```

両方`console.log`でエラーが発生してますね。

ここで注目すべき部分は、エラーメッセージです。`%% is not defined`ではなく、`Cannot access %% before initialization`、つまり初期化以前にその変数にアクセスできないというエラーメッセージになってます。

`var`とは違って`const`と`let`で変数を宣言すると、こういう変数にアクセスできない区間ができます。それを**TDZ**（**Temporal Dead Zone、一時的なデッドゾーン**）と言います。**TDZの範囲はスコープの先頭から変数の初期化が完了するまで**で、その間に変数にアクセスしたら先のようなエラーメッセージを返します。

サンプルコードで見てみたらイメージしやすいかもです。

```js
{ // fooの TDZ がスコープの先頭から始まる
  console.log(bar); // undefined
  console.log(foo); // Cannot access 'foo' before initialization
  var bar = 1;
  let foo = 2; // fooのTDZ終了（変数が初期化されたので）
}
```

ちなみにTDZはコードの作成順ではなく**コードの実行順によって生成**されます。

以下のサンプルコードの場合`let`の変数宣言がその変数にアクセスしてる`func`関数より下にありますが、`func`関数を呼び出す時点がTDZの外なので正常に動作します。

```js
{
    // letVarのTDZがスコープの先頭から始まる
    const func = () => console.log(letVar); // OK

    // TDZの内部。ここでletVarにアクセスしたらReferenceErrorになる

    let letVar = 3; // letVarのTDZ終了（変数が初期化されたので）
    func(); // TDZの外で呼び出してるのでOK
}
```

ということで`const`と`let`からは変数宣言前だとその変数にアクセスできなくなりました。有効スコープも`var`とは違ってブロックスコープなので、for文やif文で宣言した変数がその外まで巻き上げられることもなく、もっと安全な挙動になった気がします。

### +) `const`と`let`では Hoisting が発生しない？

[`const`のMDNページ](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Statements/const)と[`let`のMDNページ](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Statements/let)を読んでみると、`const`と`let`の変数宣言は non-hoisted としてみなされるという記述があります。では`const`と`let`の場合 Hoisting 自体が発生しないんでしょうか？

それに対する答えは [Hoisting のMDNページ](https://developer.mozilla.org/en-US/docs/Glossary/Hoisting)にありました。

要は「Hoisting という単語がすごく明確に合意が取れてるわけではないので non-hoisting にみなしてもいいけど、**巻き上げ自体は発生する**」ということです。

巻き上げが発生するということは以下のサンプルコードを見たら理解できます。

```js
const x = 1;
{
  console.log(x); // Cannot access 'x' before initialization
  const x = 2;
}
```

もし`const x = 2;`で全く Hoisting が起こらないなら、`console.log(x)`は上位スコープから`x`の値を読み取れるはずです。

でも実際はTDZのエラーが発生してるので、`const x = 2;`の宣言がスコープの先頭に巻き上げられ、TDZに入ってる状態ということでしょう。

個人的にも「`const`と`let`だから Hoisting が発生しないというわけではなくて、発生するにはするけどTDZで初期化完了前の変数にアクセスすることを禁止してるから Hoisting が起こってないように感じる」があってる気がします。


# 3. 関数の Hoisting

変数の整理が終わったので次に関数です。
関数も Hoisting の対象ですけど変数よりは内容が少ないのでサクッと見てみましょう。

## 3-1. 関数宣言

関数宣言をすると、それを囲む関数やグローバルスコープの先頭に巻き上げられ、関数を宣言する前に使うことができます。

```js
hoisted(); // "foo"

function hoisted() {
  console.log('foo');
}
```

宣言の前に使うこともできるところが変数とは違う部分ですね。

## 3-2. 関数式

でも関数式では宣言の前に関数を使うことはできません。

```js
notHoisted(); // notHoisted is not a function

var notHoisted = function() {
   console.log('bar');
};
```

理由は簡単です。関数式も値が関数なだけで、変数宣言キーワードを使う**変数宣言**です。だから先ほど学んだ**変数宣言キーワードの Hoisting と同じ挙動**になります。

上の例だと`var`で関数宣言してるので、宣言以前に`notHoisted`にアクセスしたら`undefined`のはずです。なのに関数実行してるから「関数ではない」というエラーになります。

では`var`を`const`に変えてみましょう。

```js
{
  notHoisted(); // Cannot access 'notHoisted' before initialization

  const notHoisted = function() {
    console.log('bar');
  }
};
```
`const`で変数宣言してるので、スコープの先頭から初期化までTDZが生成されます。関数コールしてる時点はTDZの内部なので「初期化前にはアクセスできない」というエラーになります。

こういうことがあるので、関数式で関数を定義するときは変数の Hoisting と同じ挙動をするということを理解しておいたらいいでしょう！


# 🌷 終わり

ここまでがざっくりとまとめた Javascript の Hoisting です。主にMDN見ながら書きました！

（なんとかアドカレ公開日までは間に合った、、！！）
