# github-actions-example

「フロントエンドの開発のCI/CD with Gtihub Actions」 のサンプルディレクトリです。

## ディレクトリ構成

```txt
.
├── README.md
├── ./github/workflows      # Github Actionsのワークフローを格納するディレクトリ
└── frontend                # `create-react-app`を使って作成した最小限のReactプロジェクト
```

## Github Actionsとは
「`X`したときに`Y`する」が実現できる

例えば
- `push(=X)`した時に`テストを実行(=Y)`する
- `mainブランチにマージ(=X)`した時に`本番環境に最新のコードをデプロイ(=Y)`する

## Hello Worldを作ってみる

### STEP1　ワークフローを作成
`./github/workflows/hello_world.yaml`

```yaml
name: Hello_Github_Actions
on:
  push:
    branches:
      - "main"
jobs:
  hello_github_actions:
    runs-on: ubuntu-latest
    steps:
      - run: echo "Hello Github Actions"
```

これは`Hello Github Actions`と表示するだけのシンプルなワークフローです。

※`Github Actions`は`./github/workflows`直下に定義されたファイルを自動で読み取ってくれる


### STEP2 ワークフローを実行
Githubに何でも良いのでファイルを`push`してみよう！

うまくいけば下の画像のように、`Hello Github Actions`が表示されます。

![](/assets/screenshot_helloworld_workflow.png)


# 実践演習

## mainブランチへのマージの前にテストを実行して、失敗した場合マージできないように設定する

### STEP1. テストを実行するGithub Actionsを作成

 `.github/workflows/ci.yaml`

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