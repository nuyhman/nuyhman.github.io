---
title: "[Git] 커밋 메시지 자동으로 넣어주기"
last_modified_at: 2024-04-27
tags:
  - git
  - husky
  - prepare-commit-msg
toc: true
toc_sticky: true
---

<!--
Default	.notice
Primary	.notice--primary
Info	.notice--info
Warning	.notice--warning
Success	.notice--success
Danger	.notice--danger
-->

## 개요

프로젝트에서 이슈 추적을 용이하기 위해 사용하고 있는 커밋 양식이 있는데 매번 직접 입력해주기 번거로워 자동화 하고자 한다.
자동으로 커밋에 포함할 내용은 다음과 같다.

- Staged files
- Jira issue

## 적용 방법

커밋 메시지에 자동으로 원하는 양식을 추가하기 위해선 git hook 스크립트 중 하나인 `prepare-commit-msg` 를 사용하여 넣어줄 수 있다.

- 프로젝트 내부에 공유하기 위해 git hook 라이브러리인 [husky](https://typicode.github.io/husky/) 설치

  ```bash
  npm install --save-dev husky
  or
  yarn add --dev husky
  ```

- `prepare-commit-msg` 파일 생성

  ```bash
  npx husky init
  ```

  위 명령을 실행하면 `.husky/pre-commit` 파일이 생성되는데 이 파일은 커밋이 수행되기 전에 실행되는 파일이기 때문에 커밋메시지가 작성되기 전에 수행하는 스크립트인 `prepare-commit-msg` 파일을 생성해준다.

- 다른 팀원들에게 설정을 공유하기 위한 script 설정

  ```bash
  // package.json
  "scripts": {
    "prepare": "husky install"
  }
  ```

  이를 통해 `npm install` 또는 `yarn`을 실행할 때 husky로 설정한 git hook이 공유된다.
  최신 버전에선 자동으로 추가되지만, 그렇지 않은 경우 직접 추가하면 된다.

- 커밋 양식을 추가하기 위한 스크립트 작성

  ```bash
  #!/bin/bash

  # 커밋 메시지 파일의 경로
  MESSAGE_FILE=$1

  # 현재 브랜치 이름 가져오기
  BRANCH_NAME=$(git rev-parse --abbrev-ref HEAD)

  # 스테이징된 파일 목록과 상태 가져오기
  FILES=$(git diff --cached --name-status | awk '{
      if ($1 == "M") print "modified:   " $2;
      else if ($1 == "A") print "new file:   " $2;
      else if ($1 == "C") print "copied:     " $2;
      else if ($1 == "R") print "renamed:    " $2;
      else if ($1 == "U") print "unmerged:   " $2;
      else if ($1 == "D") print "deleted:    " $2;
      else print $0
    }'
  )

  # 파일 목록과 상태를 커밋 메시지에 추가
  echo "" >> $MESSAGE_FILE
  echo "$FILES" >> $MESSAGE_FILE

  # 현재 브랜치 이름을 커밋 메시지에 추가
  echo "" >> $MESSAGE_FILE
  echo "issue: $BRANCH_NAME" >> $MESSAGE_FILE
  ```

  위 스크립트를 `.husky/prepare-commit-msg` 파일에 작성하면 된다. 스테이징된 파일 목록의 상태가 M, A, C, R, U, D 이런식으로 오는데 읽기 편한 메시지로 변경하기 위해 `git status`로 확인할 때 볼 수 있는 문자들로 매칭해주었고, 이슈 브랜치를 담아주기 위한 현재 브랜치를 가져와서 커밋 메시지에 추가해주었다.

## 결과

> git commit -m "test: git hook husky"

<figure  class="align-center">
   <img src="/assets/images/git-pre-commit-msg-result.png" />
   <figcaption>변경 파일과 브랜치가 자동으로 담기는 걸 볼 수 있다.</figcaption>
 </figure>
