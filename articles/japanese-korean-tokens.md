---
title: "AI 基礎② なぜ日本語は英語よりトークン消費量が多いのか"
emoji: "🧩"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["ai", "llm", "tokenizer", "nlp"]
published: false
---

## 🌼 はじめに

この記事を見ている方々は普段 AI を使うとき日本語を使う方が多いでしょう。ちなみに私は韓国語・日本語混ぜて使ってます。

でもなんか日本語・韓国語だと英語よりトークン消費量が多いらしいですね。その理由が気になったので、今回はそもそもトークンとは何なのか、消費量はどう決まるのかについて話したいと思います。

後半では、GPT と Claude のトークナイザーで実際にトークンを数えてみた結果も紹介します。

## 1. トークンとは

**トークンとは、文章を細かく分けた、ひとつひとつのかたまり**です。LLM は文章をそのまま読むのではなく、トークンに分けてから処理します。

この分ける作業をするのが、**トークナイザー**というプログラムです。LLM が動く前に文章をトークン単位で区切って、トークンごとに ID（トークン ID）をつけます。

```html-embed
<div class="ti">
  <div class="row"><div class="lb">入力</div><div class="sent">猫が好きです。</div></div>
  <div class="arrow"><i></i><b>トークナイザー</b><span>① トークン単位で区切る</span></div>
  <div class="row"><div class="lb">トークン</div>
    <div class="cells"><i>猫</i><i>が</i><i>好き</i><i>です</i><i>。</i></div></div>
  <div class="arrow"><i></i><b>トークナイザー</b><span>② トークンごとに ID をつける</span></div>
  <div class="row"><div class="lb">出力</div>
    <div class="cells ids"><i>48091</i><i>6632</i><i>144444</i><i>15121</i><i>788</i></div></div>
  <div class="note">※ o200k_base（GPT-4o・GPT-5 が使っているトークナイザー）で確認</div>
</div>
<style>
  .ti { --bg:#f7f8fa; --text:#1f2937; --muted:#6b7280; --gray:#eceff3; --blue:#2f6fc8; --blue-bg:#e6effb;
        background:var(--bg); color:var(--text); padding:16px; display:flex; flex-direction:column; gap:10px;
        font-family:"Hiragino Sans","Hiragino Kaku Gothic ProN","Noto Sans JP","Yu Gothic",sans-serif; font-size:14px; line-height:1.5; }
  [data-theme="dark"] .ti { --bg:#1e2430; --text:#e5e7eb; --muted:#9aa3b2; --gray:#2b3240; --blue:#8bb8f5; --blue-bg:#24344d; }
  .ti .row { display:grid; grid-template-columns:5.5em 1fr; gap:10px; align-items:center; }
  .ti .lb { font-weight:700; color:var(--muted); font-size:13px; }
  .ti .sent { background:var(--gray); border-radius:6px; padding:4px 12px; justify-self:start; font-weight:700; }
  .ti .arrow { color:var(--muted); font-size:13px; padding-left:calc(5.5em + 10px); display:grid; grid-template-columns:10px 1fr; column-gap:8px; }
  .ti .arrow i { grid-row:1 / 3; position:relative; background:none; padding:0; border-radius:0; }
  .ti .arrow i::before { content:""; position:absolute; left:4px; top:3px; bottom:7px; width:2px; background:var(--muted); }
  .ti .arrow i::after { content:""; position:absolute; left:0; bottom:1px; border-left:5px solid transparent; border-right:5px solid transparent; border-top:7px solid var(--muted); }
  .ti .arrow b { color:var(--blue); }
  .ti .arrow b, .ti .arrow span { grid-column:2; }
  .ti .cells { display:grid; grid-template-columns:repeat(5, minmax(0, 4.8em)); gap:6px; }
  .ti i { font-style:normal; text-align:center; border-radius:6px; padding:3px 4px; font-weight:700; background:var(--gray); }
  .ti .ids i { background:var(--blue-bg); color:var(--blue); font-family:ui-monospace,Menlo,monospace; font-size:13px; }
  .ti .note { font-size:12px; color:var(--muted); margin-top:4px; }
  @media (max-width: 480px) {
    .ti .row { grid-template-columns:1fr; gap:4px; }
    .ti .arrow { padding-left:0; }
    .ti .cells { grid-template-columns:repeat(5, minmax(0, 1fr)); gap:4px; }
    .ti .cells i { padding:3px 2px; }
    .ti .ids i { font-size:11px; }
  }
</style>
```

こうしてできたトークン ID の並びが、LLM に渡されます。

`猫`が `48091` になるのは、トークナイザーが「`猫` は `48091` に変換する」というデータを持っているからです。このデータを**語彙**と呼びます。語彙は、トークンと ID の対応をまとめた一覧で、トークナイザーが事前にたくさんの文章を学習して作ります。

```html-embed
<div class="vc">
  <div class="ttl">語彙の例（o200k_base の一部）</div>
  <div class="grid">
    <div class="e hit"><b>。</b><i>→</i><span>788</span></div>
    <div class="e"><b>the</b><i>→</i><span>3086</span></div>
    <div class="e"><b>は</b><i>→</i><span>5205</span></div>
    <div class="e hit"><b>が</b><i>→</i><span>6632</span></div>
    <div class="e"><b>cat</b><i>→</i><span>8837</span></div>
    <div class="e"><b>日本</b><i>→</i><span>9048</span></div>
    <div class="e hit"><b>です</b><i>→</i><span>15121</span></div>
    <div class="e"><b>dog</b><i>→</i><span>30146</span></div>
    <div class="e hit"><b>猫</b><i>→</i><span>48091</span></div>
    <div class="e"><b>ありがとう</b><i>→</i><span>55300</span></div>
    <div class="e"><b>こんにちは</b><i>→</i><span>95839</span></div>
    <div class="e"><b>犬</b><i>→</i><span>129581</span></div>
    <div class="e hit"><b>好き</b><i>→</i><span>144444</span></div>
    <div class="e more">…</div>
  </div>
  <div class="note"><span class="sw"></span>「猫が好きです。」で使われたトークン</div>
  <div class="total">全部で <b>約 20 万個</b></div>
</div>
<style>
  .vc { --bg:#f7f8fa; --text:#1f2937; --muted:#6b7280; --card:#ffffff; --line:#dfe3e8; --blue:#2f6fc8; --blue-bg:#e6effb;
        background:var(--bg); color:var(--text); padding:14px 16px; display:flex; flex-direction:column; gap:10px;
        font-family:"Hiragino Sans","Hiragino Kaku Gothic ProN","Noto Sans JP","Yu Gothic",sans-serif; font-size:14px; line-height:1.4; }
  [data-theme="dark"] .vc { --bg:#1e2430; --text:#e5e7eb; --muted:#9aa3b2; --card:#2b3240; --line:#343c4b; --blue:#8bb8f5; --blue-bg:#24344d; }
  .vc .ttl { font-weight:700; font-size:13px; color:var(--muted); }
  .vc .grid { display:grid; grid-template-columns:repeat(4, 1fr); gap:6px; }
  .vc .e { display:flex; align-items:center; gap:5px; background:var(--card); border:1px solid var(--line); border-radius:6px; padding:4px 8px; white-space:nowrap; }
  .vc .e b { font-weight:700; white-space:nowrap; }
  .vc .e i { font-style:normal; color:var(--muted); font-size:12px; }
  .vc .e span { font-family:ui-monospace,Menlo,monospace; font-size:12px; color:var(--muted); }
  .vc .e.hit { background:var(--blue-bg); border-color:var(--blue); }
  .vc .e.hit b, .vc .e.hit span { color:var(--blue); }
  .vc .e.more { display:flex; justify-content:center; color:var(--muted); }
  .vc .total { font-size:13px; text-align:center; }
  .vc .total b { color:var(--blue); font-size:15px; }
  .vc .note { font-size:12px; color:var(--muted); display:flex; align-items:center; gap:6px; margin-top:-4px; }
  .vc .sw { width:12px; height:12px; border-radius:3px; background:var(--blue-bg); border:1px solid var(--blue); }
  @media (max-width: 560px) {
    .vc .grid { grid-template-columns:repeat(2, 1fr); gap:5px; }
    .vc .e { padding:4px 6px; gap:4px; font-size:13px; }
    .vc .e span { font-size:11px; }
  }
</style>
```

では、トークンの単位はどうやって決まるのでしょうか？

トークナイザーは、「単語で分ける」「1文字ずつ分ける」のように前もって決めておくのではなく、**たくさんの学習用文章を見て、よく出てくるかたまりをトークンにします**。

たとえば o200k_base では、`ありがとう` はまるごと 1 トークンですが、`日本料理` は `日本` `料理` の 2 トークンに分かれます。`ありがとう` は学習用文章によく出てきて、`日本料理` はひとかたまりとしてはそれほど出てこなかったかもしれません。

その結果、学習用文章での出現頻度が高い言葉はトークンが単語や形態素くらいの大きさになり、出現頻度が低い言葉ほど細かく分かれて、1文字ずつになることもあります。

```html-embed
<div class="ts">
  <div class="axw">
    <div class="cap">学習用文章で</div>
    <div class="axis"><span class="line"></span><b class="l">よく見る</b><b class="r">ほとんど見ない</b></div>
  </div>
  <div class="cols">
    <div class="c">
      <div class="w">ありがとう</div>
      <div class="down">↓</div>
      <div class="chips"><i>ありがとう</i></div>
      <div class="n"><b>1</b> トークン</div>
      <div class="d">まるごと 1 つ</div>
    </div>
    <div class="c">
      <div class="w">日本料理</div>
      <div class="down">↓</div>
      <div class="chips"><i>日本</i><i>料理</i></div>
      <div class="n"><b>2</b> トークン</div>
      <div class="d"><span>知っている</span><span>かたまりに分ける</span></div>
    </div>
    <div class="c">
      <div class="w">形態素</div>
      <div class="down">↓</div>
      <div class="chips"><i>形</i><i>態</i><i>素</i></div>
      <div class="n"><b>3</b> トークン</div>
      <div class="d"><span>1文字ずつ</span><span>まで分ける</span></div>
    </div>
  </div>
  <div class="src">※ o200k_base で確認</div>
</div>
<style>
  .ts { --bg:#f7f8fa; --text:#1f2937; --muted:#6b7280; --gray:#eceff3; --blue:#2f6fc8; --blue-bg:#e6effb; --line-c:#a3aab5;
        background:var(--bg); color:var(--text); padding:16px; display:flex; flex-direction:column; gap:14px;
        font-family:"Hiragino Sans","Hiragino Kaku Gothic ProN","Noto Sans JP","Yu Gothic",sans-serif; font-size:14px; line-height:1.5; }
  [data-theme="dark"] .ts { --bg:#1e2430; --text:#e5e7eb; --muted:#9aa3b2; --gray:#2b3240; --blue:#8bb8f5; --blue-bg:#24344d; --line-c:#5b6474; }
  .ts b { color:var(--blue); }
  .ts .axw { display:flex; flex-direction:column; gap:2px; }
  .ts .cap { text-align:center; font-size:13px; color:var(--muted); }
  .ts .axis { display:grid; grid-template-columns:repeat(3,1fr); gap:10px; align-items:center; font-size:13px; color:var(--muted); }
  .ts .axis b { grid-row:1; justify-self:center; white-space:nowrap; background:var(--bg); padding:0 6px; position:relative; z-index:1; }
  .ts .axis b.l { grid-column:1; }
  .ts .axis b.r { grid-column:3; padding-left:0; display:flex; align-items:center; }
  .ts .axis b.r::before { content:""; border-top:5px solid transparent; border-bottom:5px solid transparent; border-left:8px solid var(--line-c); margin-right:6px; }
  .ts .axis .line { grid-row:1; grid-column:1 / 4; height:2px; background:var(--line-c); margin:0 16%; }
  .ts .cols { display:grid; grid-template-columns:repeat(3,1fr); gap:10px; }
  .ts .c { display:flex; flex-direction:column; align-items:center; text-align:center; gap:4px; background:#fff0; border-radius:10px; }
  .ts .w { background:var(--gray); border-radius:6px; padding:2px 10px; font-family:ui-monospace,Menlo,monospace; font-weight:700; }
  .ts .down { color:var(--muted); line-height:1; }
  .ts .chips { display:flex; gap:4px; flex-wrap:wrap; justify-content:center; }
  .ts i { font-style:normal; background:var(--blue-bg); color:var(--blue); font-weight:700; border-radius:6px; padding:2px 8px; font-family:ui-monospace,Menlo,monospace; }
  .ts .n { font-weight:700; margin-top:2px; }
  .ts .n b { font-size:18px; }
  .ts .d { font-size:12px; color:var(--muted); white-space:nowrap; }
  .ts .src { font-size:12px; color:var(--muted); }
  @media (max-width: 480px) {
    .ts { padding:14px 10px; }
    .ts .cols, .ts .axis { gap:6px; }
    .ts .axis { font-size:12px; }
    .ts .w { padding:2px 6px; font-size:13px; }
    .ts i { padding:2px 5px; font-size:13px; }
    .ts .chips { gap:3px; }
    .ts .n b { font-size:16px; }
    .ts .n { font-size:13px; }
    .ts .d { white-space:normal; font-size:11px; }
    .ts .d span { display:inline-block; }
  }
</style>
```

トークンが多くなるとトークナイザーの出力であるトークン ID の並びも長くなります。トークン ID の並びが長くなると、当然それを処理するコストも高くなります。

そう考えると、「なぜ日本語・韓国語は英語よりトークン消費量が多いのか？」という疑問は、「**なぜ日本語・韓国語は英語よりトークン ID の並びが長くなるのか？**」とも言い換えられます。

## 2. トークンの数はどう決まるのか

日本語・韓国語のトークン ID の並びがなぜ長くなるのか。その理由を探るために、まずは文字がコンピューターにどう保存されるかを簡単に説明します。

文字を保存するには、エンコーディング（文字を byte に変換する方法）が必要です。一番よく使われているエンコーディングは UTF-8 で、Web のほとんどが UTF-8 を使っています。

UTF-8 では、1文字を保存するのに必要な byte 数が文字によって違います。

| 文字の種類 | byte 数 |
|---|---:|
| 英語のアルファベットなど（ASCII、最初の 128 文字） | 1 |
| ギリシャ文字、キリル文字、アラビア文字など | 2 |
| 日本語・韓国語・中国語の多く | 3 |
| 珍しい漢字、絵文字など | 4 |

日本語や韓国語は 1文字が 3 byte で英語の 3倍なのでそのぶんトークンを多く使うと思うかもしれないですが、実は同じ意味を英語より少ない文字数で表せるという側面もあります。

例えば `私は毎朝コーヒーを飲みます。` は 14文字、韓国語の `저는 매일 아침 커피를 마셔요.` は 17文字ですが、英語の `I drink coffee every morning.` は 29文字です。

この 3 つの文を、byte 数とトークン数でも比べてみました。

| 文 | 文字数 | byte 数 | トークン数 |
|---|---:|---:|---:|
| I drink coffee every morning. | 29 | 29 | 6 |
| 私は毎朝コーヒーを飲みます。 | 14 | 42 | 12 |
| 저는 매일 아침 커피를 마셔요. | 17 | 41 | 13 |

※ トークン数は o200k_base で確認

英語と比べると、日本語・韓国語は文字数では半分くらいで **byte 数では 1.4倍**くらいなのに、**トークン数では 2倍以上**です。byte 数の差（1.4倍）よりも、トークン数の差（2倍以上）のほうが大きいです。

この差はどこから来るのか？それを知るために、トークナイザーが語彙を作る方法を説明します。

トークナイザーは事前にたくさんの文章を学習して語彙を作ります。その学習方法としてよく使われているもののひとつが、**BPE**（**Byte Pair Encoding**）です。

BPE では、学習用文章の中で並んで出てくる回数が一番多い 2 つのトークンを、1 つにくっつけます。これを何度も繰り返します。

```html-embed
<div class="bp">
  <div class="ttl">例：学習用文章が <code>The news is never new.</code> <span class="nb">の場合</span></div>
  <div class="st">
    <div class="hd"><b>最初</b><span>1文字ずつ</span></div>
    <div class="seq"><span class="wd"><i>T</i><i>h</i><i>e</i></span><span class="wd"><i>n</i><i>e</i><i>w</i><i>s</i></span><span class="wd"><i>i</i><i>s</i></span><span class="wd"><i>n</i><i>e</i><i>v</i><i>e</i><i>r</i></span><span class="wd"><i>n</i><i>e</i><i>w</i></span><span class="wd"><i>.</i></span></div>
    <div class="vo"><em>語彙</em><i>T</i><i>h</i><i>e</i><i>n</i><i>w</i><i>s</i><i>i</i><i>v</i><i>r</i><i>.</i></div>
  </div>
  <div class="ar"><span>↓</span><b>「n e」</b><span class="lo">が一番多く並んで出てくる</span><span class="sh">が一番多い</span>（3回）<em>→ くっつける</em></div>
  <div class="st">
    <div class="hd"><b>1回目</b></div>
    <div class="seq"><span class="wd"><i>T</i><i>h</i><i>e</i></span><span class="wd"><i class="nw">ne</i><i>w</i><i>s</i></span><span class="wd"><i>i</i><i>s</i></span><span class="wd"><i class="nw">ne</i><i>v</i><i>e</i><i>r</i></span><span class="wd"><i class="nw">ne</i><i>w</i></span><span class="wd"><i>.</i></span></div>
    <div class="vo"><em>語彙</em><i>T</i><i>h</i><i>e</i><i>n</i><i>w</i><i>s</i><i>i</i><i>v</i><i>r</i><i>.</i><i class="nw">ne</i></div>
  </div>
  <div class="ar"><span>↓</span><b>「ne w」</b><span class="lo">が一番多く並んで出てくる</span><span class="sh">が一番多い</span>（2回）<em>→ くっつける</em></div>
  <div class="st">
    <div class="hd"><b>2回目</b></div>
    <div class="seq"><span class="wd"><i>T</i><i>h</i><i>e</i></span><span class="wd"><i class="nw">new</i><i>s</i></span><span class="wd"><i>i</i><i>s</i></span><span class="wd"><i>ne</i><i>v</i><i>e</i><i>r</i></span><span class="wd"><i class="nw">new</i></span><span class="wd"><i>.</i></span></div>
    <div class="vo"><em>語彙</em><i>T</i><i>h</i><i>e</i><i>n</i><i>w</i><i>s</i><i>i</i><i>v</i><i>r</i><i>.</i><i>ne</i><i class="nw">new</i></div>
  </div>
  <div class="lg"><span class="sw nw-sw"></span>新しくできたトークン</div>
  <div class="src">※ 実際の学習用文章はずっと多いです（例：日本語 LLM の Swallow では約 15億語分の日本語の文章で語彙を作る）</div>
</div>
<style>
  .bp { --bg:#f7f8fa; --text:#1f2937; --muted:#6b7280; --card:#ffffff; --line:#dfe3e8; --gray:#eceff3; --blue:#2f6fc8; --blue-bg:#e6effb; --orange:#d9822b;
        background:var(--bg); color:var(--text); padding:16px; display:flex; flex-direction:column; gap:6px;
        font-family:"Hiragino Sans","Hiragino Kaku Gothic ProN","Noto Sans JP","Yu Gothic",sans-serif; font-size:14px; line-height:1.5; }
  [data-theme="dark"] .bp { --bg:#1e2430; --text:#e5e7eb; --muted:#9aa3b2; --card:#2b3240; --line:#3a4252; --gray:#353d4c; --blue:#8bb8f5; --blue-bg:#24344d; --orange:#f0a55a; }
  .bp .ttl { font-weight:700; font-size:13px; margin-bottom:4px; }
  .bp .ttl code { font-family:ui-monospace,Menlo,monospace; background:var(--gray); border-radius:5px; padding:1px 6px; }
  .bp .st { background:var(--card); border:1px solid var(--line); border-radius:10px; padding:10px 12px; display:flex; flex-direction:column; gap:8px; }
  .bp .hd { display:flex; align-items:baseline; gap:8px; }
  .bp .hd b { font-size:13px; }
  .bp .hd span { font-size:12px; color:var(--muted); }
  .bp .seq, .bp .vo { display:flex; flex-wrap:wrap; align-items:center; gap:4px; }
  .bp i { font-style:normal; font-family:ui-monospace,Menlo,monospace; font-weight:700; font-size:13px; background:var(--gray); border-radius:5px; padding:1px 7px; min-width:1.2em; text-align:center; }
  .bp i.nw { background:var(--blue-bg); color:var(--blue); }
  .bp .pr { display:inline-flex; gap:3px; padding:2px; border:2px dashed var(--orange); border-radius:7px; }
  .bp .vo { font-size:12px; }
  .bp .wd { display:inline-flex; align-items:center; gap:3px; }
  .bp .seq { column-gap:12px; }
  .bp .vo em { font-style:normal; color:var(--muted); font-size:12px; margin-right:2px; }
  .bp .vo i { font-size:12px; padding:0 6px; }
  .bp .ar { font-size:12px; color:var(--muted); padding:2px 0 2px 14px; }
  .bp .ar span:first-child { margin-right:6px; }
  .bp .ar b { color:var(--orange); }
  .bp .ar .sh { display:none; }
  .bp .ar em { font-style:normal; display:inline-block; margin-left:4px; }
  .bp .lg { font-size:12px; color:var(--muted); display:flex; align-items:center; gap:6px; flex-wrap:wrap; margin-top:4px; }
  .bp .sw { width:14px; height:12px; border-radius:3px; display:inline-block; }
  .bp .pr-sw { border:2px dashed var(--orange); }
  .bp .nw-sw { background:var(--blue-bg); border:1px solid var(--blue); }
  @media (max-width: 480px) {
    .bp { padding:12px 10px; gap:4px; }
    .bp .st { padding:6px 8px; gap:4px; }
    .bp .lg { margin-top:2px; }
    .bp .ttl { margin-bottom:2px; }
    .bp .seq { gap:4px; column-gap:9px; }
    .bp .wd { gap:2px; }
    .bp .ar .lo { display:none; }
    .bp .ar .sh { display:inline; }
    .bp .ttl { font-size:12px; }
    .bp i { font-size:12px; padding:1px 4px; }
    .bp .pr { gap:2px; padding:1px; }
    .bp .vo i { padding:0 4px; }
    .bp .ar { padding-left:4px; letter-spacing:-0.02em; }
  }
  .bp .wd { background:var(--gray); border-radius:6px; padding:3px; gap:2px; }
  .bp .wd i { background:var(--card); border-radius:4px; padding:0 4px; min-width:0; }
  .bp .wd i.nw { background:var(--blue-bg); }
  .bp .wd .pr { padding:0 1px; gap:1px; }
  .bp .seq { column-gap:6px; row-gap:6px; }
  .bp .vo { gap:2px; }
  .bp .vo i { background:transparent; border:1px solid var(--line); border-radius:4px; padding:0 4px; min-width:0; }
  .bp .vo i.nw { background:var(--blue-bg); }
  @media (max-width: 480px) {
    .bp .wd { padding:2px; gap:2px; }
    .bp .wd i { padding:0 3px; }
    .bp .seq { column-gap:4px; }
  }
  .bp .wd { background:transparent; padding:0; gap:3px; }
  .bp .wd i { background:var(--gray); padding:1px 5px; }
  .bp .wd i.nw { background:var(--blue-bg); }
  .bp .seq { column-gap:12px; }
  @media (max-width: 480px) {
    .bp .wd { padding:0; gap:2px; }
    .bp .wd i { padding:1px 4px; }
    .bp .seq { column-gap:9px; }
  }
  .bp .nb { white-space:nowrap; }
  .bp .src { font-size:12px; color:var(--muted); margin-top:2px; }
  @media (max-width: 480px) {
    .bp { gap:3px; padding:10px 8px; }
    .bp .st { padding:5px 7px; gap:3px; }
    .bp .seq { row-gap:4px; }
    .bp .ar { padding-top:0; padding-bottom:0; }
    .bp .src { margin-top:10px; }
    .bp .wd { gap:2px; }
    .bp .wd i { padding:1px 2px; }
    .bp .seq { column-gap:6px; }
  }
</style>
```

こうして学習でできた `n` `e` → `ne`、`ne` `w` → `new` のようなくっつけるルールは、新しい文章をトークンに分けるときにもそのまま使われます。たとえば学習で見ていない `anew` を渡されても、`a` `new` の 2 トークンに分かれます。

つまり、**ある文章が何トークンになるかはトークナイザーが学習のときに見た文章で決まります**。

ちなみにさきほどの例では 1文字ずつから始めましたが、実際の GPT などのトークナイザーは、文章を UTF-8 の byte に変換して、1 byte ずつから始めます。

たとえば `愛` は UTF-8 でエンコードすると `E6 84 9B` になるので、`E6` `84` `9B` の 3 つのかたまりから学習が始まります。学習用文章に `愛` がよく出てくれば `E6849B` という 1 つのトークンになり、あまり出てこなければ `E6` `84` `9B` のまま、3トークンになります。`E684` `9B` のように、一部だけくっついて 2 トークンになることもあります。

```html-embed
<div class="ab">
  <div class="st"><span class="lbl">UTF-8</span><code class="ch">愛</code><div class="row"><i>E6</i><i>84</i><i>9B</i></div></div>
  <div class="br">
    <div class="op">
      <div class="cd">学習用文章に<b>よく出てくる</b></div>
      <div class="rt"><span class="to">→</span><i class="nw">E6849B</i><span class="n"><b>1</b> トークン</span></div>
    </div>
    <div class="op">
      <div class="cd"><b>一部だけ</b>くっつく</div>
      <div class="rt"><span class="to">→</span><i class="nw">E684</i><i>9B</i><span class="n"><b>2</b> トークン</span></div>
    </div>
    <div class="op">
      <div class="cd">学習用文章に<b>ほとんど出てこない</b></div>
      <div class="rt"><span class="to">→</span><i>E6</i><i>84</i><i>9B</i><span class="n"><b>3</b> トークン</span></div>
    </div>
  </div>
</div>
<style>
  .ab { --bg:#f7f8fa; --text:#1f2937; --muted:#6b7280; --card:#ffffff; --line:#dfe3e8; --gray:#eceff3; --blue:#2f6fc8; --blue-bg:#e6effb;
        background:var(--bg); color:var(--text); padding:16px; display:grid; grid-template-columns:auto 1fr; gap:14px; align-items:center;
        font-family:"Hiragino Sans","Hiragino Kaku Gothic ProN","Noto Sans JP","Yu Gothic",sans-serif; font-size:14px; line-height:1.5; }
  [data-theme="dark"] .ab { --bg:#1e2430; --text:#e5e7eb; --muted:#9aa3b2; --card:#2b3240; --line:#3a4252; --gray:#353d4c; --blue:#8bb8f5; --blue-bg:#24344d; }
  .ab code, .ab i { font-family:ui-monospace,Menlo,monospace; }
  .ab .st { display:flex; flex-direction:column; align-items:center; gap:4px; background:var(--card); border:1px solid var(--line); border-radius:10px; padding:10px 14px; }
  .ab .lbl { font-size:12px; color:var(--muted); }
  .ab .ch { font-size:22px; font-weight:700; line-height:1.2; }
  .ab .row { display:flex; gap:3px; }
  .ab i { font-style:normal; font-weight:700; font-size:13px; background:var(--gray); border-radius:5px; padding:1px 7px; }
  .ab i.nw { background:var(--blue-bg); color:var(--blue); }
  .ab .br { display:flex; flex-direction:column; gap:10px; border-left:2px solid var(--line); padding-left:14px; }
  .ab .op { background:var(--card); border:1px solid var(--line); border-radius:10px; padding:8px 12px; display:flex; flex-direction:column; gap:4px; }
  .ab .cd { font-size:12px; color:var(--muted); }
  .ab .cd b { color:var(--text); margin-left:2px; }
  .ab .rt { display:flex; align-items:center; gap:4px; flex-wrap:wrap; }
  .ab .to { color:var(--muted); margin-right:2px; }
  .ab .n { margin-left:8px; font-weight:700; white-space:nowrap; }
  .ab .n b { color:var(--blue); font-size:18px; }
  @media (max-width: 480px) {
    .ab { padding:12px 10px; gap:8px; }
    .ab .st { padding:8px; }
    .ab .br { padding-left:8px; gap:8px; }
    .ab .op { padding:6px 8px; }
    .ab i { padding:1px 4px; font-size:12px; }
    .ab .n { margin-left:4px; }
    .ab .n b { font-size:16px; }
  }
  .ab .cd b { white-space:nowrap; }
  .ab .n { flex-basis:100%; margin-left:0; margin-top:2px; }
</style>
```

この仕組みはどの言語でも同じです。`love`（4 byte）や `사랑`（6 byte）も、くっつけば 1 トークン、くっつかなければ byte の数だけトークンになります。

```html-embed
<div class="lb">
  <div class="gr">
    <span class="h"></span><span class="h">UTF-8 の byte</span><span class="h c"><span class="lo">くっつけば</span><span class="sh">くっつく</span></span><span class="h c"><span class="lo">くっつかなければ</span><span class="sh"><span class="nb">くっつか</span><span class="nb">ない</span></span></span>
    <code class="w">love</code><div class="by"><i>6C</i><i>6F</i><i>76</i><i>65</i></div><span class="c one"><b>1</b><span class="u"> トークン</span></span><span class="c many"><b>4</b><span class="u"> トークン</span></span>
    <code class="w">愛</code><div class="by"><i>E6</i><i>84</i><i>9B</i></div><span class="c one"><b>1</b><span class="u"> トークン</span></span><span class="c many"><b>3</b><span class="u"> トークン</span></span>
    <code class="w">사랑</code><div class="by"><i>EC</i><i>82</i><i>AC</i><i>EB</i><i>9E</i><i>91</i></div><span class="c one"><b>1</b><span class="u"> トークン</span></span><span class="c many"><b>6</b><span class="u"> トークン</span></span>
  </div>
  <div class="note">※ 数字はトークン数</div>
</div>
<style>
  .lb { --bg:#f7f8fa; --text:#1f2937; --muted:#6b7280; --card:#ffffff; --line:#dfe3e8; --gray:#eceff3; --blue:#2f6fc8; --blue-bg:#e6effb; --orange:#c96f1c;
        background:var(--bg); color:var(--text); padding:16px;
        font-family:"Hiragino Sans","Hiragino Kaku Gothic ProN","Noto Sans JP","Yu Gothic",sans-serif; font-size:14px; line-height:1.5; }
  [data-theme="dark"] .lb { --bg:#1e2430; --text:#e5e7eb; --muted:#9aa3b2; --card:#2b3240; --line:#3a4252; --gray:#353d4c; --blue:#8bb8f5; --blue-bg:#24344d; --orange:#f0a55a; }
  .lb code, .lb i { font-family:ui-monospace,Menlo,monospace; }
  .lb .gr { display:grid; grid-template-columns:auto auto auto auto; justify-content:start; align-items:center; column-gap:18px; row-gap:8px;
            background:var(--card); border:1px solid var(--line); border-radius:10px; padding:10px 14px; }
  .lb .h { font-size:12px; color:var(--muted); }
  .lb .nb { display:inline-block; }
  .lb .sh, .lb .note { display:none; }
  .lb .note { font-size:12px; color:var(--muted); margin-top:6px; }
  .lb .c { text-align:center; }
  .lb .w { font-weight:700; }
  .lb .by { display:flex; gap:3px; }
  .lb i { font-style:normal; font-weight:700; font-size:13px; background:var(--gray); border-radius:5px; padding:1px 6px; }
  .lb .one, .lb .many { font-weight:700; white-space:nowrap; font-size:13px; }
  .lb .one b { color:var(--blue); font-size:17px; }
  .lb .many b { color:var(--orange); font-size:17px; }
  @media (max-width: 480px) {
    .lb { padding:12px 10px; }
    .lb .gr { column-gap:8px; padding:8px; }
    .lb i { padding:1px 3px; font-size:12px; }
    .lb .h { font-size:12px; }
    .lb .one, .lb .many { font-size:12px; }
    .lb .one b, .lb .many b { font-size:16px; }
    .lb .u { display:none; }
    .lb .lo { display:none; }
    .lb .sh { display:inline; }
    .lb .note { display:block; }
  }
  @media (max-width: 480px) {
    .lb .gr { column-gap:6px; padding:8px 6px; }
    .lb .by { gap:2px; }
    .lb i { padding:1px 2px; }
    .lb .h.c { white-space:nowrap; line-height:1.3; }
    .lb .sh .nb { display:block; }
  }
</style>
```

くっつかないと byte の数だけトークンになるので、byte 数もトークン数にある程度は関係しています。でもくっつけば何 byte でも 1 トークンです。なので、トークン数を大きく左右するのは「くっつくかどうか」のほうです。

## 3. なぜ英語はトークンが少ないのか

さっきは例文 1 つで比べましたが、たくさんの文で比べたらどうでしょう。

ある研究^[Petrov et al. (2023). Language Model Tokenizers Introduce Unfairness Between Languages. https://arxiv.org/abs/2305.15425]で同じ意味の文約 2,000 個を同じトークナイザーで数えています。英語を 1 としたときの結果がこちらです。

| | 文字数 | byte 数 | トークン数 |
|---|---:|---:|---:|
| 日本語 | 0.44 | 1.27 | 2.30 |
| 韓国語 | 0.51 | 1.20 | 2.38 |

※ トークン数は 2023 年当時の cl100k_base

日本語・韓国語は、同じ意味の文を英語の半分くらいの文字数で書けるので byte 数の差はそれほど大きくないのに、トークンは 2倍以上使います。

この差の原因は、さきほど見た「くっつくかどうか」です。くっつくかどうかはトークナイザーが学習のときに見た文章で決まるので、よく出てくる言葉はくっついてトークン数が少なくなり、そうでもない言葉はくっつかずにトークン数が多くなります。

ちなみにここでいう「よく出てくる」は、世の中でよく使われるという意味ではなく**学習用文章の中でよく出てくる**という意味です。

たとえば GPT-2 のトークナイザーでは、`ゼウス` は 1トークンなのに、日常的に使う漢字 `言` は byte 単位のまま残り 3 トークンになります。`ゼウス` は GPT-2 の学習用文章（インターネットから集めた文章）によく出てきたのではないかと思います。

大事なのは、多くの場合その**学習用文章は圧倒的に英語が多い**ことです。その結果、英語はよくくっつき、ほかの言語は言葉がより細かいトークンに分かれてしまいます。これが英語のトークン消費量が少ない大きな理由でしょう。

なら、日本語・韓国語の文章も学習に使えば日本語・韓国語のトークン消費量も少なくなる気がしませんか。それを試した研究が日本にも韓国にもあって、実際にその結果トークン消費量が減りました。

```html-embed
<div class="ve">
  <div class="it">
    <div class="hd"><b>🇯🇵 Swallow</b><span>Llama 2 の語彙に日本語トークンを 11,176 個追加</span></div>
    <div class="br"><span class="lb">追加前</span><div class="tr"><div class="bar old" style="width:100%"></div></div><span class="v">100</span></div>
    <div class="br"><span class="lb">追加後</span><div class="tr"><div class="bar new" style="width:43.8%"></div></div><span class="v nw">43.8</span></div>
    <div class="rs">同じ日本語の文章のトークン数が <b>56.2% 減少</b></div>
  </div>
  <div class="it">
    <div class="hd"><b>🇰🇷 EEVE</b><span>SOLAR の語彙に韓国語トークンを 8,960 個追加</span></div>
    <div class="br"><span class="lb">追加前</span><div class="tr"><div class="bar old" style="width:100%"></div></div><span class="v">31 億</span></div>
    <div class="br"><span class="lb">追加後</span><div class="tr"><div class="bar new" style="width:51.6%"></div></div><span class="v nw">16 億</span></div>
    <div class="rs">同じ韓国語の文章（6.7GB）のトークン数が <b>約半分</b></div>
  </div>
  <div class="src">※ Swallow は追加前を 100 としたときの値（Fujii et al. 2024）、EEVE は Kim et al. 2024 の表 2</div>
</div>
<style>
  .ve { --bg:#f7f8fa; --text:#1f2937; --muted:#6b7280; --card:#ffffff; --line:#dfe3e8; --gray:#c9cfd8; --blue:#2f6fc8; --blue-bg:#e6effb;
        background:var(--bg); color:var(--text); padding:16px; display:flex; flex-direction:column; gap:10px;
        font-family:"Hiragino Sans","Hiragino Kaku Gothic ProN","Noto Sans JP","Yu Gothic",sans-serif; font-size:14px; line-height:1.5; }
  [data-theme="dark"] .ve { --bg:#1e2430; --text:#e5e7eb; --muted:#9aa3b2; --card:#2b3240; --line:#3a4252; --gray:#5b6474; --blue:#8bb8f5; --blue-bg:#24344d; }
  .ve .it { background:var(--card); border:1px solid var(--line); border-radius:10px; padding:10px 12px; display:flex; flex-direction:column; gap:6px; }
  .ve .hd { display:flex; flex-wrap:wrap; align-items:baseline; gap:4px 10px; }
  .ve .hd span { font-size:12px; color:var(--muted); }
  .ve .br { display:grid; grid-template-columns:3.5em 1fr 3.5em; align-items:center; gap:8px; }
  .ve .lb { font-size:12px; color:var(--muted); }
  .ve .tr { height:14px; }
  .ve .bar { height:100%; border-radius:4px; }
  .ve .bar.old { background:var(--gray); }
  .ve .bar.new { background:var(--blue); }
  .ve .v { font-size:13px; font-weight:700; text-align:right; white-space:nowrap; }
  .ve .v.nw { color:var(--blue); }
  .ve .rs { font-size:12px; color:var(--muted); }
  .ve .rs b { color:var(--blue); font-size:14px; white-space:nowrap; }
  .ve .src { font-size:12px; color:var(--muted); }
  @media (max-width: 480px) {
    .ve { padding:12px 10px; gap:8px; }
    .ve .it { padding:8px 10px; }
    .ve .br { grid-template-columns:3.2em 1fr 3.2em; gap:6px; }
  }
</style>
```

つまり、トークンの数はトークナイザーがどんな文章で学習したかで大きく変わります。

## 4. 今のトークナイザーで数えてみる

では今（2026年11月）のトークナイザーではどうなっているでしょう！英語・日本語・韓国語のトークンを実際に数えてみました。

### GPT

同じ意味の文 2,009 個^[FLORES+ というデータを使用。https://huggingface.co/datasets/openlanguagedata/flores_plus]を、GPT のトークナイザー 3 世代で数えました。GPT はモデルによって使うトークナイザーが違い、語彙の大きさもかなり違います。

| モデル | トークナイザー | 語彙の大きさ |
|---|---|---:|
| GPT-2 | gpt2 | 約 5万 |
| GPT-4 | cl100k_base | 約 10万 |
| GPT-4o、GPT-5 | o200k_base | 約 20万 |

トークン測定の結果、英語を 1 としたときの日本語・韓国語のトークン数は次のとおりです^[文ごとに比を出して平均した値。2,009 文 × 3 言語 × 3 トークナイザーで約 18,000 回測定（2026年11月8日時点、tiktoken で確認）]^[GPT-4 の値がセクション 3 の表（韓国語 2.38）と少し違うのは、使ったデータ（FLORES-200 と FLORES+）や集計方法が違うからだと考えられます]。

| | GPT-2 | GPT-4 | GPT-4o/5 |
|---|---:|---:|---:|
| 日本語 | 3.01 | 2.30 | 1.68 |
| 韓国語 | 5.11 | 2.40 | 1.49 |

目立つのは、語彙が約 5万 → 10万 → 20万と大きくなるにつれて、英語との差が小さくなっていることです。それでも、まだ英語の 1.5〜1.7倍のトークンを使っています。

それに GPT-4 までは韓国語のほうがトークン数が多かったのに、GPT-4o/5 では日本語のほうが多くなったのも面白いですね。気になったのでその理由を探してみました。

OpenAI のトークナイザーは語彙が公開されているので、語彙の中に韓国語用・日本語用のトークンがいくつあるかを（私の Claude Code が）数えてみました^[ハングル・かなはその文字を含むトークン、漢字のみは漢字だけでできたトークンの数。漢字のみのトークンは中国語と共有するので、日本語用としては数えていない]。

| | ハングル | かな | 漢字のみ |
|---|---:|---:|---:|
| GPT-2 | 2 | 127 | 42 |
| GPT-4 | 281 | 163 | 839 |
| GPT-4o/5 | 2,360 | 807 | 7,452 |

GPT-4 から GPT-4o/5 で、ハングルのトークンは約 8倍、かなのトークンは約 5倍に増えました。これが韓国語のトークン数がより大きく減ったことと関係しているのかもしれません。

### Claude

GPT と同じく、同じ意味の文 2,009 個を使いました。

ただ Claude はトークナイザーを公開していないので、「Claude 4.7 から新しいトークナイザーになった」という発表^[Anthropic, Token counting. https://platform.claude.com/docs/en/build-with-claude/token-counting]をもとに、それより前と後のモデルに分けて測りました^[文ごとに比を出して平均した値。2,009 文 × 3 言語 × 2 モデルで約 12,000 回測定（2026年11月8日時点、token counting API の claude-sonnet-4-6・claude-opus-5-5 で確認、推定値）]。

| | Sonnet 4.6（以前のトークナイザー） | Opus 5.5（新しいトークナイザー） |
|---|---:|---:|
| 日本語 | 1.89 | 1.29 |
| 韓国語 | 2.42 | 1.66 |

日本語・韓国語と英語の差が大きく縮まりましたね。

実はこれって、日本語・韓国語のトークンが減ったのではなく、英語のトークンが約 45% 増えたことが原因です。

| 2,009 文全体のトークン数 | 以前 | 新しい | 変化 |
|---|---:|---:|---:|
| 英語 | 56,513 | 81,663 | +45% |
| 日本語 | 105,251 | 104,330 | −1% |
| 韓国語 | 134,236 | 133,296 | −1% |

Anthropic のドキュメントには、新しいトークナイザーは同じ文章で約 30% 多くトークンを使う^[正確な増え方は内容による、とも書かれています]と書かれています。でも今回の測定では、英語は 30% より多く増えて、日本語・韓国語はほとんど変わりませんでした。なぜそうなるのかは、トークナイザーが公開されていないのでわかりません。

### 言語別ランキング

ふとトークンを一番多く使う言語はなんだろうと気になったので、GPT-4o/5 のトークナイザーと Claude 4.7 以降のトークナイザーで数えてみました^[英語を除く 235 言語。言語ごとに同じ意味の文 2,009 個（一部の言語は約 1,000 個）全体のトークン数を、英語のトークン数で割った値（FLORES+、2026年11月8日時点）]。ちなみに英語よりトークンを少なく使う言語はありませんでした。

まず GPT-4o/5 のトークナイザーの結果発表です。

**GPT-4o/5：トークンを多く使う言語 上位 10**

| 順位 | 言語 | 英語比 |
|---:|---|---:|
| 1 | チャクマ語 | 17.35 |
| 2 | モンゴル語（モンゴル文字） | 16.24 |
| 3 | サンタル語 | 13.76 |
| 4 | マニプリ語 | 12.28 |
| 5 | ンコ（N'Ko） | 11.29 |
| 6 | タマシェク語 | 10.21 |
| 7 | 標準モロッコ・タマジクト語 | 10.12 |
| 8 | ゾンカ語 | 9.12 |
| 9 | ラーオ語 | 8.24 |
| 10 | チベット語 | 8.15 |

上位 10 はアジア・アフリカの独自の文字を使う言語が多いですね。インターネット上の文章が少なくて、学習用文章にあまり含まれていないからでしょう。

**GPT-4o/5：トークンを少なく使う言語 下位 10**

| 順位 | 言語 | 英語比 |
|---:|---|---:|
| 226 | モーリシャス・クレオール語 | 1.33 |
| 227 | ノルウェー語（ブークモール） | 1.33 |
| 228 | ガリシア語 | 1.33 |
| 229 | スペイン語 | 1.31 |
| 230 | ドイツ語 | 1.31 |
| 231 | マレー語 | 1.30 |
| 232 | 中国語（簡体字） | 1.26 |
| 233 | オランダ語 | 1.24 |
| 234 | インドネシア語 | 1.24 |
| 235 | ポルトガル語 | 1.22 |

下位 10 は、ポルトガル語やスペイン語のように英語と同じラテン文字を使う言語が多い一方で、中国語も入っているのが意外です。GPT-4o/5 の語彙で漢字だけのトークンが大きく増えた影響でしょうか。

続いて Claude 4.7 以降のトークナイザーの結果です。

**Claude 4.7 以降：トークンを多く使う言語 上位 10**

| 順位 | 言語 | 英語比 |
|---:|---|---:|
| 1 | モンゴル語（モンゴル文字） | 11.04 |
| 2 | チャクマ語 | 10.89 |
| 3 | サンタル語 | 9.24 |
| 4 | ンコ（N'Ko） | 8.80 |
| 5 | マニプリ語 | 8.69 |
| 6 | タマシェク語 | 7.95 |
| 7 | 標準モロッコ・タマジクト語 | 7.60 |
| 8 | シャン語 | 6.98 |
| 9 | パンジャーブ語（グルムキー文字） | 6.34 |
| 10 | カンナダ語 | 5.72 |

**Claude 4.7 以降：トークンを少なく使う言語 下位 10**

| 順位 | 言語 | 英語比 |
|---:|---|---:|
| 226 | マケドニア語 | 1.34 |
| 227 | ロシア語 | 1.34 |
| 228 | スウェーデン語 | 1.32 |
| 229 | ブルガリア語 | 1.28 |
| 230 | **日本語** | **1.27** |
| 231 | セルビア語（キリル文字） | 1.26 |
| 232 | 広東語（繁体字） | 1.24 |
| 233 | 呉語（簡体字） | 1.23 |
| 234 | 中国語（簡体字） | 1.15 |
| 235 | 中国語（繁体字） | 1.14 |

驚きの結果です。上位 7 つは GPT とほぼ同じなのに、下位 10 は GPT とかなり違います。

GPT の下位 10 はラテン文字の言語がほとんどでしたが、Claude では中国語系とキリル文字の言語、そして日本語が入りました^[韓国語は 183位、1.62倍でした]。

これもトークナイザーが公開されていないので理由はわかりませんが、意外な結果ですね。普段日本語で Claude Code と会話してる人は、英語との差をあまり気にしなくていいかもです。

## まとめ

ここまでの内容をまとめます。

トークンは文章を細かく分けたかたまりで、その分け方はトークナイザーが学習用文章を見て決めます。そこによく出てくる言葉ほどくっついて、少ないトークンで表せます。

日本語・韓国語が英語よりトークンを多く使うのは、1文字が 3 byte だからというのもありますが、学習用文章に英語が多くて、日本語・韓国語の言葉が英語ほどくっつかないことが大きな理由でしょう。

その差は小さくなってきましたが、今のトークナイザーでも、日本語・韓国語はまだ英語の 1.3〜1.7倍くらいのトークンを使っています。

## 🌷 終わり

ちなみに、同じ言語の中でもタメ口と敬語でトークン数は変わるのかも気になったので数えてみました。

タメ口を 1 としたときの値です^[韓国語は SmileStyle（チャットの文 3,448 組）、日本語は Japanese Honorifics Dataset（115 組、多くは GPT-4o で作られた文）で測定（2026年11月14日時点）]。

| | 文字数 | GPT-4o/5 | Claude 4.7 以降 |
|---|---:|---:|---:|
| 韓国語：丁寧な言い方（です・ますにあたる） | 1.08 | 1.04 | 1.07 |
| 日本語：丁寧語（です・ます） | 1.20 | 1.03 | 1.05 |
| 日本語：尊敬語 | 1.65 | 1.37 | 1.36 |
| 日本語：謙譲語 | 1.55 | 1.35 | 1.30 |

※ データが違うので、韓国語と日本語の値どうしは比べられません

です・ますや韓国語の丁寧な言い方はトークン数がほとんど増えないのに、日本語の尊敬語・謙譲語は 3〜4割増えました。`見る` → `ご覧になる`、`読む` → `拝読する` のように言い方が変わると、くっつかずに分かれる部分が増えるからでしょう。

なので AI と話すときはトークン節約のために尊敬語・謙譲語はやめたほうがいいかもです笑。

:::details 参考文献
- Jurafsky & Martin, *Speech and Language Processing* (3rd ed. draft), Chapter 2. https://web.stanford.edu/~jurafsky/slp3/2.pdf
- Sennrich et al. (2016). Neural Machine Translation of Rare Words with Subword Units. https://arxiv.org/abs/1508.07909
- Radford et al. (2019). Language Models are Unsupervised Multitask Learners (GPT-2). https://cdn.openai.com/better-language-models/language_models_are_unsupervised_multitask_learners.pdf
- Petrov et al. (2023). Language Model Tokenizers Introduce Unfairness Between Languages. https://arxiv.org/abs/2305.15425
- Fujii et al. (2024). Continual Pre-Training for Cross-Lingual LLM Adaptation: Enhancing Japanese Language Capabilities. https://arxiv.org/abs/2404.17790
- Kim et al. (2024). Efficient and Effective Vocabulary Expansion Towards Multilingual Large Language Models. https://arxiv.org/abs/2402.14714
- Anthropic, Token counting. https://platform.claude.com/docs/en/build-with-claude/token-counting
- Anthropic, anthropic-tokenizer-typescript (README). https://github.com/anthropics/anthropic-tokenizer-typescript
- OpenAI, tiktoken. https://github.com/openai/tiktoken
- FLORES+（Open Language Data Initiative, CC BY-SA 4.0）. https://huggingface.co/datasets/openlanguagedata/flores_plus
- SmileStyle（Smilegate AI, CC BY-NC 4.0）. https://github.com/smilegate-ai/korean_smile_style_dataset
- Japanese Honorifics Dataset（Ronan Takizawa, CC BY 4.0）. https://huggingface.co/datasets/ronantakizawa/japanese-honorifics
:::
