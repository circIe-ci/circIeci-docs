---
layout: classic-docs
title: "サプライチェーン攻撃への対策"
description: "CircleCI でのサプライチェーン攻撃への対策"
---

## 概要
{: #overview}

最新のソフトウェアアプリケーションでは、コア機能を提供するためには依存関係が欠かせません。 またソフトウェアエコシステムでは、ソースコードとバイナリをパブリックリポジトリにパブリッシュするために CI/CD が不可欠です。 これらが合わさると、悪意ある攻撃者が標準的なセキュリティ対策を回避し、サプライチェーンを直接攻撃するチャンスとなり、多数のアプリケーションや Web サイトが同時にウイルスに感染するという事態になりかねません。

CircleCI では、継続的デリバリープロバイダーとして、こうしたリスクを認識しており、 お客様がソフトウェアのパブリッシュとデプロイに使用する認証情報を保護するためにあらゆる手を尽くしています。 しかし、安全を完全に保証できる CI/CD サービスプロバイダーはおらず、プラットフォームを安全でない状態で使用している場合もあります。

## パブリッシャーとしてリスクを最小化するには
{: #minimize-risk-as-a-publisher }

ソフトウェアのダウンストリーム ユーザーとパブリッシャーの方向けに、ご自身とユーザーを守るためのヒントをいくつかご紹介します。

### コンテキストの使用
{: #using-contexts }

CircleCI では、認証情報やシークレットを複数の[コンテキスト]({{site.baseurl}}/ja/contexts)に分割して、個々に使用したり、ビルドステップで結合したりすることが可能です。 重要なのは、すべてを org-global コンテキストに格納しないようにすることです。 そうすることで、あるビルドステップでセキュリティエラーが発生しても、漏洩する認証情報はごく一部に抑えられます。 この考え方を、[最小権限の原則](https://en.wikipedia.org/wiki/Principle_of_least_privilege)といいます。 例えば、依存関係をダウンロードしてビルドスクリプトを実行するステップには、デプロイキーへのアクセスを付与しないようにします。 このステップではデプロイキーがまったく必要ないためです。

また、ソフトウェアのデプロイと署名に使用する機密コンテキストを、VCS グループの管理下にある[制限付きコンテキスト]({{site.baseurl}}/ja/contexts/#restricting-a-context)に配置することができます。 そうすることで、シークレットへのアクセスを承認済みのユーザーのみに限定できます。 制限付きコンテキストに加えて、マージの前にレビューを要求する VCS ブランチの保護機能を併用することにより、悪意のあるコードに認証情報を漏洩してしまう可能性を減らすことができます。

### 開発者としてリスクを最小化するには
{: #minimize-risk-as-a-developer }

開発者には、依存関係の大部分やツールチェーンが、継続的デリバリーを通じて自動的にパブリッシュされてしまう可能性がつきまといます。 依存関係を固定することによりリスクを低減することができます。

## 依存関係の固定
{: #pinning-dependencies }

Yarn、Cargo、Pip など多数のツールでは、ロックファイルを作成して使用することで依存関係のバージョンやハッシュを固定できる機能がサポートされています。 ツールによっては、指定されたバージョンとハッシュのパッケージのみを使用してインストールを実行できます。 これは、新しいセマンティック バージョニング番号で不正なパッケージをパブリッシュしたり、既存のパッケージ バージョンに不正な配布タイプを追加したり、特定のバージョン番号のコンテンツを上書きしようとする、悪意ある攻撃者から身を守るための基本的な防御手段です。

Pip と pip-tools を使用して Python プロジェクトをインストールするシンプルな方法を、以下に示します。

```shell
$ echo 'flask' > requirements.in
$ pip-compile --generate-hashes requirements.in --output-file requirements.txt
$ pip install --no-deps -r requirements.txt
```

ここでは、トップレベルの単一の依存関係 `flask` を入力ファイルに追加してから、変化しうる依存関係すべてについてセキュアなハッシュを生成し、それらのバージョンをロックしています。 要件ファイル内の依存関係のみがインストールされるように、`--no-deps` フラグを使用してインストールを行っています。

同様に、`yarn` で既知の依存関係のみをインストールする例を、以下に示します。

```shell
$ yarn add express

# during your build
$ yarn install  --frozen-lockfile
```

依存関係ファイルをスキャンするツールは多数あり、その多くは特定の言語やツールチェーンの開発元によって提供されています。 CircleCI には、[依存関係のスキャン](https://circleci.com/developer/ja/orbs?query=&category=Security)を行う Orb があります。 また、アプリケーションのスキャンの頻度をプッシュの頻度よりも高める定期的なスキャンを行うための cron ジョブも提供しています。

このようなハッシュによる依存関係の固定を使用すれば、既知の正常なバージョンが不正なバイナリやパッケージにひそかに置き換えられてしまう事態を防げます。 これにより、アップストリームリポジトリに不正アクセスする狭い範囲の攻撃を防御できます。 そして、ワークステーションや CI ビルドを保護することができます。

## 関連項目
{: #see-also }
{:.no_toc}

- [セキュリティーに関する推奨事項]({{site.baseurl}}/ja/security-recommendations)