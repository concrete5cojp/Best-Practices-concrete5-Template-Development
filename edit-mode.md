---
layout: page
title: 編集モードとの共存
---

## concrete5 がウェブページと共存するために行なっている対策

concrete5 では、ウェブページを見たまま編集することができます。技術的には、concrete5 のインターフェースと、ウェブページそのものが、同じ HTML の中に両方出力されているという状態です。普段 CSS や JavaScript を書いている方であれば、そりゃマズイ、とピンとくるかもしれません。何の対策もしなければ、お互いがお互いに影響を与えてしまい、concrete5 のインターフェースが使えなくなったり、ウェブページが崩れてしまったりしてしまいます。もちろん、concrete5 はそのような干渉を防ぐための工夫があります。

### ウェブページは全て div.ccm-page の中へ

「同じ HTML の中に concrete5 のインターフェースとウェブページそのものが同時に出力される」問題に対して、concrete5 が採用した解決策はシンプルです。

* ウェブページは全て `<div class="ccm-page"></div>` の中に入れること。
* concrete5 のインターフェースは全て `<div class="ccm-ui"></div>` の中に入れること。

このシンプルなルールを守れば、お互いのCSSが影響し合うことを避けられます。  
concrete5 サイトのために HTML コーディングを行う際には、必ずこのルールを守ってください。

悪い例：

```css
h1 { font-size: 18px; }
```

良い例：

```css
.ccm-page h1 { font-size: 18px; }
```

### concrete5 関連の JavaScript は命名規則で分離

concrete5 の直感的なインターフェースの挙動は、JavaScript によって実現されています。concrete5 が定義する関数は `Concrete` や `ccm_` が先頭に付き、jQuery プラグインは `concrete` が先頭に付きます。

例： `ConcreteToolbar`, `ccm_addError`, `$.concreteMenu()`

concrete5 サイトのために JavaScript を記述する際は、`concrete` や `ccm` という単語を使わないように気をつけてください。

## concrete5 の編集モードを破壊する避けるべき記述

### CSS で避けるべき記述

### JavaScript で避けるべき記述

## concrete5 が出力するコードを知ろう

### ページラッパークラスを活用しよう

### デザイン機能でできること、起こること
