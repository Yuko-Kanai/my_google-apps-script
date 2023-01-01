# my_google-apps-script
google apps script for my work

# Installation
1. github上に新しいリポジトリを作成

2. node.jsのインストール（claspがnodeのライブラリのため）16.15.1にする

3. ローカルへclone
```
git clone git@github.com:Yuko-Kanai/my_google-apps-script.git
```

4. Google Apps Script API を「オン」にする
https://script.google.com/home/usersettings

5. 以下を参考にpackage.jsonを作成してインストールする
```
npm install
npm install @google/clasp
```
参考：https://github.com/C-FO/qa-google-apps-srcipt/blob/main/package.json

6. (任意）qaのリポジトリと構成を合わせておく
```
mkdir qa_process
cd qa_process
```

7. 任意のディレクトリを作成する
```
mkdir study
```

8. ログインする
ログインすると認証を求められるので許可する
```
cd study
npx clasp login
```

### スタンドアロンプロジェクトの場合
googledriveから作成する場合、スタンドアロンプロジェクトとなります。
アプリケーションから作成するコンテナバインドプロジェクトでのみ使える機能が使えません。

9. googledriveにて新規作成＞Google Apps Scriptを押下すると新しいScriptが作成できるので今回はそちらでためす
作成したScriptをChromeで開いてURLからIDを取得する

10. 以下をターミナルで実行する
```
npx clasp clone 1_r06p6-znUd8Cm1Hwm2zHPJrLZOc3KlQ6zTTLz_fcelH_vyYRZF3UqCf
```

11. 任意のエディタで編集する

12. 編集したコードをGASへアップする
```
npx clasp push
```

13. 動作確認
```
npx clasp open
```
ブラウザで開くので「実行」を押下

14. 問題なければgithubへpush

### GAS側の更新を手元のclasp環境へ反映させたい時
```
npx clasp pull
```

### スタンドアロンをアプリケーションから使う方法
参考：https://teratail.com/questions/357958

1. アプリケーションのGASを開き、ライブラリからスタンドアロンプロジェクトのIDを追加

2. アプリケーション側のコンテナバインドプロジェクトに呼び出し方法を記述
```
プロジェクト名.関数名()
```
3. アプリケーション側のコンテナバインドプロジェクトを「実行」して動作確認

参考：https://teratail.com/questions/357958

# コミット前にフォーマットチェックなどを通す
huskyとlint-stagedを使います。
- [git hook](https://git-scm.com/book/ja/v2/Git-%E3%81%AE%E3%82%AB%E3%82%B9%E3%82%BF%E3%83%9E%E3%82%A4%E3%82%BA-Git-%E3%83%95%E3%83%83%E3%82%AF)
- `git commit`時に発生するGitフック「pre-commit」をhuskyでハンドリング
- `lint-staged`ででGitのステージに上がっているファイルを対象にeslint等のコマンドを実行できる

### フォーマットチェックなどのツールの準備
1. インストール
```
npm install -D eslint eslint-plugin-googleappsscript
npm install -D xo prettier-config-xo
```

2. スクリプトの設定
eslintとprettierでやりたいことを書きます。
```package.json
"scripts": {
    "lint": "eslint --cache --fix",
    "format": "prettier --write",
  },
```

3. eslintの定義をします。
プロジェクトルートに`.eslintrc`ファイルを作成し以下の内容を記述します。
```
{
    "parserOptions": {
        "ecmaVersion": "latest"
    },
    "plugins": [
      "googleappsscript"
    ],
    "env": {
      "googleappsscript/googleappsscript": true
    }
}
```

### pre-commitの設定
1. インストール
```
npm install -D husky lint-staged
```

2. huskyの有効化
```
npx husky install
```

3. 今後のためにprepareにhuskyの有効化を追記
```package.json
"scripts": {
    "prepare": "husky install"
  },
```

4. huskyの初期準備
package.jsonの最後（devDependenciesの後ろなど）に以下を追記します。
```
  "husky": {
    "hooks": {
      "pre-commit": "lint-staged"
    }
  }
```

5. hooksの作成
以下のコマンドにてlint-stafedを実行するpre-commitファイルを作成します。
```
npx husky add .husky/pre-commit "npx lint-staged"
```

6. lint-stagedで何をするかを定義
package.jsonの最後（devDependenciesの後ろなど）に以下を追記します。
```
  "lint-staged": {
    "**/**/*.{js, ts}": [
      "npm run format",
      "npm run lint"
    ]
  }
```
# GitHub ActionsでGASへのデプロイまで行う
これまでは、GASへのpushとgithubへのpush両方手動で行なっていましたが、githubへのpushでGASへのpushも行うようにします。

1. Github Actionsのワークフローファイルを格納するためのディレクトリを作成します。
```
mkdir .github
mkdir .github/workflows
```

2. release.ymlを作成します。
25行目の~/.clasprc.jsonファイルは、claspをインストールしたローカル端末に作成されていますがシークレット情報のためgithubにあげられるものではありません。
そのためここでは、github上でclaspコマンドを使うためにgithub上でclasprc.jsonファイルを作成しています。

```:release.yml
name: Publish Release

on:
  push:
    tags:
      - "v*"

jobs:
  check-bats-version:
    runs-on: ubuntu-latest

    env:
      CLASPRC_ACCESS_TOKEN: ${{ secrets.CLASPRC_ACCESS_TOKEN }}
      CLASPRC_CLIENT_ID: ${{ secrets.CLASPRC_CLIENT_ID }}
      CLASPRC_CLIENT_SECRET: ${{ secrets.CLASPRC_CLIENT_SECRET }}
      CLASPRC_EXPIRY_DATE: ${{ secrets.CLASPRC_EXPIRY_DATE }}
      CLASPRC_ID_TOKEN: ${{ secrets.CLASPRC_ID_TOKEN }}
      CLASPRC_REFRESH_TOKEN: ${{ secrets.CLASPRC_REFRESH_TOKEN }}

    steps:
      - name: Checkout
        uses: actions/checkout@v3
      - name: Setup Node.js 16.x
        uses: actions/setup-node@v3
        with:
          node-version: '16'
      - name: Install Clasp
        run: |
          npm init -y
          npm install clasp -g
      - name: Create ~/.clasprc.json
        run: |
          echo $(cat <<-EOS
          {
            "token": {
              "access_token": "${CLASPRC_ACCESS_TOKEN}",
              "scope": "https://www.googleapis.com/auth/script.deployments https://www.googleapis.com/auth/userinfo.profile https://www.googleapis.com/auth/drive.file openid https://www.googleapis.com/auth/service.management https://www.googleapis.com/auth/script.projects https://www.googleapis.com/auth/userinfo.email https://www.googleapis.com/auth/drive.metadata.readonly https://www.googleapis.com/auth/logging.read https://www.googleapis.com/auth/cloud-platform https://www.googleapis.com/auth/script.webapp.deploy",
              "token_type": "Bearer",
              "id_token": "${CLASPRC_ID_TOKEN}",
              "expiry_date": ${CLASPRC_EXPIRY_DATE},
              "refresh_token": "${CLASPRC_REFRESH_TOKEN}"
            },
            "oauth2ClientSettings": {
              "clientId": "${CLASPRC_CLIENT_ID}",
              "clientSecret": "${CLASPRC_CLIENT_SECRET}",
              "redirectUri": "http://localhost"
            },
            "isLocalCreds": false
          }
          EOS
          ) > ~/.clasprc.json
      - name: Get version
        id: get_version
        run: echo ::set-output name=VERSION::${GITHUB_REF#refs/tags/}

      - name: Upload files in study
        run: |
          cd qa_process/study
          npx @google/clasp push --force
      - name: Add version
        run: npx @google/clasp version ${{ steps.get_version.outputs.VERSION }}
```
参考：https://docs.github.com/ja/actions/learn-github-actions/understanding-github-actions

参考：https://dev.classmethod.jp/articles/github-actions-gas-deploy/

また、ここの部分はgasの種類分定義する必要があります。
```
      - name: Upload files in study
        run: |
          cd qa_process/study
          npx @google/clasp push --force
```

3. ローカルの`~/.clasprc.json`ファイルの内容を、Githubの環境変数に設定して、前手順の25行目のファイル作成時に参照できるようにします。
Githubへアクセスし、対象のリポジトリ＞settings>Secrets>Actionsの「New repository secret」を押下して以下を追加
- access_tokenの値
- id_tokenの値
- refresh_tokenの値
- clientIdの値
- clientSecretの値

4. .clasp.jsonのルートディレクトリをgithubにも対応させる。
clacp cloneをしたディレクトリには`.clasp.json`ファイルが作成されますが絶対パスのため、このままだとGithub上で`appsscript.json`などを参照できません。
これを
```:変更前
"rootDir":"/Users/yuko-kanai/work/my_google-apps-script/qa_process/study"
```
こうします
```:変更後
"rootDir":"./"
```

5. ワークフローがtagをつけたpushに対応しているため、デプロイしたいときは、tagをつける必要があります。
```:例
git commit -m "github actions 対応"
git tag -a v1.0 -m "version 1.0"
git push
