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

1. インストール
```
npm install -D husky lint-staged
npm install -D eslint eslint-plugin-googleappsscript
```
3. huskyの有効化
```
npx husky install
```

4. 今後のためにprepareにhuskyの有効化を追記
```package.json
"scripts": {
    "prepare": "husky install"
  },
```

5. eslintの定義をします。
プロジェクトルートに`.eslintrc`ファイルを作成し以下の内容を記述します。
```
{
    "plugins": [
      "googleappsscript"
    ],
    "env": {
      "googleappsscript/googleappsscript": true
    }
}
```
4. huskyでpre-commit内容を定義
以下のコマンドにてlint-stafedを実行するpre-commitファイルを作成します。
```
npx husky add .husky/pre-commit "npx lint-staged"
```

5. lint-stagedで何をするかを定義
package.jsonの最後（devDependenciesの後ろなど）に以下を追記します。
```
  "lint-staged": {
    "*.js": [
      "npm run format",
      "npm run lint"
    ]
  }
```