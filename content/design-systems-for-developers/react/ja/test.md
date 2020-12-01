---
title: '品質を保つためにテストする'
tocTitle: 'テスト'
description: 'デザインシステムの見た目、機能、アクセシビリティをテストする方法'
commit: 5b71208
---

第 5 章では、UI バグを防ぐためにデザインシステムのテストを自動化します。この章では UI コンポーネントのどの特性がテストを保証するか、また避けるべき落とし穴について深く掘り下げていきます。包括的なカバレージ、簡単なセットアップ、保守容易性のバランスをとるテスト戦略に着手するため、Wave、BBC、Salesforce といったプロフェッショナルのチームを調べました。

<img src="/design-systems-for-developers/ui-component.png" width="250">

## UI コンポーネントテストの基礎

テストを始める前に、何をテストするべきかを洗い出してみましょう。デザインシステムは UI コンポーネントから構成されています。それぞれの UI コンポーネントは与えられた入力 (props) に対応するルックアンドフィールを説明するストーリーを含んでいます。ストーリーはさらにエンドユーザーのブラウザーやデバイスで描画されます。

![コンポーネントの状態が組み合わされる](/design-systems-for-developers/component-test-cases.png)

ご覧のようにあるコンポーネントにはたくさんの状態があります。その状態にデザインシステムのコンポーネントの数を掛けると、なぜそのすべてを追跡していくことが果てしなく無駄な仕事なのかわかりますよね。実際には、手動で全てをレビューしていくことは継続可能ではありません。特にデザインシステムが成長するときには。

**今**自動テストをセットアップする最大の理由は**未来の**作業を楽にするためなのです。

## テストの用意

Storybook のワークフローに関する[以前の記事](https://blog.hichroma.com/the-delightful-storybook-workflow-b322b76fd07)を 4 つのフロントエンドチームに感想を聞きました。そして、ストーリーを書いて、テストを簡単でわかりやすくするための以下のベストプラクティスに同意を得ました。

**サポートするコンポーネントの状態を明確にする**: ストーリーがどういった入力の組み合わせでその状態を作り出すのかを明確にする。サポートされていない状態を容赦なく省略して、ノイズを減らします。

**コンポーネントの描画を一貫させる**: ランダム性 (Math.random の使用) や相対性 (Date.now の使用) のある入力により引き起こされる変動を緩和します。

> 「よいストーリーはコンポーネントが野生に出て経験する状態を全て視覚化することが出来ます。」 – Tim Hingston, Tech lead at Apollo GraphQL

## 見た目をビジュアルテストする

デザインシステムにはプレゼンテーショナルコンポーネントが含まれており、プレゼンテーショナルコンポーネントは本質的に視覚的です。ビジュアルテストでは視覚的な視点から描画された UI を検証します。

ビジュアルテストでは全ての UI コンポーネントの画像を一貫したブラウザーの環境で取得します。新しいスクリーンショットは自動的に前回承認された基準となるスクリーンショットと比較され、もし視覚的な差異があれば、通知されます。

![コンポーネントをビジュアルテストする](/design-systems-for-developers/component-visual-testing.gif)

もしモダンな UI を作っているのなら、ビジュアルテストでフロントエンドチームの時間がかかるレビューが節約でき、大規模な UI のリグレッションを予防できます。

<a href="https://www.learnstorybook.com/design-systems-for-developers/react/ja/review/#publish-storybook">前の章</a>では、Storybook を [Chromatic](https://www.chromatic.com/) を使用して発行する方法を学びました。その際、`Button` コンポーネントに赤い太線を入れて、チームメートにレビューを依頼しました。

![ボタンの赤線](/design-systems-for-developers/chromatic-button-border-change.png)

それでは、Chromatic の組み込み[テストツール](https://www.chromatic.com/features/test)を使用して、ビジュアルテストの使用方法を見ていきましょう。プルリクエストが作成されたとき、Chromatic は変更されたイメージをキャプチャーし、前回の同じコンポーネントのイメージと比較し、3 つの変更点が見つかりました。

![プルリクエストのチェックの一覧](/design-systems-for-developers/chromatic-list-of-checks.png)

レビューするため、**🟡UI Tests** チェックをクリックしてください。

![Chromatic 上の 2 回目のビルド (変更点あり)](/design-systems-for-developers/chromatic-second-build-from-pr.png)

変更が意図的 (改善) なのか意図的でない (バグ) のかをレビューで確認します。もし変更を受け入れるのであれば、テストの基準が更新されます。つまり、今後のコミットではその新しい基準イメージと比較することでバグを見つけます。

![Chromatic で変更点をレビューする](/design-systems-for-developers/chromatic-review-changes-pr.png)

前回の章では、チームメートはなぜか `Button` の赤い枠を気に入りませんでした。変更点を戻すよう示すために、変更を拒否します。

![Chromatic で変更点を拒否する](/design-systems-for-developers/chromatic-review-deny.png)

変更を取り消して、再度コミットすることで、ビジュアルテストをパスします。

## 単体テストの役割

単体テストでは
Unit tests verify whether the UI code returns the correct output given a controlled input. They live alongside the component and help you validate specific functionality.

Everything is a component in modern view layers like React, Vue, and Angular. Components encapsulate diverse functionality from modest buttons to elaborate date pickers. The more intricate a component, the trickier it becomes to capture nuances using visual testing alone. That’s why we need unit tests.

![Unit test components](/design-systems-for-developers/component-unit-testing.gif)

For instance, our Link component is a little complicated when combined with systems that generate link URLs (“LinkWrappers” in ReactRouter, Gatsby, or Next.js). A mistake in the implementation can lead to links without a valid href value.

Visually, it isn’t possible to see if the `href` attribute is there and points to the right location, which is why a unit test can be appropriate to avoid regressions.

#### Unit testing hrefs

Let’s add a unit test for our `Link` component. create-react-app has set up a unit test environment for us already, so we can simply create a file `src/Link.test.js`:

```javascript
//src/Link.test.js

import React from 'react';
import ReactDOM from 'react-dom';
import { Link } from './Link';

// A straightforward link wrapper that renders an <a> with the passed props. What we are testing
// here is that the Link component passes the right props to the wrapper and itself.
const LinkWrapper = props => <a {...props} />; // eslint-disable-line jsx-a11y/anchor-has-content

it('has a href attribute when rendering with linkWrapper', () => {
  const div = document.createElement('div');
  ReactDOM.render(
    <Link href="https://learnstorybook.com" LinkWrapper={LinkWrapper}>
      Link Text
    </Link>,
    div
  );

  expect(div.querySelector('a[href="https://learnstorybook.com"]')).not.toBeNull();
  expect(div.textContent).toEqual('Link Text');

  ReactDOM.unmountComponentAtNode(div);
});
```

We can run the above unit test as part of our `yarn test` command.

![Running a single Jest test](/design-systems-for-developers/jest-test.png)

Earlier we configured our GitHub Action to deploy Storybook, we can now adjust it to include testing as well. Our contributors will now benefit from this unit test. The Link component will be robust to regressions.

```yaml
# .github/workflows/chromatic.yml

# ... same as before
jobs:
  test:
    # the operating system it will run on
    runs-on: ubuntu-latest
    # the list of steps that the action will go through
    steps:
      - uses: actions/checkout@v1
      - run: yarn
      - run: yarn test # adds the test command
      - uses: chromaui/action@v1
        # options required to the GitHub chromatic action
        with:
          # our project token, to see how to obtain it
          # refer to https://www.learnstorybook.com/intro-to-storybook/react/en/deploy/ (update link)
          projectToken: project-token
          token: ${{ secrets.GITHUB_TOKEN }}
```

![Successful circle build](/design-systems-for-developers/gh-action-with-test-successful-build.png)

<div class="aside"> Note: Watch out for too many unit tests which can make updates cumbersome. We recommend unit testing design systems in moderation.</div>

> "Our enhanced automated test suite has empowered our design systems team to move faster with more confidence." – Dan Green-Leipciger, Senior software engineer at Wave

## Accessibility test

“Accessibility means all people, including those with disabilities, can understand, navigate, and interact with your app... Online [examples include] alternative ways to access content such as using the tab key and a screen reader to traverse a site.” writes developer [Alex Wilson from T.Rowe Price](https://medium.com/storybookjs/instant-accessibility-qa-linting-in-storybook-4a474b0f5347).

Disabilities affect 15% of the population according to the [World Health Organization](https://www.who.int/disabilities/world_report/2011/report/en/). Design systems have an outsized impact on accessibility because they contain the building blocks of user interfaces. Improving accessibility of just one component means every instance of that component across your company benefits.

![Storybook accessibility addon](/design-systems-for-developers/storybook-accessibility-addon.png)

Get a headstart on inclusive UI with Storybook’s Accessibility addon, a tool for verifying web accessibility standards (WCAG) in realtime.

```shell
yarn add --dev @storybook/addon-a11y

```

Add the addon in `.storybook/main.js`:

```javascript
// .storybook/main.js

module.exports = {
  stories: ['../src/**/*.stories.mdx', '../src/**/*.stories.@(js|jsx|ts|tsx)'],
  addons: [
    '@storybook/addon-links',
    '@storybook/addon-essentials',
    '@storybook/preset-create-react-app',
    '@storybook/addon-a11y',
  ],
};
```

Update your `.storybook/preview.js`'s [parameters](https://storybook.js.org/docs/react/writing-stories/parameters) and add the following `a11y` configuration:

```javascript
//.storybook/preview.js

import React from 'react';

import { GlobalStyle } from '../src/shared/global';

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
  // Storybook a11y addon configuration
  a11y: {
    // the target DOM element
    element: '#root',
    // sets the execution mode for the addon
    manual: false,
  },
};
```

Once all is setup, you’ll see a new “Accessibility” tab in the Storybook addons panel.

![Storybook a11y addon](/design-systems-for-developers/storybook-addon-a11y-6-0.png)

This shows you accessibility levels of DOM elements (violations and passes). Click the “highlight results” checkbox to visualize violations in situ with the UI component.

<video autoPlay muted playsInline loop>
  <source
    src="/design-systems-for-developers/storybook-addon-a11y-6-0-highlighted.mp4"
    type="video/mp4"
  />
</video>

From here, follow the addon’s accessibility recommendations.

## Other testing strategies

Paradoxically, tests can save time but also bog down development velocity with maintenance. Be judicious about testing the right things – not everything. Even though software development has many test strategies, we discovered the hard way that some aren’t suited for design systems.

#### Snapshot tests (Jest)

This technique captures the code output of UI components and compares it to previous versions. Testing UI component markup ends up testing implementation details (code), not what the user experiences in the browser.

Diffing code snapshots is unpredictable and prone to false positives. At the component level, code snapshotting doesn’t account for global changes like design tokens, CSS, and 3rd party API updates (web fonts, Stripe forms, Google Maps, etc.). In practice, developers resort to “approving all” or ignoring snapshot tests altogether.

> Most component snapshot tests are really just a worse version of screenshot tests. Test your outputs. Snapshot what gets rendered, not the underlying (volatile!) markup. – Mark Dalgliesh, Frontend infrastructure at SEEK, CSS modules creator

#### End-to-end tests (Selenium, Cypress)

End-to-end tests traverse the component DOM to simulate the user flow. They’re best suited for verifying app flows like the signup or checkout process. The more complex functionality the more useful this testing strategy.

Design systems contain atomic components with relatively simple functionality. Validating user flows are often overkill for this task because the tests are time-consuming to create and brittle to maintain. However, in rare situations, components may benefit from end-to-end tests. For instance, validating complex UIs like datepickers or self-contained payment forms.

## Drive adoption with documentation

A design system is not complete with tests alone. Since design systems serve stakeholders from across the organization, we need to teach others how to get the most from our well-tested UI components.

In chapter 6, we’ll learn how to accelerate design system adoption with documentation. See why Storybook Docs is a secret weapon to create comprehensive docs with less work.
