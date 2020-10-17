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
[詳細](./ROMFS.md)

# Tools/
様々な関連ツールの置き場
[詳細](./Tools.md)

# boards/
各種フライトコントローラ基盤毎の設定
[詳細](./boards.md)

# cmake/
cmake用の設定？
[詳細](./cmake.md)

# integrationtests/
他システムとの統合テスト用？
## python_src/
### px4_it/
#### dronekit/
#### mavros/
#### util/

# launch/
起動時のパラメータ設定？

# mavlink/
## include/
### mavlink/
謎(空っぽ)
外部へのリンクになっている？

# msg/
対象の`.msg`ファイルがあり、内部で変数が定義されている
[詳細](./msg.md)

## templates/
多数の`.em`ファイル

## tools/
各種モジュール間のメッセージング処理？

# platforms/
Nuttx, Posix, QurtのOS毎の設定？

# posix-configs/
posix系のosの設定？

# src/
PX4のソースコード群

# test/
各種テストコード群

# test_data/
テスト用のデータ
(csv的なtxtファイルが複数ある)

# validation/
## module_schema.yaml
PythonのバリデーションライブラリCerberus Validationの設定
https://docs.python-cerberus.org/en/stable/validation-rules.html

# .ackrc
grep的コマンドのack用設定
何のためか不明

# .clang-tidy
C言語のコードチェックツールClang-Tidyの設定
http://clang.llvm.org/extra/clang-tidy/index.html

# .gitattributes
ファイル拡張子毎の改行コードの指定
参考：
https://git-scm.com/docs/gitattributes

# .github_changelog_generator
ChangeLogの自動生成ツールの設定。使われてる？？
https://github.com/github-changelog-generator/github-changelog-generator

# .gitignore
gitignoreです。

# .gitmodules
外部のgitリポジトリを自身のgitのサブディレクトリとして扱う仕組み

# .travis.yml
travis ci用設定

# .ycm_extra_conf.py
vim用コード補間ツールYouCompleteMeの設定
https://github.com/ycm-core/YouCompleteMe

# CMakeLists.txt
CMakeLists.txt
https://cmake.org/cmake/help/latest/
[詳細](./CMakeLists.md)

# CODE_OF_CONDUCT.md
行動規範
[参照](./CODE_OF_CONDUCT.md)

# CONTRIBUTING.md
オープンソースへの貢献手順

# CTestConfig.cmake
WIP
テストのために必要な設定が書かれている模様
https://cmake.org/cmake/help/latest/module/CTest.html

# Firmware.sublime-project
エディタsublime用の設定

# Jenkinsfile
Jenkinsの設定

# LICENSE
BSD 3-Clause License

# Makefile
Makefile
[詳細](./Makefile.md)

# PULL_REQUEST_TEMPLATE.md
github上でプルリク時、自動的に差し込まれるテキスト

# README.md
README.md

# appveyor.yml
CIサービスAppVeyorの設定

# eclipse.cproject
IDEのEclipseのCDT (C/C++ Development Tooling)用設定

# eclipse.project
IDEのEclipse用設定

# package.xml
不明