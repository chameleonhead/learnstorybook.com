---
title: 'チーム内でレビューする'
tocTitle: 'レビュー'
description: '継続的インテグレーションと視覚的なレビューで協業する'
commit: eabed3d
---

第 4 章では、不整合を解消しつつデザインシステムを改善するプロフェッショナルのワークフローを学びます。この章では UI へのフィードバックをまとめ、チーム内でコンセンサスを得るためのテクニックを紹介します。ここで紹介するプロセスは Auth0 や、Spotify、Discovery Network などで使用されています。

## 信頼できる唯一の情報源か単一障害点か

前にデザインシステムがフロントエンドチームの[単一障害点](https://blog.hichroma.com/why-design-systems-are-a-single-point-of-failure-ec9d30c107c2)になると書きました。本質的にデザインシステムは依存先になります。デザインシステムでコンポーネントを変更すると、その変更は依存しているアプリケーションに影響します。変更を展開するメカニズムは隔たりなく、改善とバグのどちらも配布してしまいます。

![デザインシステムの依存性](/design-systems-for-developers/design-system-dependencies.png)

バグはデザインシステムにとって実存的なリスクであり、バグを防ぐためにできる限りのことをしなければなりません。ちょっとした変更が雪だるま式に無数のリグレッションへとつながります。継続的な保守戦略がなければ、デザインシステムは衰退してしまいます。

> 「私の環境では動いたのに?!」 – みんな

## チーム内で UI コンポーネントを視覚的にレビューする

視覚的なレビューは UI の振る舞いと美しさを確認するプロセスです。UI を開発する際、チーム内で QA (品質保証) をする際のどちらでも発生します。

ほとんどの開発者はコードレビューに慣れています。コードレビューはコードの品質を上げるために他の開発者からフィードバックを得るプロセスです。UI コンポーネントはコードをグラフィカルに表現するので、視覚的なレビューは UI/UX のフィードバックを集めるために不可欠です。

### 誰でも見られる参照先を確立する

node_modules を削除してください。パッケージを再インストールしてください。localstorage を消してください。cookie を消してください。こういった表現をよく聞くのなら、チームメンバーに最新のコードを参照させるのがどれだけ大変かわかりますよね。全く同じ開発環境をが整備されていないとき、開発環境の問題と実際のバグを見分けるのは悪夢のようです。

幸いにも、フロンエンド開発者には共通のコンパイル先があります。ブラウザーです。理解しているチームは Storybook を視覚的なレビューのために誰でも見られる参照先として公開します。これによりローカル開発環境に固有の複雑さを避けることができます。(テクニカルサポートになるのは面倒ですし。)

![クラウド上で成果をレビューする](/design-systems-for-developers/design-system-visual-review.jpg)

URL で生の UI コンポーネントを参照できるようになれば、利害関係者たちは自分たちの快適なブラウザーで UI のルックアンドフィールを確認できます。これで開発者もデザイナーも PM もローカル開発環境で大騒ぎをしなくて済みますし、スクリーンショットをあちこち飛ばさなくて済みますし、最新ではない UI を参照することもありません。

> 「Storybook を PR 毎にデプロイすることが視覚的なレビューを簡単にし、プロダクトオーナーがコンポーネント単位で考えられるようにしてくれる」– Storybook のメンテナー Norbert de Langen

<h2 id="publish-storybook">Storybook を公開する</h2>

それでは視覚的なレビューのワークフローを [Chromatic](https://www.chromatic.com/) を使用して実演してみましょう。Chromatic は Storybook 　のメンテナーが作成した無料のホスティングサービスで、Storybook をクラウド上でセキュアにデプロイして、ホストしてくれます。[Storybook を静的サイトにビルドしてデプロイする](https://storybook.js.org/docs/basics/exporting-storybook/)のはとても簡単なので他のホスティングサービスを使用することも可能です。

### Chromatic を手に入れる

まずは、[chromatic.com](https://chromatic.com) へアクセスし、GitHub のアカウントを使用してサインアップします。

![Chromatic でサインアップする](/design-systems-for-developers/chromatic-signup.png)

そこからデザインシステムのリポジトリ―を選択します。裏側では、これによりアクセス許可が同期され、PR チェックが設定されます。

<video autoPlay muted playsInline loop style="width:520px; margin: 0 auto;">
  <source
    src="/design-systems-for-developers/chromatic-setup-learnstorybook-design-system.mp4"
    type="video/mp4"
  />
</video>

[chromatic](https://www.npmjs.com/package/chromatic) のパッケージを npm でインストールします。

```shell
yarn add --dev chromatic
```

インストールしたら、Storybook をビルドしてデプロイするために以下のコマンドを実行します。(`project-token` には Chromatic で発行されたものを使用する必要があります。)

```shell
npx chromatic --project-token=<project-token>
```

![コマンドライン上の Chromatic](/design-systems-for-developers/chromatic-manual-storybook-console-log.png)

表示されたリンクをコピーし、ブラウザーに貼り付けて公開された Storybook を見てみてください。ローカルの Storybook の開発環境がオンライン上に公開されています。

![Chromatic でビルドされた Storybook](/design-systems-for-developers/chromatic-published-storybook-6-0.png)

これでローカルでしか見られなかった UI コンポーネントをチームに簡単にレビューしてもらえるようになります。Chromatic 上で確認する際の表示結果です。

![最初の Chromatic のビルド結果](/design-systems-for-developers/chromatic-first-build.png)

おめでとうございます！これで Storybook を公開するインフラを構築できました。継続的インテグレーションで改善していきましょう。

### 継続的インテグレーション

継続的インテグレーション (CI) とはモダンなウェブアプリケーションを保守する手法です。コードをプッシュした時にテストや、分析、デプロイなどの動作をスクリプト化できるようになります。このテクニックを借りて、繰り返しの手作業から開放されましょう。

ここでは GitHub Actions を使用します。今回の使い方ならば無料です。同じ原則が他の CI サービスにも適用できます。

`.github` フォルダーをトップレベルに追加し、`workflows` というフォルダーを作成します。

さらに以下の内容の `chromatic.yml` というファイルを作成します。これで CI プロセスの振る舞いをスクリプト化できます。今は小さく作っておき、作業を進めるにつれて改善し続けます:

```yaml
# .github/workflows/chromatic.yml

# name of our action
name: 'Chromatic'
# the event that will trigger the action
on: push

# what the action will do
jobs:
  test:
    # the operating system it will run on
    runs-on: ubuntu-latest
    # the list of steps that the action will go through
    steps:
      - uses: actions/checkout@v1
      - run: yarn
      - uses: chromaui/action@v1
        # options required to the GitHub chromatic action
        with:
          projectToken: project-token
          token: ${{ secrets.GITHUB_TOKEN }}
```

以下のコマンドで変更を追加し:

```shell
git add .
```

コミットします:

```shell
git commit -m "Storybook deployment with GitHub action"
```

最後にリモートリポジトリーにプッシュします。:

```shell
git push origin master
```

成功しました！インフラの改善ができました。

## チームへ視覚的なレビューを依頼する

ユーザーに配信される内容のコンセンサスを得るため、UI の変更が含まれるプルリクエスト毎に、利害関係者への視覚的なレビューのプロセスを開始するのが便利です。そうすれば、嬉しくない驚きや高価な手直しが不要になります。

新しいブランチで UI を変更し、視覚的なレビューのデモをしてみましょう。

```shell
git checkout -b improve-button
```

まずはボタンコンポーネントを変更します。デザイナーが気に入るよう、「ポップにします」。

```javascript
//src/Button.js

// ...
const StyledButton = styled.button`
  border: 10px solid red;
  font-size: 20px;
`;
// ...
```

変更をコミットし、GitHub のリポジトリーにプッシュします。

```shell
git commit -am "make Button pop"
git push -u origin improve-button
```

GitHub.com に移動し、`improve-button` ブランチをプルリクエストしてください。プルリクエストを作成すると、Storybook を公開する CI ジョブが実行されます。

![GitHub で PR を作成する](/design-systems-for-developers/github-created-pr-actions.png)

ページの下の方にある PR チェックのリストで **Storybook Publish** をクリックし、今回の変更が反映された Storybook が公開されているのを見ることができます。

![デプロイされたサイト上での Button コンポーネントの変更点](/design-systems-for-developers/chromatic-deployed-site-with-changed-button.png)

変更されたそれぞれのコンポーネントとストーリーについて、ブラウザーのアドレスバーから URL をコピーし、チームがタスクを管理している場所 (GitHub や Asana、Jira など) に貼り付けると、チームメートがすぐに関連するストーリーのレビューをできます。

![Storybook へのリンクがある GitHub の PR](/design-systems-for-developers/github-created-pr-with-links-actions.png)

チームメートを issue にアサインし、フィードバックが来るのを待ちましょう。

![なんでやねん?!](/design-systems-for-developers/github-visual-review-feedback.gif)

<div class="aside">Chromatic では製品の一部として完全な UI レビューのワークフローを有料で提供しています。Storybook のリンクを GitHub の PR に貼り付ける手法は小さなプロジェクトでは上手くいきます (Chromatic だけではなく Storybook をホストできる他のサービスでも) が、使用頻度が上がると、そのプロセスを自動化するサービスを検討したくなることでしょう。</div>

ソフトウェア開発において、欠陥の原因はほとんどテクノロジーではなく誤解から生じます。視覚的なレビューは、チームが継続的に開発中のフィードバックを集め、デザインシステムを迅速に配信するのに役立ちます。

![視覚的なレビューのプロセス](/design-systems-for-developers/visual-review-loop.jpg)

> プルリクエストごとに Storybook の URL をデプロイするのは Shopify のデザインシステムの Polaris でずっとやってきたことです。とても役に立ちました。- Shopify の エンジニア Ben Scott

## デザインシステムをテストする

視覚的なレビューはかけがえのないものですが、数百のコンポーネントを手動でレビューするのには時間がかかります。理想としては、意図的な変更 (追加・改善) のみを見て、意図的でないリグレッションは自動で捉えてほしいところです。

第 5 章では、視覚的なレビューをする際のノイズを減らし、コンポーネントが堅牢であり続けるためのテスト戦略を紹介します。
