# cmake/
各所のCMakeList.txtで呼ばれる関数を定義している

# gtest/
GoogleのGoogleTestというモジュールを利用したテスト
https://github.com/google/googletest/

## CMakeLists.txt.in
GoogleTestの読み込みなどの基本設定

## gtest.cmake
CMakeLists.txt.inを元にGoogleTestをダウンロードして設定する

## px4_add_gtest.cmake
`px4_add_unit_gtest`と`px4_add_functional_gtest`というCMake用関数を定義

# bloaty.cmake
bloatyというプログラムを利用したcmakeのターゲットを設定している
bloatyについては詳細不明。
おそらく
https://github.com/google/bloaty
で、各種ファイルサイズなどを算出するツール

# ccache.cmake
ccacheというコマンドのパラメータ設定

# coverage.cmake