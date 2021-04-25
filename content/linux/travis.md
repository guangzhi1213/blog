---
title: "Travis"
date: 2021-04-25T10:40:53+08:00
lastmod: 2021-04-25T10:40:53+08:00
draft: false
keywords: []
description: ""
tags: ["linux","sre","ops"]
categories: ["linux"]
author: "王清"

# You can also close(false) or open(true) something for this content.
# P.S. comment can only be closed
comment: false
toc: true
autoCollapseToc: false
postMetaInFooter: false
hiddenFromHomePage: false
# You can also define another contentCopyright. e.g. contentCopyright: "This is another copyright."
contentCopyright: false
reward: false
mathjax: false
mathjaxEnableSingleDollar: false
mathjaxEnableAutoNumber: false

# You unlisted posts you might want not want the header or footer to show
hideHeaderAndFooter: false

# You can enable or disable out-of-date content warning for individual post.
# Comment this out to use the global config.
#enableOutdatedInfoWarning: false

flowchartDiagrams:
  enable: false
  options: ""

sequenceDiagrams: 
  enable: false
  options: ""

---

# github travis 配置

```shell
language: go

go:
- tip
install:
- go get github.com/Kisesy/gscan_quic

before_install:
- sudo apt-get install p7zip-full

script:
- git clone https://github.com/Kisesy/gscan_quic.git
- cd gscan_quic/

- export EXE=gscan_quic
- export TAR_EXCLUDE="--exclude=*.go --exclude=*.bz2 --exclude=*.7z --exclude=vendor"

- export GOOS=windows GOARCH=386 && go build -ldflags "-w -s"
- 7za a ${EXE}_${GOOS}_${GOARCH}-${TRAVIS_TAG}.7z -ms=on -mx=9 -r- -mmt -x'!*.go' -x'!*.7z' -x'!*.zip' -x'!vendor/' *
- rm *.exe
- export GOOS=windows GOARCH=amd64 && go build -ldflags "-w -s"
- 7za a ${EXE}_${GOOS}_${GOARCH}-${TRAVIS_TAG}.7z -ms=on -mx=9 -r- -mmt -x'!*.go' -x'!*.7z' -x'!*.zip' -x'!vendor/' *
- rm *.exe

- export GOOS=linux GOARCH=386 && go build -ldflags "-w -s"
- tar cjf ${EXE}_${GOOS}_${GOARCH}-${TRAVIS_TAG}.tar.bz2 * ${TAR_EXCLUDE}
- export GOOS=linux GOARCH=amd64 && go build -ldflags "-w -s"
- tar cjf ${EXE}_${GOOS}_${GOARCH}-${TRAVIS_TAG}.tar.bz2 * ${TAR_EXCLUDE}
- export GOOS=linux GOARCH=arm && go build -ldflags "-w -s"
- tar cjf ${EXE}_${GOOS}_${GOARCH}-${TRAVIS_TAG}.tar.bz2 * ${TAR_EXCLUDE}
- export GOOS=darwin GOARCH=amd64 && go build -ldflags "-w -s"
- tar cjf ${EXE}_${GOOS}_${GOARCH}-${TRAVIS_TAG}.tar.bz2 * ${TAR_EXCLUDE}

- sha1sum *.bz2
- sha1sum *.7z

deploy:
  provider: releases
  api_key:
    secure: ${TOKEN}
  file_glob: true
  file:
    - "*.7z"
    - "*.bz2"

  skip_cleanup: true
  on:
    tags: true
```