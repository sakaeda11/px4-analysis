# .ci
## Jenkinsfile-compile
TODO

## Jenkinsfile-hardware
TODO

# .devcontainer
VS Code Remote Development用の設定

参照：
https://code.visualstudio.com/docs/remote/remote-overview

ローカルのVSCodeからリモートにあるコンテナやサーバの環境にアクセスして実装出来、プログラムの実行環境としてもリモートにあるコンテナやサーバの環境を利用出来る仕組み。

## devcontainer.json
VS Code Remote Developmentの設定ファイル


# .github

## stale.yml
github上の動作設定
https://probot.github.io/apps/stale/

古くなったissue等を自動的にCloseするといった設定を行っている

## ISSUE_TEMPLATE
以下のissue作成時のテンプレート
github上でissueを作成しようとすると以下のテンプレートがUI上に現れます。

### 1_Bug_report.md
### 2_Feature_request.md
### 3_Support_question.md
### 4_Documentation_issue.md


## workflows
github上で各種アクションがあったときに自動的に処理されるようにする設定群

https://docs.github.com/ja/free-pro-team@latest/actions/reference/workflow-syntax-for-github-actions

### checks.yml
### clang-tidy.yml
### ...割愛
### sitl_tests.yml

## .vscode
Visual Studio Code用の設定

### 