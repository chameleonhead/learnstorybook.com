---
title: 'UI コンポーネントをビルドする'
tocTitle: 'ビルド'
description: 'デザインシステムのコンポーネントをビルドしカタログ化するために Storybook をセットアップする'
commit: e7b6f00
---

第 3 章では、人気の高いコンポーネントのビューアーである、Storybook をはじめとして、デザインシステムに不可欠なツールをセットアップしていきます。このガイドの目的はプロフェッショナルチームがどのようにデザインシステムをビルドするかを示すことにありますので、コードの衛生管理や、Storybook の手軽なアドオン、ディレクトリー構成といった詳細についても注目していきましょう。

![Storybook の場所](/design-systems-for-developers/design-system-framework-storybook.jpg)

## フォーマットしてチェックしてコードの衛生を管理する

デザインシステムは共同作業です。そのため、構文を修正し標準的なフォーマットに整形するツールが、品質を向上に役立ちます。ツールによってコードの一貫性を保つ方が、手作業でポリシーを守るよりも労力がかかりませんので、デザインシステムの作者にとってメリットがあります。

このチュートリアルでは VSCode をエディターとして使用しますが、同じことは Atom、Sublime、InttelliJ といったモダンなエディターでもできるでしょう。

Prettier をプロジェクトに導入し、エディターを正しく設定すれば、それほど難しくなく、一貫性のあるフォーマットを実現できます:

```shell
yarn add --dev prettier
```

初めて Prettier を使用する場合、エディターに設定が必要です。VSCode では Prettier アドオンをインストールします:

![VSCode の Prettier アドオン](/design-systems-for-developers/prettier-addon.png)

保存時にフォーマットする (`editor.formatOnSave`) オプションを有効にしてください。Prettier をインストールすれば、ファイルを保存するたびに自動でフォーマットされます。

## Storybook をインストールする

Storybook は業界標準のコンポーネントを独立した環境で開発するための[コンポーネントエクスプローラー](https://blog.hichroma.com/the-crucial-tool-for-modern-frontend-engineers-fb849b06187a)です。デザインシステムは UI コンポーネントにフォーカスしているので、Storybook は理想的なツールです。Storybook には以下の機能があります:

- 📕UI コンポーネントをカタログ化する
- 📄 コンポーネントの多様性をストーリーとして保存する
- ⚡️Hot Module Reloading を代表とする開発エクスペリエンス
- 🛠React など、多くのビュー層フレームワーク・ライブラリーへの対応

それでは Storybook をインストールして実行します。

```shell
npx -p @storybook/cli sb init
yarn storybook
```

次のようになれば成功です:

![最初の Storybook の UI](/design-systems-for-developers/storybook-initial-6-0.png)

コンポーネントエクスプローラーのセットアップが完了しました！

Storybook をアプリケーションにインストールする度に `stories` フォルダーにサンプルが追加されます。時間があれば見てみるのもいいでしょう。けれど、これから作るデザインシステムには不要なので `stories` フォルダーは削除しても問題ありません。

これで Storybook は以下の次のようになっていることでしょう (フォントのスタイルが少しずれていることに注目してください。「Avatar: Initials」ストーリーがその例です):

<video autoPlay muted playsInline loop>
  <source
    src="/design-systems-for-developers/storybook-initial-stories-without-styles-6-0.mp4"
    type="video/mp4"
  />
</video>

#### グローバルなスタイルを追加する

このデザインシステムにはコンポーネントのドキュメントを正しく描画するためにグローバルなスタイル (CSS リセット) を適用する必要があります。それには styled-components の `GlobalStyle` タグを使用することで簡単に追加できます。`src/shared/global.js` にあるグローバルなスタイルを以下のように変更してください:

```javascript
// src/shared/global.js

import { createGlobalStyle, css } from 'styled-components';
import { color, typography } from './styles';

export const fontUrl = 'https://fonts.googleapis.com/css?family=Nunito+Sans:400,700,800,900';

export const bodyStyles = css`
  /* same as before */
`;

export const GlobalStyle = createGlobalStyle`
 body {
   ${bodyStyles}
 }
`;
```

`GlobalStyle` 「コンポーネント」を Storybook で使用するには、[デコレーター (decorator)](https://storybook.js.org/docs/react/writing-stories/decorators) (コンポーネントのラッパー) を使います。アプリケーションではトップレベルのレイアウトに配置しますが、Storybook ではプレビュー設定ファイル [`.storybook/preview.js`](https://storybook.js.org/docs/react/configure/overview#configure-story-rendering) を使用し、全てのストーリーをグローバルなスタイルを適用するデコレーターでラップします。

```javascript
// .storybook/preview.js

import React from 'react';

import { GlobalStyle } from '../src/shared/global';

// Global decorator to apply the styles to all stories
export const decorators = [
  Story => (
    <>
      <GlobalStyle />
      <Story />
    </>
  ),
];

export const parameters = {
  actions: { argTypesRegex: '^on[A-Z].*' },
};
```

上記のデコレーターにより `GlobalStyle` がどのストーリーを選択した場合でも描画されるようになります。

<div class="aside">デコレーターの <code><></code> はミスタイプではありません。これは <a href="https://reactjs.org/docs/fragments.html">React Fragment</a> で、不要な HTML タグが出力されるのを避けるために使用しています。</div>

#### font タグを追加する

このデザインシステムでは Nuito Sans というフォントにも依存しています。フォントをアプリケーションに導入する方法はフレームワークによって異なります (詳細については[こちら](https://github.com/storybookjs/design-system#font-loading)を参照してください) が、Storybook で最も簡単な方法は [`.storybook/preview-head.html`](https://storybook.js.org/docs/react/configure/story-rendering#adding-to-head) を使用してページの `<head>` に直接 `<link>` タグを追加する方法です。

```html
<link rel="stylesheet" href="https://fonts.googleapis.com/css?family=Nunito+Sans:400,700,800,900" />
```

これで Storybook は以下のようになるでしょう。グローバルなフォントスタイルを追加したので「T」がサンセリフで表示されていることに注目してください。

![グローバルなスタイルが読み込まれた Storybook](/design-systems-for-developers/storybook-global-styles-6-0.png)

## Storybook をアドオンで加速させる

Storybook にはコミュニティによって作成された強力な[アドオンのエコシステム](https://storybook.js.org/addons)があります。実際の開発者にとって、自分たちで (とても時間がかかるかもしれない) カスタムツールを作成するよりもこのエコシステムを使用してワークフローを作成する方が簡単です。

<h4>アクションアドオンを用いて作用を検証する</h4>

[アクションアドオン](https://storybook.js.org/docs/react/essentials/actions) は Button や Link といった対話的な要素に対してアクションが実行された際に Storybook 上に UI フィードバックを提供します。アクションアドオンは Storybook にデフォルトでインストールされているので、コンポーネントのコールバック用の prop に「action」を渡すだけで使用できます。

アクションアドオンが Button 要素でどのように使われているか見てみましょう。Button 要素ではクリックに反応するラッパーコンポーネントをオプションとして引き受けます。そのラッパーにアクションを渡しているストーリーです:

```javascript
// src/Button.stories.js

import React from 'react';
import styled from 'styled-components';

// When the user clicks a button, it will trigger the `action()`,
// ultimately showing up in Storybook's addon panel.
function ButtonWrapper(props) {
  return <CustomButton {...props} />;
}

export const buttonWrapper = (args) => (
  return <CustomButton {...props}/>;
// … etc ..
)
```

<video autoPlay muted playsInline loop>
  <source
    src="/design-systems-for-developers/storybook-addon-actions-6-0.mp4"
    type="video/mp4"
  />
</video>

<h4>コンポーネントの負荷テストをするためのコントロールアドオン</h4>

Storybook を初めてインストールすると[コントロールアドオン](https://storybook.js.org/docs/react/essentials/controls)が含まれており、すぐに使えるように設定済みです。

コントロールアドオンを使用することで、動的にコンポーネントへの入力 (props) を変更することができるようになります。[Args](https://storybook.js.org/docs/react/writing-stories/args) を使用することでコンポーネントの props に複数の値を UI を使って提供することができます。また、デザインシステムの使用者が導入する前にコンポーネントを試すことができ、各入力 (props) がどのようにコンポーネントに影響するかが理解できます。

`src/Avatar.stories.js` の `Avatar` コンポーネントに新しいストーリーを追加して、どのように機能するか見てみましょう:

```javascript
// src/Avatar.stories.js

import React from 'react';

// …

// New story using controls
const Template = args => <Avatar {...args} />;

export const Controls = Template.bind({});
Controls.args = {
  loading: false,
  size: 'tiny',
  username: 'Dominic Nguyen',
  src: 'https://avatars2.githubusercontent.com/u/263385',
};
```

アドオンパネルの Controls タブに注目してください。コントロールアドオンは props を調整できるように画面上に UI を自動的に生成します。例えば、「size」の select 要素では `tiny`、`small`、`medium`、`large` といった Avatar で使用可能なサイズを選んでいくことができます。コンポーネントの残りの props (「loading」、「username」、「src」) も同様です。この機能により、コンポーネントに対する負荷テストをお手軽に実施できます。

<video autoPlay muted playsInline loop>
  <source
    src="/design-systems-for-developers/storybook-addon-controls-6-0.mp4"
    type="video/mp4"
  />
</video>

とはいえ、コントロールアドオンがストーリーを置き換えることにはなりません。コントロールアドオンはコンポーネントのエッジケースを見つけるのには最高であり、ストーリーはそういった条件における見本として使用します。

アドオンについては、アクセシビリティアドオンとドキュメントアドオンを後の章で見ることにしましょう。

> 「Storybook は強力なフロントエンドの作業環境を提供するツールで、チームがビジネスロジックや配管につまづくことなく UI コンポーネントを (フルスクリーンで！) デザインし、製造し、組み合わせられるようにしてくれます。」 – Atomic Design の著者 Brad Frost

## 自動的にメンテナンスする方法を学ぶ

これでデザインシステムが Storybook にできたので、業界標準のデザインシステムを作るもう一歩を踏み出しました。リモートリポジトリ―にコミットする良いタイミングでしょう。そうすれば、メンテナンスを継続的に合理化していく自動ツールのセットアップ方法を考えていくことができます。

デザインシステムは、他のソフトウェアと同様に、進化していかなければなりません。デザインシステムが成長しても、UI コンポーネントのルックアンドフィールが意図通りであることを確実にすることが挑戦です。

第 4 章では、継続的インテグレーションのセットアップする方法と、デザインシステムを協業のために自動的に公開する方法学びます。
