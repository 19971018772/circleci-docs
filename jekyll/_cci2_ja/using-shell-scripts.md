---
layout: classic-docs
title: "シェル スクリプトの使用"
short-title: "シェル スクリプトの使用"
description: "CircleCI 設定ファイルでのシェル スクリプト使用に関するベスト プラクティス"
categories:
  - はじめよう
contentTags:
  platform:
    - クラウド
    - Server v4.x
    - Server v3.x
    - Server v2.x
---

## 概要
{: #overview }

CircleCI の設定では、シェルスクリプトの記述が必要になることは少なくありません。 While shell scripting can give you finer control over your build, it is possible you will come across a few errors. 以下に説明するベストプラクティスを参照すれば、これらのエラーの多くを回避することができます。

## シェルスクリプトのベストプラクティス
{: #shell-script-best-practices }

### ShellCheck の使用
{: #use-shellcheck }

[ShellCheck](https://github.com/koalaman/shellcheck) は、シェル スクリプトの静的解析ツールです。bash/sh シェル スクリプトに対して警告と提案を行います。

ShellCheck を `version: 2.1` の設定に追加するには、[ShellCheck Orb](https://circleci.com/ja/developer/orbs/orb/circleci/shellcheck) の使用が最も簡単な方法です ( 必ず `x.y.z` を有効なバージョンに変更してください)。

```yaml
version: 2.1

orbs:
  shellcheck: circleci/shellcheck@x.y.z

workflows:
  check-build:
    jobs:
      - shellcheck/check # job defined within the orb so no further config necessary
      - build-job:
          requires:
            - shellcheck/check # only run build-job once shellcheck has run
          filters:
            branches:
              only: main # only run build-job on main branch

jobs:
  build-job:
    ...
```

または、version 2 の設定をご使用の場合は、 Orb を使わなくても ShellCheck を設定できます。

```yaml
version: 2
jobs:
  shellcheck:
    docker:
      - image: koalaman/shellcheck-alpine:stable
        auth:
          username: mydockerhub-user
          password: $DOCKERHUB_PASSWORD  # context / project UI env-var reference
    steps:
      - checkout
      - run:
          name: Check Scripts
          command: |
            find . -type f -name '*.sh' | wc -l
            find . -type f -name '*.sh' | xargs shellcheck --external-sources
  build-job:
    ...

workflows:
  version: 2
  check-build:
    jobs:
      - shellcheck
      - build-job:
          requires:
            - shellcheck # only run build-job once shellcheck has run
          filters:
            branches:
              only: main # only run build-job on main branch
```

Take caution when using `set -o xtrace` / `set -x` with ShellCheck. When the shell expands secret environment variables, they will be exposed in a not-so-secret way, as in the example below.
{: class="alert alert-info" }

As cautioned above, observe how the `tmp.sh` script file reveals too much.

```shell
> cat tmp.sh
#!/bin/sh

set -o nounset
set -o errexit
set -o xtrace

if [ -z "${SECRET_ENV_VAR:-}" ]; then
  echo "You must set SECRET_ENV_VAR!" fi
> sh tmp.sh
+ '[' -z '' ']'
+ echo 'You must set SECRET_ENV_VAR!'
You must set SECRET_ENV_VAR!
> SECRET_ENV_VAR='s3cr3t!' sh tmp.sh
+ '[' -z 's3cr3t!' ']'
```

### エラーフラグの設定
{: #set-error-flags }

いくつかのエラーフラグを設定することで、好ましくない状況が発生した場合にスクリプトを自動的に終了できます。 厄介なエラーを回避するために、各スクリプトの先頭に以下のフラグを追加することをお勧めします。

```shell
#!/usr/bin/env bash

# Exit script if you try to use an uninitialized variable.
set -o nounset

# Exit script if a statement returns a non-true return value.
set -o errexit

# Use the error status of the first failure, rather than that of the last item in a pipeline.
set -o pipefail
```

## Run a shell script
{: #run-a-shell-script }

In your terminal, navigate to the folder/location of the script you want to run. You can use `ls` to verify you have navigated to the correct path for the script. You should now be able to run the following in your terminal:

```bash
sh <name-of-file>.sh
```

Occassionally, a script might not be executable by default, and you will be required to make the file executable before you run it. This process differs per platform, and you will need to search how to do this for your specific platform. For example, you can try to right-click on the script file and see if there is an option to make it executable. If you are on macOS or Linux, you can also look up how to use `chmod` commands to make a script file executable with different permissions.

## 関連リソース
{: #additional-resources }
{:.no_toc}

For more detailed explanations and additional techniques, see this [Writing Robust Bash Shell Scripts](https://www.davidpashley.com/articles/writing-robust-shell-scripts) blog post on writing robust shell scripts.
