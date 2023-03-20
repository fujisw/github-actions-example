# github-actions-example

「フロントエンドの開発のCI/CD with Gtihub Actions」 のサンプルディレクトリです。

## ディレクトリ構成

```txt
.
├── README.md
├── ./github/workflows      # Github Actionsのワークフローを格納するディレクトリ
├── frontend                # `create-react-app`を使って作成した最小限のReactプロジェクト
└── openapi                 # OpenAPIのSPECファイルが格納されている
```

## Github Actionsとは
「`X`したときに`Y`する」が実現できる

例えば
- `push(=X)`した時に`テストを実行(=Y)`する
- `mainブランチにマージ(=X)`した時に`本番環境に最新のコードをデプロイ(=Y)`する

## Hello Worldを作ってみる

### STEP1　ワークフローを作成

```yaml
# ./github/workflows/hello_world.yaml
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

## リンク

### [act](https://github.com/nektos/act)

ワークフローを更新するたびに、毎回リモートリポジトリにpushして動作確認をするのは大変。

actを使用すると、ローカル環境でワークフローを実行できる。便利。

### [GitHub Actions: The Full Course - Learn by Doing!](https://www.youtube.com/playlist?list=PLArH6NjfKsUhvGHrpag7SuPumMzQRhUKY)
GithubActionsをハンズオン形式で全般的に解説した動画。

Github Actionsのチュートリアルだけでなく、CI/CDの解説も充実している。

英語なのが難点だが、第一回の動画は日本語字幕があるので大丈夫。

### [チートシート](https://zenn.dev/masaaania/articles/c930f2f755a577)

GithubActionsを用いてどのようなことができるかがざっくりと記載されている。

これら全てを今覚える必要は全くないが、一度目を通しておくと、今後の課題解決のアイデア創出に役立ちそう。



# 実践演習

## mainブランチへのマージの前にテストを実行して、<br/>失敗した場合マージできないように設定する

### STEP1. テストを実行するGithub Actionsを作成


```yml
# .github/workflows/ci.yaml
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


## OpenAPIのSPECファイルが更新されるたびに、コードジェネレータを実行して別のレポジトリにPR(プルリクエスト)を出す


### STEP1 適当なSPECファイルを用意する

今回は[openapi/spec.yml](/openapi/spec.yml)のようなファイルを用意しました。


### STEP2 PRを出す先のレポジトリを作成する

なんでも良いですが、今回は[github-actions-example-openapi-client-repository]( https://github.com/fujisw/github-actions-example-openapi-client-repository )という名前のレポジトリを作成しました。


### STEP3 権限を獲得する
今回は他のリポジトリに対してcloneやPRの作成をするため、普通のワークフローでは権限が足りない。

そのためワークフローに対して、何かしらの方法で権限を付与する必要がある。

主に２通りのやり方がある

1. Github Appを使用する
2. Private Access Tokenを発行する

|                       | PAT                                                                           | GitHub App                                               |
|-----------------------|-------------------------------------------------------------------------------|----------------------------------------------------------|
| メリット              | * 手軽に利用可能<br>* 必要な権限のみを与えられる<br>* APIでのアクセスが容易   | * セキュリティが高い<br>* アクセス許可を細かく設定できる | 
| デメリット            | * こまかいアクセス制御がしずらい                                              | * 導入への学習コストが高い                               |


#### Github Appを使用する場合

鋭意製作中

#### Private Access Tokenを使用する場合

1. `Setteings/Developer Settings`より`Fine-grained-tokens`の発行手続きを開始する
2. トークンの名前や権限などを設定する
    - 権限は必要最低限に設定する。
    - 今回の場合、`github-actions-example-openapi-client-repository`にアクセスできれば良いので、それだけ指定する
      ![](/assets/screenshot_repository_access.png)
3. `Permissions`を設定する
    - `Codespaces`: `Read and Write`
    - `Contents`: `Read and Write`
    - `Metadata`: `Readonly`
    - `Pull requests`: `Read and Write`
4. 発行したトークンをGithub Actionsが利用できるように登録
    - ![](/assets/screenshot_create_token.png)



### STEP4 ワークフローを作成する

```yml
# ./github/workflows/openapi_genereate.yaml

name: OpenAPI Client Generate

on:
  workflow_dispatch:
  pull_request:
    branches:
      - main
    types:
      - "closed"
    paths:
      - 'openapi/spec.yml'
  

jobs:
  generate-code:
    if: |
      github.event_name == 'workflow_dispatch' || github.event.pull_request.merged == true
    runs-on: ubuntu-latest
    permissions:
      pull-requests: write

    steps:
    - name: Checkout code
      uses: actions/checkout@v3

    - name: Checkout Client Respository
      uses: actions/checkout@v3
      with:
        token: ${{ secrets.CLIENT_TOKEN }}
        repository: fujisw/github-actions-example-openapi-client-repository
        path: tmp
        
    - name: Generate code
      uses: docker://openapitools/openapi-generator-cli
      with:
        args: generate -g typescript-axios -i openapi/spec.yml -o ./tmp/__generated__
    
    - name: Grant file permissions to prevent error on `Create Pull Request` step
      run: sudo chown -R $USER:$USER .

    - name: Create Pull Request
      uses: peter-evans/create-pull-request@v3.14.0
      with:
        token: ${{ secrets.CLIENT_TOKEN }}
        path: tmp
        commit-message: auto generated from ${{ github.event.pull_request.html_url }}
        title: _ ${{ github.event.pull_request.title }}
        branch: feature/codegen
        branch-suffix: short-commit-hash
```


### 解説


下のように書くと、mainブランチへのプルリクエストがマージされてクローズした場合のみ`generete-code`が実行される

```yml
on:
  pull_request:
    branches:
      - main
    types:
      - "closed"
    paths:
      - 'openapi/spec.yml'

jobs:
  generate-code:
    if: github.event.pull_request.merged == true
    .
    .
    .
```