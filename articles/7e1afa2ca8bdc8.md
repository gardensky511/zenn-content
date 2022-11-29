---
title: "React で state は更新しても再レンダリングはしたくないときは useRef"
emoji: "👏"
type: "tech"
topics: ["reactnative", "react", "frontend"]
published: true
published_at: "2021-09-19 20:47"
---

最近実務の実装で起きた不具合をなんとか直したのでその話を共有したいと思います。

![](https://storage.googleapis.com/zenn-user-upload/f5979d305f2bd8894f787705.jpg)


## 1. 事件発生

### 1-1. やりたかったこと

こういうレイアウトを実装してました。

![](https://storage.googleapis.com/zenn-user-upload/83eec084ec64dad63a0f83c3.gif)

スクロールがタブより上にあるときはタブ押下時コンテンツが切り替わる、タブより下にあるときはコンテンツが切り替わるかつスクロールがコンテンツの最上部まで上がる、という仕様でした。


### 1-2. 起きたこと

ですが、実装してみたら問題が発生しました。

![](https://storage.googleapis.com/zenn-user-upload/d668ab2b63a76289505e445b.gif)

なぜかスクロールがタブを過ぎるたびにコンテンツが再読み込みされてしまうのです、、

## 2. 原因分析

原因は割とすぐわかりました。

スクロール位置によってスクロールを上げる動きを入れるかどうかが決まる仕様だったので、以下のコードで制御してました。

```javascript
const [shouldScrollToTop, setShouldScrollToTop] = useState(false);

const handleScroll = (event: NativeSyntheticEvent<NativeScrollEvent>) => {
    // スクロールが Overview より下の場合
    if (currentScrollPosition >= heightOfOverview) {
      if (shouldScrollToTop) return; // shouldScrollToTop が既に true なら return
      setShouldScrollToTop(true);
      return;
    }

    // スクロールが Overview より上の場合
    if (!shouldScrollToTop) return; // shouldScrollToTop が既に false なら return
    setShouldScrollToTop(false);
  }
```
（React Native のコードなので`event`の型が見慣れないものかもです）

`shouldScrollToTop`という state を生成し、スクロールイベントのコールバック関数で**スクロールがタブを過ぎるたびに`shouldScrollToTop`を更新**してます。

そして**それにつられコンポーネントが再レンダリングされる、というのが原因**でした。


## 3. 解決案

原因が state 更新により再レンダリングだとわかったので、次は解決案ですね。

仕様を守るために state 更新は必要だったので、不要な再レンダリングを防止する `useCallback` & `React.memo` は使えないと思いました。（しかも実際のコードではたくさんの state とコンポーネントが複雑に絡んでたので難しかったです…^_^）

だったら **state が更新しても再レンダリングに影響がないようにする**！という結論になり、 `useRef` にたどり着いたのです。

+) `useCallback` & `React.memo`の概要が知りたい方は私が以前書いた「[useCallbackは、本当にパフォーマンスを向上させる？](https://zenn.dev/luvmini511/articles/57c9aa40632ea3)」をご参考ください。

## 4. useRef

### 4-1. state管理

`useRef`はDOMにアクセスする手段でお馴染みかもしれませんが、状態管理にも使えるフックです。

よく使われる`useState`と違って`useRef`で管理する state は更新されてもコンポーネントの再レンダリングは起きません。[React 公式サイト](https://ja.reactjs.org/docs/hooks-reference.html#useref)にも同じことが書かれています。

> `useRef` は中身が変更になってもそのことを通知**しない**ということを覚えておいてください。`.current` プロパティを書き換えても再レンダーは発生しません。

つまり中身が変わっても React に教えないから再レンダリングも起きないということですね。これを利用します！

### 4-2. `useRef`の定義


[@types/react の index.d.ts](https://github.com/DefinitelyTyped/DefinitelyTyped/blob/beceb9d58e7e51562604ad9dfe10746af660218b/types/react/index.d.ts#L1032-L1073)を見ると、`useRef`には３つの定義があります。

**(1) `useRef<T>(initialValue: T): MutableRefObject<T>;`**
初期値の型とジェネリックで渡した型が一致する場合、`MutableRefObject<T>`を返します。
```typescript
// MutableRefObject<T>の定義
interface MutableRefObject<T> {
    current: T;
}
```
[型定義](https://github.com/DefinitelyTyped/DefinitelyTyped/blob/beceb9d58e7e51562604ad9dfe10746af660218b/types/react/index.d.ts#L903)でも名前でも分かるように、`current`プロパティはミュータブル、つまり直接修正できます。

**(2) `useRef<T>(initialValue: T|null): RefObject<T>;`**
初期値の型が`null`を許容する場合、`RefObject<T>`を返します。
```typescript
// RefObject<T>の定義
interface RefObject<T> {
    readonly current: T | null;
}
```
[型定義](https://github.com/DefinitelyTyped/DefinitelyTyped/blob/beceb9d58e7e51562604ad9dfe10746af660218b/types/react/index.d.ts#L88)で`current`プロパティに`readonly`をつけてます。つまりイミュータブル、直接修正できません。

**(3) `useRef<T = undefined>(): MutableRefObject<T | undefined>;`**
ジェネリックの型が`undefined`の場合（型を渡してない場合）、`MutableRefObject<T | undefined>`を返します。

色々ありますが、私は`true`と`false`しか持たない state を管理したいので、(1)でいきます。


## 5. 修正コード

解決案も探したので、コード修正しました。

```javascript
const shouldScrollToTop = useRef<boolean>(false);

const handleScroll = (event: NativeSyntheticEvent<NativeScrollEvent>) => {
    // スクロールが Overview より下の場合
    if (scrollPosition >= heightOfOverview) {
      if (shouldScrollToTop.current) return; // shouldScrollToTop が既に true なら return
      shouldScrollToTop.current = true;
      return;
    }

    // スクロールが Overview より上の場合
    if (!shouldScrollToTop.current) return; // shouldScrollToTop が既に false なら return
    shouldScrollToTop.current = false;
  };
```
`useRef`の初期値は`false`、ジェネリックは`boolean`で初期値とジェネリックの型が一致してます。ということで、ミュータブルな`.current`を返してるはずなので直接書き換えています。

これで state を更新しても React はわからない、そして再レンダリングも起きなくなりました。動作確認でも再読み込みが起きないこと確認できたので、一件落着です。

---

state 管理で`useRef`を使うのはメージャーな方法ではないと思いますが、知っといたらいつか役に立つかもしれません。

と言ってもこれは割と思いつきの解決案ですので、もっといい方法ご存知の方はぜひぜひ教えてくださいー！