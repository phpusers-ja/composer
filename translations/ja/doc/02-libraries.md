# ライブラリ

この章ではライブラリをComposerでインストールできるようにする方法を学びます。

## 全てのプロジェクトはパッケージである

`composer.json`をディレクトリに配置した時点で、そのディレクトリはパッケージとなります。
[`require`](04-schema.md#require)をプロジェクトに追加すると、他のパッケージに依存したパッケージを作っていることになります。
プロジェクトとライブラリの唯一の違いは、プロジェクトは名前のないパッケージだということです。

インストール可能なパッケージを作成するためには命名する必要があります。
`composer.json`に[`name`プロパティ](04-schema.md#name)を追加してください。

```json
{
    "name": "acme/hello-world",
    "require": {
        "monolog/monolog": "1.0.*"
    }
}
```

この例では、プロジェクト名は`acme/hello-world`です。
`acme`はベンダー名です。
ベンダー名を与えることは必須です。

> **注意：**
> もしベンダー名に何をつけていいか分からない場合は、大抵は自分のGitHubの利用者名をつけるといいでしょう。
> パッケージ名は全て小文字でなければならず、単語の区切りはダッシュにするのが慣習です。

## ライブラリのバージョン

まず間違いなくgit, svn, hg, fossilといった何らかの類のバージョン管理システムを使ってライブラリを管理することでしょう。
こうした場合、ComposerはVCSからバージョンを推定するので`composer.json`ファイルではバージョンを指定すべきでは**ありません**（ComposerがVCSのブランチとタグを使ってバージョン制約を解決する方法について学ぶためには[バージョンについての記事](articles/versions.md)を参照してください）。

パッケージの管理を手作業でしている（つまりVCS無しの）場合、`composer.json`ファイルに`version`値を加えて、バージョンを明示的に指定する必要があるでしょう。

```json
{
    "version": "1.0.0"
}
```

> **補足：** VCSに埋め込まれているバージョンを加えた場合、バージョンはタグ名と干渉することでしょう。
そうするとComposerはバージョン値を決定できなくなります。

### VCSのバージョン管理

ComposerはVCSのブランチとタグの機能を使って[`require`](04-schema.md#require)フィールドで指定したバージョン制約を特定のファイルの集まりに至るまで解決します。
利用できる妥当なバージョンを決定する際、Composerは全てのタグとブランチを見て、それらの名前を内部的なオプションの一覧に翻訳し、それから与えられたバージョン制約に対して照合します。

Composerがタグとブランチを扱う方法とパッケージのバージョン制約を解決する方法についての詳細は、[バージョン](articles/versions.md)の記事をお読みください。

## 固定ファイル

ご自身のライブラリについて、 **`composer.lock`ファイルをコミットすることが推奨されます。**
これにより、チームが常に同じ依存関係の版に対してテストする助けになり、伝播してくるサプライチェーン攻撃を防止することに役立つことがあります。
というのも、新しく公開された（悪意のある可能性のある）依存関係の版は、陽に更新しない限りは、CIや開発環境へ取り込まれないので。

This protection only covers working on the library itself. The lock file has
no effect on projects that depend on your library, they install whatever
versions your constraints allow. You can therefore exclude it from dist
archives using `export-ignore`, see [Light-weight distribution
packages](#light-weight-distribution-packages).

As your users will not get the locked versions, keep the lock file up to
date and also run CI jobs with `composer update` (and `composer update
--prefer-lowest`)  to catch breakage from new releases or too low lower
bounds. As these jobs install unlocked versions, consider configuring a
[cooldown](06-config.md#cooldown), do not give them access to secrets or
write permissions (e.g. to your repository), and do not run `composer
update` in any build or release steps.

## VCSに公開する

`composer.json`ファイルを含むVCSリポジトリ（バージョン管理システム、例えばgit）があれば、ライブラリは既にcomposerでインストール可能です。
この例ではGitHubで`acme/hello-world`ライブラリを`github.com/username/hello-world`として公開するとしましょう。

それでは`acme/hello-world`パッケージのインストールを試すために、ローカルに新しいプロジェクトを作成しましょう。
私たちはそれを`acme/blog`と呼ぶことにします。
このブログは`acme/hello-world`に依存し、更に`monolog/monolog`に依存するとします。
以下の`composer.json`を含む新しい`blog`ディレクトリを作成することで達成されます。

```json
{
    "name": "acme/blog",
    "require": {
        "acme/hello-world": "dev-master"
    }
}
```

名前はこの場合必須ではありません。
このブログをライブラリとして公開することはないからです。
ここではどの`composer.json`が説明されているのかを明確にするために加えられています。

このブログアプリに依存物`hello-world`の所在を知らせる必要があります。
これにはパッケージのリポジトリ指定をこのブログの`composer.json`に追加します。

```json
{
    "name": "acme/blog",
    "repositories": [
        {
            "type": "vcs",
            "url": "https://github.com/username/hello-world"
        }
    ],
    "require": {
        "acme/hello-world": "dev-master"
    }
}
```

パッケージリポジトリの挙動や他にどのようなタイプが利用できるかについての詳細は、[リポジトリ](05-repositories.md)を参照してください。

以上です。
これでComposerの[`install`](03-cli.md#install--i)コマンドを実行すれば依存関係をインストールできます！

**まとめ：**
`composer.json`を含むあらゆるgit/svn/hg/fossilリポジトリは、[`require`](04-schema.md#require)フィールドでパッケージリポジトリを指定して依存関係を宣言することで、プロジェクトに追加できます。

## Packagistに公開する

よろしい、今やパッケージを公開できるようになりました。
しかし、毎回VCSリポジトリを指定するのは厄介なことです。
全ての利用者にそんなことはさせたくないでしょう。

`monolog/monolog`のためのパッケージリポジトリを指定しなかったこととにお気付きかもしれません。
どのような仕組みなのでしょうか？
答えはPackagistです。

[Packagist](https://packagist.org/)はComposerの主眼のパッケージリポジトリで、 既定で有効になっています。
Packagistで公開されている全てのものは自動的にComposerで利用可能です。
[monologはPackagistにある](https://packagist.org/packages/monolog/monolog)ので、追加のリポジトリ指定なくして依存できるのです。

`hello-world`を世界に共有したければ、同様にPackagistに公開するのが良いでしょう。

[Packagist](https://packagist.org)を開いて"Submit"ボタンを押します。
まだサインアップしていなかったらその旨の表示がされます。
それからVCSリポジトリのURLを送信できます。
送信した時点でPackagistはクローリングを始めます。
完了すると、パッケージは誰でも使えるようになります！

## 軽量配布パッケージ

有用でない情報は普通、配布パッケージに含めない方が良いでしょう。
これには`.github`ディレクトリ、嵩張る例、テストデータなどがあります。

`.gitattributes`ファイルは`.gitignore`のようなgit固有のファイルです。
またライブラリの根幹ディレクトリにあります。
これが存在してgitで追跡されているとき、局所的な構成と大域的な構成（それぞれ`.git/config`と`~/.gitconfig`）がオーバーライドされます。

zipの配布パッケージが肥大化させる望ましくないファイルが入らないようにするには、`.gitattributes`を使ってください。

```text
// .gitattributes
/demo export-ignore
phpunit.xml.dist export-ignore
/.github/ export-ignore
/composer.lock export-ignore
```

生成されたzipファイルを手作業で調べて確認するには、次のようにします。

```shell
git archive branchName --format zip -o file.zip
```

> **補足：**
> ファイルはgitで追跡されたままであり、zipの配布物に含まれていないだけです。
> これはdist（タグ付けされたリリース）でインストールしたパッケージでのみ機能します。
> 配布元はGitHub、GitLab、BitBucketがあります。

## セキュリティ勧告

When a vulnerability exists in a library you manage, you can fix the
vulnerability and publish a new release. Users watching your repository may
see the release, but most will not know they need to update to fix the
underlying vulnerability.

### How users get notified about advisories

Composer retrieves security advisories for all dependencies from the
configured repositories, which for most packages means the [packagist.org
API](https://packagist.org/apidoc#list-security-advisories). This happens
when users run `composer audit`, but also during `composer update`,
`require` and `remove`, which by default refuse to install versions affected
by an advisory (see the [policy](06-config.md#policy) config).

Packagist aggregates security advisories from multiple sources:

- [GitHub Security
  Advisories](#creating-and-publishing-an-advisory-on-github)
- [FriendsOfPHP Security
  Advisories](#registering-an-advisory-with-friendsofphp)

It doesn't retrieve published advisories from your repository itself.

### Creating and publishing an advisory on GitHub

If you are using GitHub to host the code for your library, you can [create a
security
advisory](https://docs.github.com/en/code-security/security-advisories/working-with-repository-security-advisories/creating-a-repository-security-advisory)
from the repository's security tab.

This will result in a unique url and GHSA-ID that can be used to reference
this advisory. Please note it does NOT mean that users of your library will
immediately get notified.

### How GitHub Security Advisories are picked up

Once you have [created and
published](#creating-and-publishing-an-advisory-on-github)  a security
advisory on GitHub, it MIGHT automatically show up in the [GitHub advisory
database](https://github.com/github/advisory-database). According to
documentation, "GitHub will review each published security advisory", and
mentions [this might take up to 72
hours](https://docs.github.com/en/code-security/concepts/vulnerability-reporting-and-management/repository-security-advisories#dependabot-alerts-for-published-security-advisories),
but in practice this may take significantly longer given that these reports
are reviewed manually.

The `github/advisory-database` repository also includes advisories from the
`friendsofphp/security-advisories` repository. As such, it is recommended to
also add your advisory to that repository to make sure it is visible to
users as soon as possible.

### Registering an advisory with FriendsOfPHP

To add an advisory to the `friendsofphp/security-advisories`
[repository](https://github.com/FriendsOfPHP/security-advisories), open a
Pull Request using the guide in their README. Once this Pull Request is
merged, packagist.org syncs the new advisory shortly after.

This will also eventually result in the advisory ending up in the
`github/advisory-database` repository if it's not in there already.

### Verifying advisory discoverability on packagist.org

You can see if a security advisory is already known to Packagist.org by
searching for the PKSA, CVE or GHSA IDs in the [Security Advisories
search](https://packagist.org/security-advisories/).

### Why is GitHub's Dependabot not notifying users of my library's advisory?

Dependabot currently only uses the `github/advisory-database` repository as
a source of advisories. If your published advisory is not in there, it won't
trigger dependabot. To resolve this, publish your advisory to the
[`friendsofphp/security-advisories`](#registering-an-advisory-with-friendsofphp)
repository. That will at least ensure that Composer warns, and will also
immediately be picked up by other alerting services like [Private
Packagist](https://packagist.com).

&larr; [基本的な使い方](01-basic-usage.md) |  [コマンドラインインターフェース](03-cli.md) &rarr;
