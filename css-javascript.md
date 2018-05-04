---
layout: page
title: CSS / JavaScript
---

## concrete5 の編集モードとウェブページの共存

concrete5 では、ウェブページを見たまま編集することができます。技術的には、concrete5 のインターフェースと、ウェブページそのものが、同じ HTML の中に両方出力されているという状態です。普段 CSS や JavaScript を書いている方であれば、そりゃマズイ、とピンとくるかもしれません。何の対策もしなければ、お互いがお互いに影響を与えてしまい、concrete5 のインターフェースが使えなくなったり、ウェブページが崩れてしまったりしてしまいます。

干渉を防ぐためには、concrete5 が行なっている干渉を防ぐための工夫を知り、あなたが作成するテーマで使用する CSS や JavaScript のコーディングで特定の記述を避ける必要があります。

### ウェブページは全て div.ccm-page の中へ

「同じ HTML の中に concrete5 のインターフェースとウェブページそのものが同時に出力される」問題に対して、concrete5 が採用した解決策はシンプルです。

* ウェブページは全て `<div class="ccm-page"></div>` の中に入れること。
* concrete5 のインターフェースは全て `<div class="ccm-ui"></div>` の中に入れること。

このシンプルなルールを守れば、お互いのCSSが影響し合うことを避けられます。  
concrete5 サイトのために HTML コーディングを行う際には、必ずこのルールを守ってください。

悪い例：このまま concrete5 のテーマにしてしまうと concrete5 のインターフェースと干渉します。

```html
<body>
  <h1>タイトル</h1>
</body>
```

```css
h1 { font-size: 18px; }
```

良い例：concrete5 のインターフェースと分けられているので良い例です。

```html
<body>
  <div class="ccm-page">
    <h1>タイトル</h1>
  </div>
</body>
```

```css
.ccm-page h1 { font-size: 18px; }
```

### concrete5 関連の JavaScript は命名規則で分離

concrete5 の直感的なインターフェースの挙動は、JavaScript によって実現されています。concrete5 が定義する関数は `Concrete` や `ccm_` が先頭に付き、jQuery プラグインは `concrete` が先頭に付きます。

例： `ConcreteToolbar`, `ccm_addError`, `$.concreteMenu()`

concrete5 サイトのために JavaScript を記述する際は、`concrete` や `ccm` という単語を使わないように気をつけてください。

## concrete5 の編集モードを破壊する避けるべき記述

ウェブページの HTML を全て `div.ccm-page` の中に収めることでほとんどの問題は回避できますが、いくつかそのほかにも気をつけるべき点があります。

### CSS で避けるべき記述

#### z-index, position

concrete5 の編集モードで、クリックするとメニューが出現する領域や、ブロックをドラッグする際につかむための領域は、`position: absolute;` で配置されています。

![メニュークリックプロキシ](https://raw.githubusercontent.com/concrete5cojp/Best-Practices-concrete5-Template-Development/master/images/click-proxy.png)

そのため、ウェブページ内の要素でも同様に CSS で `position` を利用して配置を行なっていると、concrete5 のこれらのインターフェースより上の階層にかぶさってしまい、ブロックの編集や移動ができなくなる場合があります。それを避けるために、ウェブページ内の要素では、CSS の `z-index` の値で 500 以上の値を使ってはいけません。また、`z-index` の値が 500 未満であっても、スタック文脈を変更することで、やはり concrete5 のインターフェースより上にかぶさってしまう現象が起こります。

* [ブロックをドラッグするハンドルが掴めなくなる現象のデモ](https://codepen.io/hissy-github/pen/XEBYJB)
* [スタック文脈 - ウェブデベロッパーガイド | MDN](https://developer.mozilla.org/ja/docs/Web/Guide/CSS/Understanding_z_index/The_stacking_context)

もっとも確実な回避方法は、concrete5 で編集したい領域では、いっさい `position` や `z-index` を使わないことです。しかし、デザインによってはこの制限は不可能な場合もありますので、その際は[編集モードかどうかを判定し](https://concrete5-japan.org/help/5-7/developer/working-with-pages/getting-data-about-a-page/)、編集モード中は `position` や `z-index` を打ち消す CSS を追加で読み込ませることになるでしょう。ただし、CSS の打ち消しは必ずしも常にうまくいくわけではなく、場合によっては CSS の大幅な修正が発生することになります。

### JavaScript で避けるべき記述

JavaScript では、幸いなことに CSS ほどの制限はありませんが、外部ライブラリには気をつける必要があります。特に気をつけるべきは **jQuery** です。

#### concrete5 に同梱されている jQuery 以外の jQuery に依存する

[jQuery](http://jquery.com/) は非常に普及している JavaScript ライブラリで、多くのサイトやウェブアプリケーションで使われていますが、concrete5 も例外ではありません。実際、concrete5 の編集モードは jQuery に多くを依存しているため、jQuery の読み込みをやめると言うことはできません。そのため、ウェブページで使用するための JavaScript ファイルの方が譲歩する必要があります。まず、jQuery のバージョンは、[concrete5 に同梱されているもの](https://github.com/concrete5/concrete5/blob/develop/concrete/js/jquery.js)と同じにする必要があります。jQuery は現在バージョン3.xがリリースされていますが、concrete5 に同梱されているのはバージョン1.xです。

また、jQuery のバージョンを合わせているのに concrete5 が動かなくなると言うトラブルも起こりがちですが、この場合は concrete5 に同梱されている jQuery とは別に、重複して jQuery を読み込ませていることが原因です。concrete5 はその動作のために独自の jQuery プラグインを実装していますが、重複して jQuery を読み込ませることで、それらのプラグインが全てリセットされてしまうのです。

#### concrete5 に同梱されている Bootstrap 以外の Bootstrap に依存する

[Bootstrap](https://getbootstrap.com/) もレスポンシブデザインのウェブサイトではよく使われるライブラリです。concrete5 は[バージョン3.xの Bootstrap のライブラリのいくつかを使用しています](https://github.com/concrete5/concrete5/tree/develop/concrete/js/build/vendor/bootstrap)。Bootstrap は現在バージョン4.xがリリースされていますが、jQuery と同様に concrete5 に同梱されているバージョンに合わせる必要があります。

#### その他のライブラリ

そのほかにも concrete5 では様々な JavaScript ライブラリを利用していますが、編集モードでは使われないものや、利用される頻度の低いものが多いため、あまり問題にはなりません。気になる方はドキュメントの[アセットリスト](https://concrete5-japan.org/help/5-7/developer/appendix/asset-list/)をご確認ください。

## concrete5 が出力するコードを知ろう

### ページラッパークラスを活用しよう

### デザイン機能でできること、起こること
