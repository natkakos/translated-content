---
title: '<bdo>: 双方向文字列上書き要素'
slug: Web/HTML/Element/bdo
tags:
  - BiDi
  - 双方向テキスト
  - 要素
  - HTML
  - HTML 文字レベルの意味付け
  - HTML:フローコンテンツ
  - HTML:知覚可能コンテンツ
  - HTML:記述コンテンツ
  - 左書き
  - リファレンス
  - テキスト
  - 右書き
  - 書字方向
  - テキスト描画
  - Web
  - ltr
  - rtl
translation_of: Web/HTML/Element/bdo
browser-compat: html.elements.bdo
---
{{HTMLRef}}

**`<bdo>`** は [HTML](/ja/docs/Web/HTML) の要素で、現在のテキストの書字方向を上書きし、中のテキストが異なる書字方向で描画されるようにします。

{{EmbedInteractiveExample("pages/tabbed/bdo.html", "tabbed-standard")}}

テキストの文字は指定された方向の開始位置から描画されます。それぞれの文字の向きには影響を与えません (ですから、例えば、文字は裏返しにはなりません)。

<table class="properties">
  <tbody>
    <tr>
      <th scope="row">
        <a href="/ja/docs/Web/Guide/HTML/Content_categories"
          >コンテンツカテゴリー</a
        >
      </th>
      <td>
        <a href="/ja/docs/Web/Guide/HTML/Content_categories#フローコンテンツ"
          >フローコンテンツ</a
        >,
        <a href="/ja/docs/Web/Guide/HTML/Content_categories#記述コンテンツ"
          >記述コンテンツ</a
        >, 知覚可能コンテンツ
      </td>
    </tr>
    <tr>
      <th scope="row">許可されている内容</th>
      <td>
        <a href="/ja/docs/Web/Guide/HTML/Content_categories#記述コンテンツ"
          >記述コンテンツ</a
        >
      </td>
    </tr>
    <tr>
      <th scope="row">タグの省略</th>
      <td>{{no_tag_omission}}</td>
    </tr>
    <tr>
      <th scope="row">許可されている親要素</th>
      <td>
        <a href="/ja/docs/Web/Guide/HTML/Content_categories#記述コンテンツ"
          >記述コンテンツ</a
        >を受け入れるすべての要素
      </td>
    </tr>
    <tr>
      <th scope="row">暗黙の ARIA ロール</th>
      <td>
        <a href="https://www.w3.org/TR/html-aria/#dfn-no-corresponding-role"
          >対応するロールなし</a
        >
      </td>
    </tr>
    <tr>
      <th scope="row">許可されている ARIA ロール</th>
      <td>すべて</td>
    </tr>
    <tr>
      <th scope="row">DOM インターフェイス</th>
      <td>
        {{domxref("HTMLElement")}}。Gecko 1.9.2 (Firefox 4)
        以前では、Firefox はこの要素に対し
        <code
          ><a href="/ja/docs/Web/API/HTMLSpanElement">HTMLSpanElement</a></code
        >
        インターフェイスを実装しています。
      </td>
    </tr>
  </tbody>
</table>

## 属性

この要素には[グローバル属性](/ja/docs/Web/HTML/Global_attributes)があります。

- {{htmlattrdef("dir")}}

  - : この要素の内容において、テキストが描画される方向です。以下の値が指定可能です。

    - `ltr`: テキストを左から右へ (left-to-right) 向かわせることを意味する指定。
    - `rtl`: テキストを右から左へ (right-to-left) 向かわせることを意味する指定。

## 例

```html
<!-- 書字方向を切り替える -->
<p>This text will go left to right.</p>
<p><bdo dir="rtl">This text will go right
to left.</bdo></p>
```

### 結果

{{EmbedLiveSample('Examples')}}

## メモ

HTML 4 の仕様では、この要素にイベントが指定されていません。イベントは XHTML で追加されました。これは、恐らく見落としでしょう。

## 仕様書

{{Specifications}}

## ブラウザーの互換性

{{Compat}}

## 関連情報

- 関連する HTML 要素: {{HTMLElement("bdi")}}