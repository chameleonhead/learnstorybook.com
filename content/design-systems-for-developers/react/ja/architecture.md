---
title: 'システムを構築する'
tocTitle: 'アーキテクチャー'
description: 'コンポーネントライブラリーからデザインシステムを抽出する方法'
commit: b696f85
---

第 2 章では、既に存在するコンポーネントライブラリーからデザインシステムを抽出します。そのプロセスを通して、デザインシステムに属すべきコンポーネントを判断し、また、始める際に開発者が直面する共通的な課題を説明します。

大きな会社では、この課題はデザインチーム、技術チーム、プロダクトチームが協力して解決します。Chromatic (Storybook の会社) と Storybook は 3 つ以上の製品で、800 以上のオープンソースコントリビューターにサービスを提供する、活発なフロントエンドのインフラチームを共有していますので、そのプロセスをご説明しましょう。

## 課題

開発チームで働いているなら、大きなチームがそれほど効率的でないと気付いていることでしょう。誤解はチームが成長するにつれてはびこります。UI パターンはドキュメント化されなくなるか、全くなくなってしまいます。つまり、開発者は新機能の追加よりも、車輪の再発明に時間が割かれます。いずれ、プロジェクトには 1 度しか使われないコンポーネントが溢れていきます。

我々もこの窮地に陥りました。経験豊富なチームの最善の選択にも関わらず UI コンポーネントは繰り返し作られ、コピーされていきます。同じはずの UI パターンでも、見た目と機能が多様化していきました。それぞれのコンポーネントはユニークな雪の結晶のように、新参の開発者が信頼できる情報源の区別を不可能にし、生産性を大幅に低下させます。

![多様化していく UI](/design-systems-for-developers/design-system-inconsistent-buttons.jpg)

## デザインシステムを作る

デザインシステムは共通の UI コンポーネントの集まりで、よくメンテナンスされたリポジトリーで、パッケージマネージャーを介して配布されます。開発者は複数のプロジェクトで同じコードを使い回す代わりに標準化された UI コンポーネントをインポートします。

ほとんどのデザインシステムは最初から作成されるわけではなく、会社の中で使用されている、信頼ある UI コンポーネントから、デザインシステムとして再構成します。我々のプロジェクトも例外ではありません。使用されているコンポーネントライブラリーからコンポーネントをチェリーピックし、短い時間で利害関係者にデザインシステムを届けます。

![デザインシステムの構成](/design-systems-for-developers/design-system-contents.jpg)

## デザインシステムはどこにあるのか

デザインシステムというのは、1 つのアプリケーションに対してだけでなく、組織全体に対してのコンポーネントライブラリーのようなものです。デザインシステムは UI 要素にフォーカスし、プロジェクト固有のコンポーネントライブラリーは画面についての複合的なコンポーネントを含みます。

従って、デザインシステムはどのプロジェクトにも属さず、また、全てのプロジェクトから参照されます。デザインシステムへの変更はバージョン管理されたパッケージとしてパッケージマネージャーを介して伝播します。プロジェクトでは、デザインシステムのコンポーネントを再利用したり、必要ならば大幅に変更したりできます。この制約が、フロントエンドのプロジェクトを構築する際の青写真となるのです。

![誰がデザインシステムを使うのか](/design-systems-for-developers/design-system-consumers.jpg)

## create-react-app と GitHub でリポジトリーをセットアップする

[State of JS](https://stateofjs.com/) の調査によれば、React はビュー層で最も人気があります。Storybook においても React での使用が圧倒的なので、このチュートリアルでは React を [create-react-app](https://github.com/facebook/create-react-app) のボイラープレートとともに使用します。

コマンドラインで次のコマンドを実行してください:

```shell
# Clone the files
npx degit chromaui/learnstorybook-design-system#setup learnstorybook-design-system

cd learnstorybook-design-system

# Install the dependencies
yarn install
```

<div class="aside">
GitHub からフォルダーをダウンロードするために <a href="https://github.com/Rich-Harris/degit">degit</a> を使用します。手動でダウンロードする際は、<a href="https://github.com/chromaui/learnstorybook-design-system/tree/setup">こちら</a>からどうぞ。
</div>

ファイルをダウンロードして、依存関係のインストールが終わったら、GitHub (デザインシステムのコードをホストするために使用) にプッシュできます。GitHub.com にサインインして新しいリポジトリーを作りましょう。

![GitHub のリポジトリーを作成する](/design-systems-for-developers/create-github-repository.png)

次に、GitHub の指示に従ってリポジトリーを (今のところはほとんど空ですが) セットアップします:

```shell
git init
git add .
git commit -m "first commit"
git branch -M master
git remote add origin https://github.com/your-username/learnstorybook-design-system.git
git push -u origin master
```

`your-username` は忘れずにアカウント名で置き換えてください。

![GitHub のリポジトリーに初回コミットを作成](/design-systems-for-developers/created-github-repository.png)

<div class="aside">デザインシステムを作成する他の方法として、素の HTML/CSS を配布する、他のビュー層のフレームワークを使用する、Svelte を使用してコンポーネントをコンパイルする、Web コンポーネントを使用するといった方法もあります。チームにあったものを選んでください。</div>

## デザインシステムに含まれるものと含まれないもの

デザインシステムには[プレゼンテーショナルコンポーネント](https://medium.com/@dan_abramov/smart-and-dumb-components-7ca2f9a7c7d0)のみが含まれるべきです。プレゼンテーショナルコンポーネントはどのように UI を表示するかを扱います。コンポーネントに渡されるプロパティに反応し、アプリケーション独自のビジネスロジックは含まず、データの読み込み方法は関知しません。これらの性質がコンポーネントを再利用するために不可欠です。

デザインシステムは組織のコンポーネントライブラリーの全てを含むわけではありません。全体を含んでしまうと変更を追跡するのが苦痛です。

ビジネスロジックを含むアプリケーション独自のコンポーネントは含むべきではありません。なぜならデザインシステムを使用する側の全てのプロジェクトでビジネス要求が一致させなければならず、再利用を難しくしてしまいます。

再利用していない、一度きりのコンポーネントは除外します。もしそれがデザインシステムの一部となるべきコンポーネントでも、機敏なチームでは余計なコードのメンテナンスをできる限り避けます。

## コンポーネント一覧を作る

デザインシステムの最初の仕事は最も使用されているコンポーネントを特定するため、コンポーネント一覧を作成することです。共通的な UI コンポーネントを識別するために、大抵は手動で複数のウェブサイトやアプリケーションの画面をカタログ化することになります。[Brad Frost](http://bradfrost.com/blog/post/interface-inventory/) や [Nathan Curtis](https://medium.com/eightshapes-llc/the-component-cut-up-workshop-1378ae110517) といったデザイナーがコンポーネント一覧の作り方に関する手法を公開していますので、このチュートリアルでは詳細は説明しません。

開発者向けの経験則:

- ある UI パターンが 3 回以上使用されている場合、再利用可能な UI コンポーネントにしましょう。
- ある UI コンポーネントが 3 つ以上のプロジェクトやチームで使用されている場合、デザインシステムに含めましょう。

![デザインシステムの中身](/design-systems-for-developers/design-system-grid.png)

この手法に従って、Avatar、Badge、Button、Checkbox、Input、Radio、Select、Textarea、Highlight (コード用)、Icon、Link、Tooltip などを UI プリミティブとしました。クライアントアプリケーションで無数の機能や、レイアウトを作りあげるために、このようなブロックを様々に組み合わせられるようになっています。

![コンポーネントのバリエーション](/design-systems-for-developers/design-system-consolidate-into-one-button.jpg)

リポジトリーを簡潔にするため、コードサンプルにそういったコンポーネントの一部を選びました。チームによっては Table や Form といったコンポーネント向けにサードパーティーのコンポーネントをカスタマイズしたものを含めます。

<div class="aside">JavaScript 内の CSS: このガイドではコンポーネントに影響範囲を限定してスタイルを設定するライブラリーとして <a href="https://www.styled-components.com">styled-components</a> を使用します。手動で CSS クラスを設定する CSS modules など、他にもコンポーネントにスタイルを設定する方法はあります。</div>

UI コンポーネントだけでなく、フォント、色、余白といったプロジェクトをまたいで再利用する定数もデザインシステムに含めるのが良いでしょう。デザインシステムの用語ではグローバルなスタイル変数は「デザイントークン」と呼んでいます。このガイドではデザイントークンの背景にある原理について深くは追求しませんが、ネットで調べてみてください。(こちらに[良い記事](https://medium.com/eightshapes-llc/tokens-in-design-systems-25dd82d58421)があります。)

## 開発を始めましょう

何を作り、どのように組み合わせるかを示したので、作業を始めてみましょう。第 3 章ではデザインシステムの基礎となるツールをセットアップします。生の UI コンポーネントのディレクトリが、Storybook でカタログ化され、参照できるようになります。
