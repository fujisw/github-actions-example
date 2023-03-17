# github-actions-example

「フロントエンドの開発のCI/CD with Gtihub Actions」 のサンプルディレクトリです。

## ディレクトリ構成

```txt
.
├── README.md
├── ./github/workflows      # Github Actionsのワークフローを格納するディレクトリ
└── frontend                # `create-react-app`を使って作成した最小限のReactプロジェクト
```


## mainブランチへのマージの前にテストを実行して、失敗した場合マージできないように設定する

### STEP1. テストを実行するGithub Actionsを作成

 `.github/workflows`に下記のようなファイルを作成する

```yml
name: Continuous Integration

on:
  pull_request:
    branches:
      - main

jobs:
  build-and-test:
    runs-on: ubuntu-latest

    steps:
    - name: Checkout code
      uses: actions/checkout@v2

    - name: Set up Node.js
      uses: actions/setup-node@v2
      with:
        node-version: 18.x

    - name: Install dependencies
      working-directory: frontend
      run: npm ci

    - name: Run tests
      working-directory: frontend
      run: npm test -- --coverage --watchAll=false
```

### STEP2. Githubのブランチ保護のルールを記載

下の画像のように設定することで、 上のYMLで定義した`build-and-test`ジョブをマージの前に実行できる。

もし、`build-and-test`が失敗すれば、マージをすることはできなくなる。

![](/assets/screen_shot_of_branch_protection_setting.png)


## Github Actionsをデバッグする方法

ワークフローを更新するたびに、毎回リモートリポジトリにpushして動作確認をするのは大変。

[act](https://github.com/nektos/act)を使用すると、ローカルでワークフローを実行できる。便利。