# github-actions-example


## mainブランチへのマージの前にテストを実行して、失敗した場合マージを拒否する


`.github/workflows`に下のようなワークフローを用意する

```yml
name: Continuous Integration

on:
  pull_request:
    branches:
      - main

jobs:
  build-and-test:
    runs-on: ubuntu-latest

    strategy:
      matrix:
        node-version: [18.x]

    steps:
    - name: Checkout code
      uses: actions/checkout@v2

    - name: Set up Node.js
      uses: actions/setup-node@v2
      with:
        node-version: ${{ matrix.node-version }}

    - name: Install dependencies
      working-directory: frontend
      run: npm ci

    - name: Run tests
      working-directory: frontend
      run: npm test -- --coverage --watchAll=false
```

Githubのブランチ保護のルールを記載

![](/assets/スクリーンショット_ブランチ保護.png)