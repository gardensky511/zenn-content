---
title: "このプロパティ、型定義に存在しないけどエラーになる？ならない？~ TS と Structural Typing ~"
emoji: "🦆"
type: "tech"
topics:
  - "javascript"
  - "typescript"
  - "web"
  - "frontend"
published: true
published_at: "2022-09-24 18:01"
---

# 🌼 はじめに

この記事を見に来てくださった皆さんにクイズを出します！

以下のような型と関数、変数があるとしましょう。

```ts
interface Post {
    title: string
    isPublic: boolean
}

interface Game {
    charactorName?: string
    isAlive?:boolean
}

const getPostTitle = (post: Post) => post.title

const post = {
    title: "typescriptを理解する",
    isPublic: true,
    author: "みんちゃん"
}
```
そして4つの問題があります。
```ts
// 問1. これはエラーになるでしょうか？
getPostTitle(post)

// 問2. これはエラーになるでしょうか？
const post1: Post = {
    title: "typescriptを理解する",
    isPublic: true,
    author: "みんちゃん"
}

// 問3. これはエラーになるでしょうか？
const post2: Post = post

// 問4. これはエラーになるでしょうか？
const rpg = {
    CHARACTOR_NAME: "勇者"
}
const newRpg: Game = rpg
```

正解は TS Playground で確認できます。

https://www.typescriptlang.org/play?ssl=36&ssc=25&pln=19&pc=1#code/JYOwLgpgTgZghgYwgAgAoHsDOZkG8BQyRyYwYANhAFzLZSgDmhxwmqArgEbnAI2fp0lOCHwBffPlCRYiFAHE4AWxQFiyBAAs4URGHRQAcsogB+GnUbMirAII8AbmaoChEEeMkJ0IbMgYQYBjYACpklMgAvMgAFAAOWGA0wWAAlFEAfMgJ2AB0pBQQXj5+OTjRasQFlDQARGAAnnEQmAj0cWCASQyAYC6Ax5GAmgyA0Qy1ADTWyKwc3Lw0YFDsEGPqcOxgmgZ1gP4MgMkMgIYMgMEM27We+AD0p8iA8qoAjLnIgMoMgDEMgPYMgBUMgJcMgD8MgNYMgFYMg4BzBkA6gyAcIZAGIMgGkGQD4-4AIf8AMuSASH-8AEgokwoV4olUpJzlcAEx3J5vL5-QGgyGwwCq1IB4P6R3l8ODK12SiSieHG1WoyHqTRabWAHR6A2GSxYbC4PD4JHmi3GKzWG25OwORxOuMuAGZCS8Pj9-sDwdD4XSSozEniWX5omUcRdLgAWbXEvVkw1U2n4el+KBxBhsypEADCAAlbAAlWyBkIAeTDAH1DLYALIAUTqgHGlQCgAccJF6cCAIAB3MO+miKFRsn0MIA

皆さん全問正解できましたか？

「え？」と思った方々もいらっしゃるかもしれません。今からなぜ typescript の世界観でこういうことが起きるのかを解説していきいきたいと思います！

# 1. Structural Typing

上記の問題を理解するのに必要な前提知識は、「**Javascriptは Duck Typing に基づいており、Typescript もそれをモデリングして Structural Typing に基づく**」ということです。

そもそも Duck Typing とは、、？

> "If it walks like a duck and it quacks like a duck, then it must be a duck"
> もしアヒルのように歩いてアヒルのように鳴くなら、それはきっとアヒルだ

![](https://storage.googleapis.com/zenn-user-upload/9c705d06ce41-20220923.jpeg)
*プラグを挿せる２つの穴があるなら、それはきっとコンセントだ(ｗ)*

javascript ではある関数に引数の値がちゃんと全部与えられたら、その値がどう作られたかは気にせず使います。

従って javascript をモデリングしてる typescript の世界観では、**ある型の最小限の条件を満たすオブジェクトはその型に割り当てることができます**。構造的(structual)に型をチェックしてると言えるかもしれません。

ここで問1の解説をしましょう。

```ts
interface Post {
    title: string
    isPublic: boolean
}

const getPostTitle = (post: Post) => post.title

const post = {
    title: "typescriptを理解する",
    isPublic: true,
    author: "みんちゃん"
}

// 問1. これはエラーになるでしょうか？（❌）
getPostTitle(post)
```
`post`は`Post`型で定義してる`title`と`isPublic`を持っていますが、定義してない`author`も持ってます。この場合`Post`型の引数を受け取る`getPostTitle`に`post`を引数として渡してもエラーにならないのです。

理由は、**`Post`型の最小限の条件を満たしてるので、他のプロパティがあってもpost は`Post`型に割り当てることができる**からです。



# 2. 余剰プロパティチェック

Typescript が Structural Typing に基づいてることを理解したので、問2も理解できそうな気がします。では正解を確認してみましょう。

```ts
interface Post {
    title: string
    isPublic: boolean
}

// 問2. これはエラーになるでしょうか？（⭕️）
const post1: Post = {
    title: "typescriptを理解する",
    isPublic: true,
    // Type '{ title: string; isPublic: true; author: string; }' is not assignable to type 'Post'.
    // Object literal may only specify known properties, and 'author' does not exist in type 'Post'.
    author: "みんちゃん"
}
```

「え？ Structural Typing の考え方的に最小限の条件を満たしてるからエラーにならないはずなのでは？話が違うんですけど」という気持ちになるかもしれません（私はなってました）

エラーになる理由は簡単です。実際話が違うからです。

問2の場合は、**TypeScriptの余剰プロパティチェック**というプロセスが走ってます。これは Structural Typing に基づいた通常の型チェックとはまた**別物**です。

> 余剰プロパティチェックとは、オブジェクト型に存在しないプロパティを持つオブジェクトの代入を禁止する検査です。

https://typescriptbook.jp/reference/values-types-variables/object/excess-property-checking

余剰プロパティチェックが行われたらオブジェクトリテラルに未知のプロパティを許可しないことができます。おかげでより厳密な型チェックができるし、タイポ防止にも役立つのでとても便利ですね。

これで問2も理解できました！

一つ注意点は、**余剰プロパティチェックはオブジェクトリテラル[^1]だけを検査**します。これが問3の解説です。

```ts
interface Post {
    title: string
    isPublic: boolean
}

const post = {
    title: "typescriptを理解する",
    isPublic: true,
    author: "みんちゃん"
}

// 問3. これはエラーになるでしょうか？（❌）
const post2: Post = post
```
`post`の右側はオブジェクトリタラルですが、問3の`post2`の右側はオブジェクトリタラルではありません。だから余剰プロパティチェック走らず、エラーになることもなくなるということです。

また、型アサーションの場合も余剰プロパティチェックは行われません。

```ts
interface Options {
    title: string;
}

const option1: Options = {
    title: "みんちゃん",
    // Type '{ title: string; feature: string; }' is not assignable to type 'Options'.
    // Object literal may only specify known properties, and 'feature' does not exist in type 'Options'.
    feature: "眠い"
}

const option2 = {
    title: "みんちゃん",
    feature: "眠い" // エラーなし、正常
} as Options
```

余剰プロパティチェックは便利ではありますが、こういう限界もあるので注意する必要がありそうですね。そしてやっぱアサーションよりちゃんと型宣言してあげましょう（^_^）


# 3. 弱い(weak)型の型チェック

では最後の問4です。正解は「エラーになる」でした。

```ts
interface Game {
    charactorName?: string
    isAlive?:boolean
}

// 問4. これはエラーになるでしょうか？（⭕️）
const rpg = {
    CHARACTOR_NAME: "勇者"
}
const newRpg: Game = rpg // Type '{ CHARACTOR_NAME: string; }' has no properties in common with type 'Game'.
```

「え？newRpg はオブジェクトリタラルでもないし、`Game`のプロパティは全部任意だから Structural Typing の考え方的にエラーにならないのでは？また話が違う」と思うかもしれません（私は思いました）

実は、また違う話なのです。

`Game`型はすべてのプロパティが任意なので、すべてのオブジェクトに割り当てられることができてしまいます。このように任意のプロパティだけ持つ型を弱い(weak)型とも言います。

そして**弱い型の場合は値と型に共通のプロパティがあるかどうかをチェックする**検査が走ります。余剰プロパティチェックと似てるように見えますが、別の検査です。（もちろん通常の型チェックとも別物）

だから型に割当てられない（is not assignable）のエラーではなく、共通プロパティがない（has no properties in common）というエラーが出たのでしょう！

ちなみに関数の引数に渡すときも同じ検査が行われます。

```ts
const playGame = (game: Game) => {
    // ...
}
const myGame = { myName: "みんちゃん" }
playGame(myGame) // Type '{ myName: string; }' has no properties in common with type 'Game'
```

# 5. まとめ

諸々説明が多かったのでまとめてみました。

:::message
- typescript は  Structural Typing に基づいてるので、ある型の最小限の条件を満たすオブジェクトはその型に割り当てることができる
- オブジェクトリテラルを変数に代入するときや関数に引数として渡すときは余剰プロパティチェックが走る
- 弱い型の場合は値と型に共通のプロパティがあるかどうかをチェックする処理が走る
:::

typescript の型チェックの動きを理解し、またそれとは別で余剰プロパティチェックと弱い型の共通プロパティチェックが存在することを知っといたらもっといいTSコードを書けるのではないかと思います。

# 🌷 終わり

Effective Typescript の内容一部を私なりに再構成してみました。これで typescirpt に対する理解がもっと深まった気がします！



[^1]: オブジェクトリタラルの定義は js primer を参考にしましょう: https://jsprimer.net/basic/data-type/#object