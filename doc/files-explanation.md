# .ci/
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


# .github/
## stale.yml
github上の動作設定
https://probot.github.io/apps/stale/

古くなったissue等を自動的にCloseするといった設定を行っている

## ISSUE_TEMPLATE/
以下のissue作成時のテンプレート
github上でissueを作成しようとすると以下のテンプレートがUI上に現れます。

### 1_Bug_report.md
### 2_Feature_request.md
### 3_Support_question.md
### 4_Documentation_issue.md


## workflows/
github上で各種アクションがあったときに自動的に処理されるようにする設定群

https://docs.github.com/ja/free-pro-team@latest/actions/reference/workflow-syntax-for-github-actions

### checks.yml
### clang-tidy.yml
### ...割愛
### sitl_tests.yml

## .vscode/
Visual Studio Code用の設定

### c_cpp_properties.json
C/C++ Extension用設定ファイル
https://code.visualstudio.com/docs/cpp/c-cpp-properties-schema-reference

### cmake-kits.json
https://vector-of-bool.github.io/docs/vscode-cmake-tools/kits.html

### cmake-variants.yaml
https://vector-of-bool.github.io/docs/vscode-cmake-tools/variants.html

### extensions.json
VSCodeにインストール推奨するExtension
https://code.visualstudio.com/docs/editor/extension-gallery#_workspace-recommended-extensions

### settings.json
VSCodeや各種Extensionの設定

### tasks.json
VSCodeのタスク設定
https://code.visualstudio.com/docs/editor/tasks#vscode

# Documentation/
## Doxyfile.in
Doxygen (https://www.doxygen.nl/index.html) という、ソースコード(ソースコード上のコメント)からのドキュメント生成ツール用の設定

# ROMFS/
FMU(flight management unit)に書き込まれるファイルに関する設定のようだが詳細はWIP

# Tools/
様々な関連ツールの置き場
[詳細](doc/Tools.md)

# boards/
各種フライトコントローラ基盤毎の設定
