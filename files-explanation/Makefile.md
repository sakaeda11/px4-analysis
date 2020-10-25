https://github.com/PX4/Firmware/blob/master/Makefile

# 前処理

## gitチェック
```
ifeq ($(wildcard .git),)
    $(error YOU HAVE TO USE GIT TO DOWNLOAD THIS REPOSITORY. ABORTING.)
endif
```

Makefileのerrorという関数を利用し、gitが利用されていなかった場合に処理を停止する

## make_list関数の定義
```
# define a space character to be able to explicitly find it in strings
space := $(subst ,, )

define make_list
     $(shell cat .github/workflows/compile_${1}.yml | sed -E 's|[[:space:]]+(.*),|check_\1|g' | grep check_${2})
endef
```

`.github/workflows/`にあるcompile_***.ymlファイルから、対象のリストを取り出す関数を定義している。

- 引数
  - 1：OS名
    - linux
    - macos
    - nuttx
  - 2:(option)
    - px4
    - nxp
https://github.com/PX4/Firmware/blob/master/.github/workflows/compile_${1}.yml
から`<何かしらの文字>`と`,`で終わっている行を"check_"というprefixをつけて抜き出している。

```
check_px4_fmu-v2_default
check_px4_fmu-v2_fixedwing
```
と言った文字列が抜き出される。

(ちなみに、sedコマンドのデリミタは`/`以外でもよく、`s`のあとに書いたものがデリミタになるらしい。この場合は`|`。[参考](https://qiita.com/takech9203/items/b96eff5773ce9d9cc9b3#%E3%83%87%E3%83%AA%E3%83%9F%E3%82%BF))


## パラメータ設定

```
FIRST_ARG := $(firstword $(MAKECMDGOALS))
ARGS := $(wordlist 2,$(words $(MAKECMDGOALS)),$(MAKECMDGOALS))
```
makeコマンド実行時に指定したgoalのリストから、最初のものを`FIRST_ARG`、それ移行(2番目から最後まで)のものを`ARGS`に格納する


## 同時実行数の取得

```
# Get -j or --jobs argument as suggested in:
# https://stackoverflow.com/a/33616144/8548472
MAKE_PID := $(shell echo $$PPID)
j := $(shell ps T | sed -n 's|.*$(MAKE_PID).*$(MAKE).* \(-j\|--jobs\) *\([0-9][0-9]*\).*|\2|p')

# Default j for clang-tidy
j_clang_tidy := $(or $(j),4)
```


makeコマンド実行時の同時実行数指定オプション`-j`(または`--jobs`)に指定した数を取得し、j_clang_tidyに指定している。
未指定の場合は4としている。

## ninjaの設定
高速なビルドシステムninja
https://ninja-build.org/
https://github.com/ninja-build/ninja

### ninjaのバージョンを設定

```
NINJA_BIN := ninja
ifndef NO_NINJA_BUILD
	NINJA_BUILD := $(shell $(NINJA_BIN) --version 2>/dev/null)

	ifndef NINJA_BUILD
		NINJA_BIN := ninja-build
		NINJA_BUILD := $(shell $(NINJA_BIN) --version 2>/dev/null)
	endif
endif
```

ちなみにmacならpx4-devをbrew installした際にninjaもインストールされているはず。
NINJA_BUILDにninjaのバージョンを設定している。

### makeコマンドのninjaへの差し替え

```
ifdef NINJA_BUILD
	PX4_CMAKE_GENERATOR := Ninja
	PX4_MAKE := $(NINJA_BIN)

	ifdef VERBOSE
		PX4_MAKE_ARGS := -v
	else
		PX4_MAKE_ARGS :=
	endif

	# Only override ninja default if -j is set.
	ifneq ($(j),)
		PX4_MAKE_ARGS := $(PX4_MAKE_ARGS) -j$(j)
	endif
else
	ifdef SYSTEMROOT
		# Windows
		PX4_CMAKE_GENERATOR := "MSYS\ Makefiles"
	else
		PX4_CMAKE_GENERATOR := "Unix\ Makefiles"
	endif

	# For non-ninja builds we default to -j4
	j := $(or $(j),4)
	PX4_MAKE = $(MAKE)
	PX4_MAKE_ARGS = -j$(j) --no-print-directory
endif
```

ninjaが有る場合は、PX4_MAKEをninjaで行うように指定する。
また、makeコマンドに渡されてきたパラメータ(VERBOSEと同時ジョブ数)をninjaに渡すようにコピーしている

## ソースディレクトリの取得
```
SRC_DIR := $(shell dirname "$(realpath $(lastword $(MAKEFILE_LIST)))")
```
makeコマンドを実行したディレクトリを取得して`SRC_DIR`に登録している


## replay
```
# check if replay env variable is set & set build dir accordingly
ifdef replay
	BUILD_DIR_SUFFIX := _replay
else
	BUILD_DIR_SUFFIX :=
endif
```
おそらくPX4内でのmake実行時、内部的にreplayという値が渡されて実行されることがあり、その場合は`BUILD_DIR_SUFFIX`に`_replay`と付与する

## cmakeに渡すその他のパラメータ設定
```
# additional config parameters passed to cmake
ifdef EXTERNAL_MODULES_LOCATION
	CMAKE_ARGS += -DEXTERNAL_MODULES_LOCATION:STRING=$(EXTERNAL_MODULES_LOCATION)
endif

ifdef PX4_CMAKE_BUILD_TYPE
	CMAKE_ARGS += -DCMAKE_BUILD_TYPE=${PX4_CMAKE_BUILD_TYPE}
else

	# Address Sanitizer
	ifdef PX4_ASAN
		CMAKE_ARGS += -DCMAKE_BUILD_TYPE=AddressSanitizer
	endif

	# Memory Sanitizer
	ifdef PX4_MSAN
		CMAKE_ARGS += -DCMAKE_BUILD_TYPE=MemorySanitizer
	endif

	# Thread Sanitizer
	ifdef PX4_TSAN
		CMAKE_ARGS += -DCMAKE_BUILD_TYPE=ThreadSanitizer
	endif

	# Undefined Behavior Sanitizer
	ifdef PX4_UBSAN
		CMAKE_ARGS += -DCMAKE_BUILD_TYPE=UndefinedBehaviorSanitizer
	endif

endif

# Pick up specific Python path if set
ifdef PYTHON_EXECUTABLE
	CMAKE_ARGS += -DPYTHON_EXECUTABLE=${PYTHON_EXECUTABLE}
endif
```

状況におうじてcmakeにわたすパラメータ`CMAKE_ARGS`の値を調整している

## cmake-build関数の定義
```
# describe how to build a cmake config
define cmake-build
	@$(eval BUILD_DIR = "$(SRC_DIR)/build/$(1)")
	@# check if the desired cmake configuration matches the cache then CMAKE_CACHE_CHECK stays empty
	@$(call cmake-cache-check)
	@# make sure to start from scratch when switching from GNU Make to Ninja
	@if [ $(PX4_CMAKE_GENERATOR) = "Ninja" ] && [ -e $(BUILD_DIR)/Makefile ]; then rm -rf $(BUILD_DIR); fi
	@# only excplicitly configure the first build, if cache file already exists the makefile will rerun cmake automatically if necessary
	@if [ ! -e $(BUILD_DIR)/CMakeCache.txt ] || [ $(CMAKE_CACHE_CHECK) ]; then \
		mkdir -p $(BUILD_DIR) \
		&& cd $(BUILD_DIR) \
		&& cmake "$(SRC_DIR)" -G"$(PX4_CMAKE_GENERATOR)" $(CMAKE_ARGS) \
		|| (rm -rf $(BUILD_DIR)); \
	fi
	@# run the build for the specified target
	@cmake --build $(BUILD_DIR) -- $(PX4_MAKE_ARGS) $(ARGS)
endef
```
cmakeを実行する
- 引数
  - 1:ターゲットのコンフィグ名
    - この指定の場所にビルド結果が生成される

## cmake-cache-check関数の定義
```
# check if the options we want to build with in CMAKE_ARGS match the ones which are already configured in the cache inside BUILD_DIR
define cmake-cache-check
	@# change to build folder which fails if it doesn't exist and CACHED_CMAKE_OPTIONS stays empty
	@# fetch all previously configured and cached options from the build folder and transform them into the OPTION=VALUE format without type (e.g. :BOOL)
	@$(eval CACHED_CMAKE_OPTIONS = $(shell cd $(BUILD_DIR) 2>/dev/null && cmake -L 2>/dev/null | sed -n 's|\([^[:blank:]]*\):[^[:blank:]]*\(=[^[:blank:]]*\)|\1\2|gp' ))
	@# transform the options in CMAKE_ARGS into the OPTION=VALUE format without -D
	@$(eval DESIRED_CMAKE_OPTIONS = $(shell echo $(CMAKE_ARGS) | sed -n 's|-D\([^[:blank:]]*=[^[:blank:]]*\)|\1|gp' ))
	@# find each currently desired option in the already cached ones making sure the complete configured string value is the same
	@$(eval VERIFIED_CMAKE_OPTIONS = $(foreach option,$(DESIRED_CMAKE_OPTIONS),$(strip $(findstring $(option)$(space),$(CACHED_CMAKE_OPTIONS)))))
	@# if the complete list of desired options is found in the list of verified options we don't need to reconfigure and CMAKE_CACHE_CHECK stays empty
	@$(eval CMAKE_CACHE_CHECK = $(if $(findstring $(DESIRED_CMAKE_OPTIONS),$(VERIFIED_CMAKE_OPTIONS)),,y))
endef
```

上述の`cmake-build`の中で利用される。
`CMAKE_ARGS`の指定がすでにキャッシュに無いかをチェックして、ない場合は`CMAKE_CACHE_CHECK`を`y`とする。

`cmake-build`ではキャッシュが無い場合は初期化処理(`build`ディレクトリの作成など)を行う


## colorecho関数の定義
```
COLOR_BLUE = \033[0;94m
NO_COLOR   = \033[m

define colorecho
+@echo -e '${COLOR_BLUE}${1} ${NO_COLOR}'
endef
```
`colorecho`に渡した文字列を、背景色黒&文字色青で、表示することが出来る関数です。

補足：echoでは`\033`(8進数、ascii文字でESCを示す)と`[`のあとに文字色変更の指示などを行える。
参考：
http://site.m-bsys.com/linux/echo-color-1
http://linuxjm.osdn.jp/html/LDP_man-pages/man4/console_codes.4.html
https://stackoverflow.com/questions/5947742/how-to-change-the-output-color-of-echo-in-linux/28938235#28938235

## 全てのConfigリスト化
```
# Get a list of all config targets boards/*/*.cmake
ALL_CONFIG_TARGETS := $(shell find boards -maxdepth 3 -mindepth 3 ! -name '*common*' ! -name '*sdflight*' -name '*.cmake' -print | sed -e 's|boards\/||' | sed -e 's|\.cmake||' | sed -e 's|\/|_|g' | sort)
```

`boards/*/*.cmake`にある全てのコンフィグをリスト化し、`ALL_CONFIG_TARGETS`に入れている

# Makefileのターゲット群

## CONFIG TARGET
### ALL_CONFIG_TARGETS
```
$(ALL_CONFIG_TARGETS):
	@$(eval PX4_CONFIG = $@)
	@$(eval CMAKE_ARGS += -DCONFIG=$(PX4_CONFIG))
	@$(call cmake-build,$(PX4_CONFIG)$(BUILD_DIR_SUFFIX))
```

ALL_CONFIG_TARGETSには
```
x4_sitl_default
px4_sitl_nolockstep
px4_sitl_replay
px4_sitl_rtps
px4_sitl_test
```
などのターゲットのリストが入っているので
`make px4_sitl_nolockstep`
などと実行すると、これがターゲットとして選ばれる。
`$@`はターゲット名になるので`PX4_CONFIG`はターゲット名=コンフィグ名になる。

そして、このコンフィグ名を元にcmake-build関数を呼び出し実行される
なお、`_replay`というサフィックスが付与されることがある

### デフォルトコンフィグ
```
# Filter for only default targets to allow omiting the "_default" postfix
CONFIG_TARGETS_DEFAULT := $(patsubst %_default,%,$(filter %_default,$(ALL_CONFIG_TARGETS)))
$(CONFIG_TARGETS_DEFAULT):
	@$(eval PX4_CONFIG = $@_default)
	@$(eval CMAKE_ARGS += -DCONFIG=$(PX4_CONFIG))
	@$(call cmake-build,$(PX4_CONFIG)$(BUILD_DIR_SUFFIX))
```

`ALL_CONFIG_TARGETS`の内、末尾が`_default`で終わっているもののみをcmakeする。

### 上記の全実行
```
all_config_targets: $(ALL_CONFIG_TARGETS)
all_default_targets: $(CONFIG_TARGETS_DEFAULT)
```
の`all_config_targets`または`all_default_targets`をターゲットとして指定することで、該当するものを全てmake対象とする


## deprecation_warning関数の定義
```
# board reorganization deprecation warnings (2018-11-22)
define deprecation_warning
	$(warning $(1) has been deprecated and will be removed, please use $(2)!)
endef
```
廃止予定のものを呼んだときに表示するようの関数定義
(今今は使っていない？)

## ターゲット間の依存関係の指定
```
# All targets with just dependencies but no recipe must either be marked as phony (or have the special @: as recipe).
.PHONY: all px4_sitl_default all_config_targets all_default_targets
```

# (一部省略)
```
# Multi- config targets.
eagle_default: atlflight_eagle_default atlflight_eagle_qurt
eagle_rtps: atlflight_eagle_rtps atlflight_eagle_qurt-rtps

excelsior_default: atlflight_excelsior_default atlflight_excelsior_qurt
excelsior_rtps: atlflight_excelsior_rtps atlflight_excelsior_qurt-rtps

.PHONY: eagle_default eagle_rtps
.PHONY: excelsior_default excelsior_rtps

# Other targets
# --------------------------------------------------------------------

.PHONY: qgc_firmware px4fmu_firmware misc_qgc_extra_firmware check_rtps

# QGroundControl flashable NuttX firmware
qgc_firmware: px4fmu_firmware misc_qgc_extra_firmware

# px4fmu NuttX firmware
px4fmu_firmware: \
	check_px4_io-v2_default \
	check_px4_fmu-v2_default \
	check_px4_fmu-v3_default \
	check_px4_fmu-v4_default \
	check_px4_fmu-v4pro_default \
	check_px4_fmu-v5_default \
	check_px4_fmu-v5x_default \
	sizes

misc_qgc_extra_firmware: \
	check_nxp_fmuk66-v3_default \
	check_nxp_fmurt1062-v1_default \
	check_intel_aerofc-v1_default \
	check_mro_x21_default \
	check_bitcraze_crazyflie_default \
	check_airmind_mindpx-v2_default \
	check_px4_fmu-v2_lpe \
	sizes

# builds with RTPS
check_rtps: \
	check_px4_fmu-v3_rtps \
	check_px4_fmu-v4_rtps \
	check_px4_fmu-v4pro_rtps \
	check_px4_sitl_rtps \
	sizes

.PHONY: sizes check quick_check check_rtps uorb_graphs

sizes:
	@-find build -name *.elf -type f | xargs size 2> /dev/null || :

# All default targets that don't require a special build environment
check: check_px4_sitl_default px4fmu_firmware misc_qgc_extra_firmware tests check_format

# quick_check builds a single nuttx and SITL target, runs testing, and checks the style
quick_check: check_px4_sitl_test check_px4_fmu-v5_default tests check_format
```
この部分の説明は省略

# check_***ターゲット
```
check_%:
	@echo
	$(call colorecho,'Building' $(subst check_,,$@))
	@$(MAKE) --no-print-directory $(subst check_,,$@)
	@echo
```

check_***というターゲットが指定された場合これがターゲットとして再度`make`実行される。
$(MAKE)という記述の仕方により、コマンドラインで指定したオプションなども引き継いで実行される


# uorb_graphsターゲット
```
uorb_graphs:
	@./Tools/uorb_graph/create_from_startupscript.sh
	@./Tools/uorb_graph/create.py --src-path src --exclude-path src/examples --file Tools/uorb_graph/graph_full
	@$(MAKE) --no-print-directory px4_fmu-v2_default uorb_graph
	@$(MAKE) --no-print-directory px4_fmu-v4_default uorb_graph
	@$(MAKE) --no-print-directory px4_sitl_default uorb_graph
```

# ドキュメント生成
```
# Documentation
# --------------------------------------------------------------------
.PHONY: parameters_metadata airframe_metadata module_documentation px4_metadata doxygen

parameters_metadata:
	@$(MAKE) --no-print-directory px4_sitl_default metadata_parameters

airframe_metadata:
	@$(MAKE) --no-print-directory px4_sitl_default metadata_airframes

module_documentation:
	@$(MAKE) --no-print-directory px4_sitl_default metadata_module_documentation

px4_metadata: parameters_metadata airframe_metadata module_documentation

doxygen:
	@mkdir -p "$(SRC_DIR)"/build/doxygen
	@cd "$(SRC_DIR)"/build/doxygen && cmake "$(SRC_DIR)" $(CMAKE_ARGS) -G"$(PX4_CMAKE_GENERATOR)" -DCONFIG=px4_sitl_default -DBUILD_DOXYGEN=ON
	@$(PX4_MAKE) -C "$(SRC_DIR)"/build/doxygen
	@touch "$(SRC_DIR)"/build/doxygen/Documentation/.nojekyll
```