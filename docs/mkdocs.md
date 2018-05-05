# MkDocs

## 参考サイト

* [mkdocs.org](http://mkdocs.org)

* [DARING FIREBALL](https://daringfireball.net/projects/markdown/)

    Markdown創始者

* [Material for MkDocs](https://squidfunk.github.io/mkdocs-material/)

* [MkDocsによるドキュメント作成](https://qiita.com/mebiusbox2/items/a61d42878266af969e3c)

## インストール

 1. Pythonをインストールする。

    PATH設定要。Program Files配下にインストールしたい場合は、Customインストールする。

 1. pipコマンドでmkdocs, mkdocs-materialをインストールする。

    `pip install mkdocs`  
    `pip install mkdocs-material`

 1. プロジェクトを作成する。

    ``` bash
    mkdocs new test
    ```

 1. mkdocs.ymlを編集する。

## GitHubへのデプロイ

 1. デプロイ先のGitHubで、Personal Access Tokenを発行する。

    発行するURLは[こちら](https://github.com/settings/tokens)

 1. デプロイするリポジトリのRempte Origin URLを下記のように指定する。

    `git config remote.origin.url https://{token}@github.com/{username}/{project}.git`

 1. Powershellを立ち上げる。下記コマンドで、文字コードをutf-8に変更する。

    `chcp 65001`

 1. 下記コマンドで、GitHubにデプロイする。

    `mkdocs gh-deploy`

 1. 下記URLで公開される。

    `https://{username}.github.io/{project}/`

## コマンド

* `mkdocs new [dir-name]` - Create a new project.
* `mkdocs serve` - Start the live-reloading docs server.
* `mkdocs build` - Build the documentation site.
* `mkdocs help` - Print this help message.
