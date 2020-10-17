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