---
layout: post
title: GDSC(1) - git & shell script
date: 2023-04-19 14:58 +0900
categories: [GDSC]
tags: [gdsc yonsei, git, shell script]
render_with_liquid: false
---

`GDSC (Google Developer Student Club) Yonsei` 주간 세션으로 주어진 키워드를 정리하는 포스트이다. 이번 주차는 **git**과 **shell script**에 대한 내용을 다룬다.

<br>

## git

<br>

개발자라면 사용하고 있을 git 은 분산 버전 관리 시스템(VCS)으로, Linux를 개발한 리누스 토발즈가 기존 버전 관리 시스템이 마음에 들지 않아 2주만에(...) 만든 시스템이다. 한 프로그램을 개발하는데 있어 여러 개발자들이 협업할 때, 각자의 개발 내용을 하나로 합치는 것을 간편하게 할 수 있으며 프로그램의 버전을 관리하는데 용이하다. <span style="color:gray">_(버전 관리 시스템을 사용하기 이전 리누스 토발즈는 Linux를 개발하는데 다른 사람들의 기여 내용을 메일로 받아 일일이 합쳤다고 한다.)_</span>

git의 파일 상태는 **Untracked, Unmodified, Modified, Staged**가 있다.

<br>

![Desktop View](/assets/img/posts/git_lifecycle.png){: w="700" h="400" }

git 로컬 저장소에 파일을 처음 생성하면 git에 추적되지 않은 파일이라는 의미로 Untracked 상태로 시작한다. 이후 add, commit 등의 동작에 따라 파일의 상태가 달라지게 된다. Track 되고 있는 상태의 파일이 수정되었을 경우 Modified 상태가 된다. 이후 소개할 명령어의 대부분은 옵션을 부여해 추가적인 동작을 수행할 수 있다. (레퍼런스 참고)

### git add

Untracked file이 생성되거나, Unmodified file이 수정되어 Modified가 되면 이를 stage에 올릴 수 있다. 파일 상태를 Staged로 바꾸는 동작이다. 파일이 staged 상태가 되어야 현재 브랜치에 변경 사항을 추가하는 commit 동작을 수행할 수 있다. 다음과 같은 명령어로 실행한다:

```terminal
$ git add <filename>
```

{: .nolineno}

### git commit

Stage된 내용을 브랜치에 반영하는 동작이다. 다음과 같은 명령어로 실행한다:

```terminal
$ git commit -m "<commit message>"
```

{: .nolineno}

### git push

로컬에서 작업하는 저장소인 로컬 저장소 브랜치의 변경사항을 협업 저장소인 원격 저장소 브랜치에 붙이는 명령어이다:

```terminal
$ git push <remote_name> <branch_name>
```

{: .nolineno}

### git remote

git remote 명령어는 옵션별로 다음과 같은 동작을 수행한다.

로컬 저장소에 연결되어있는 원격 저장소를 확인하는 명령어이다:

```terminal
$ git remote -v
```

{: .nolineno}

로컬 저장소를 원격 저장소에 연결하는 명령어이다:

```terminal
$ git remote add <name> <url>
```

{: .nolineno}

저장소 연결을 끊는 명령어이다:

```terminal
$ git remote remove <name>
```

{: .nolineno}

### git hook

hook 이라는 이름에서 알 수 있듯, 어떤 이벤트가 생겼을 때 자동으로 특정 스크립트를 실행할 수 있게 하는 도구이다.

git init을 통해 git 저장소를 초기화하게 되면 .git 폴더가 생성된다. 개발 과정에서의 수많은 커밋과 브랜치, 관리되는 파일의 정보들이 모두 .git 안에서 관리되며, .git/hooks 폴더에 있는 파일들을 수정하여 각 git 이벤트 발생 시 실행되는 스크립트를 변경할 수 있다.

#### pre-commit

커밋할 때 가장 먼저 실행되는 훅으로, 코드 스타일 검사(lint 등), 공백 문자 검사, 주석 검사를 한다.

#### pre-push

push 작업이 곧바로 진행되는 것을 방지한다. 특정 브랜치로의 의도치 않은 수동 push 를 방지하거나, 미리 설정된 검사 (단위테스트나 문법 검사 등) 들이 실패한 경우 push 를 막는 경우 등이 있을 수 있다.

<br>

## Shell script

<br>

Shell script는 Shell에서 돌아가도록 작성되었거나 한 운영체제를 위해 쓰인 스크립트이다. Shell script가 수행하는 기능으로는 파일 이용, 프로그램 실행, 문자열 출력 등이 있다. 이 섹션에서는 Shell의 종류, 초기화 파일, 간단한 명령어, 편집기 등을 살펴본다.

### Shell의 종류

여러가지가 있지만, 많이 쓰이는 종류만 정리한다.

- Bourne Shell (sh)<br>
  유닉스 7버전의 기본 Shell

- Bourne Again Shell (bash)<br>
  본쉘을 기반으로 만들어졌으며, 현재 리눅스의 표준 Shell

- Z Shell (zsh)<br>
  본쉘의 확장된 버전으로, 다양한 기능, 플러그인, 테마가 존재<br>
  현 MacOS의 기본 Shell이며, 필자는 MacOS를 사용하므로 zsh를 기준으로 스크립트를 작성할 예정

### Shell의 초기화 파일

Shell의 초기화 설정 파일이 있으며, 하나의 Shell에 여러개의 파일이 존재한다.

bash에는 다음과 같은 초기화 파일이 있으며 번호는 실행 순서이다:

1. `/etc/profile`{: .filepath}
2. `~/.bash_profile or ~/.profile`{: .filepath}
3. `~/.bashrc`{: .filepath}
4. `/etc/bashrc`{: .filepath}

1,4 파일은 모든 User에게 적용되며, 2,3 파일은 현재 User에게 적용된다.

2,3 파일의 특성을 다음과 같다:

- `.profile` : 로그인 되는 시점에 실행, bash와 상관 없는 것들을 넣음
- `.bash_profile` : bash로 로그인 되는 시점에 실행
- `.bashrc` : 새로운 콘솔을 열 때 실행 (로그인과 무관)

zsh에서는 `.zshrc`가 동일한 역할을 수행한다.

### bash/zsh 기본 명령어

- **ls**<br>
  파일 목록 출력

- **pwd**<br>
  현재 디렉토리 위치<br>

- **cd**<br>
  디렉토리 이동

  ```terminal
  $ cd <directory>
  ```

  {: .nolineno}

- **sudo**<br>
  관리자 권한 실행

  ```terminal
  $ sudo <command>
  ```

  {: .nolineno}

- **chmod**<br>
  파일 권한 변경

  ```terminal
  $ chmod [option] <permission-8bit> <filename>
  or
  $ chmod [option] <permission-symbolic> <filname>
  ```

  {: .nolineno}

  - chmod option

    `-R` : 하위 디렉토리 모든 권한 변경

    `-c` : 권한 변경 파일내용 출력

  - chmod symbolic

    대상

    `u` : user

    `g` : group

    `o` : other

    `a` : all

    <br>

    +/-/=

    `+` : 해당 권한 추가

    `-` : 해당 권한 제거

    `=` : 해당 권한을 설정한대로 변경

    <br>

    rwx

    `r` : read

    `w` : write

    `x` : execute

- **chown**<br>
  파일과 그룹의 소유권을 변경하는 명령어

  ```terminal
  $ chown [option] <username:groupname> <filename>
  ```

  {: .nolineno}

  - chown option

    `-R` : 하위 디렉토리에도 모든 권한 변경

  - \<username:groupname>

    \<username> : 소유자만 변경

    \<:groupname> : 그룹만 변경

    \<username:> : 소유자와 그룹 모두 동일한 것으로 변경

    \<username:groupname> : 소유자와 그룹을 각각 변경

- **ln**<br>
  파일 간 링크를 만드는 명령어

  ```terminal
  $ ln [option] <file> <link>
  ```

  {: .nolineno}

  symbolic link를 만들기 위해서는 option으로 -s를 주면 된다.

- **source**<br>
  Shell script 파일을 실행하는 명령어

  ```terminal
  $ source <filename>
  ```

  {: .nolineno}

- **Shell 명령행 편집 기능**<br>
  Shell의 명령어는 아니지만, Shell을 사용할 때 유용하게 사용할 수 있는 단축키이다. (Mac에서도 동일)

  - Ctrl + a : 가장 왼쪽으로 이동
  - Ctrl + e : 가장 오른쪽으로 이동
  - Ctrl + k : 커서 오른쪽 행 전체 삭제
  - Ctrl + u : 행 전체 삭제
  - Ctrl + y : 삭제 취소

### vi, vim

**vi(visual editer)**는 파일의 내용을 보여줌과 동시에 파일의 내용을 편집할 수 있게 해주는 프로그램(텍스트 에디터)이다.
**vim(vi improved)**는 vi의 업그레이드 버전으로, vi와 달리 설정 파일의 특정 위치에 있는 단어나 의미있는 단어를 다른 색으로 표시해준다. alias 기능을 활용하여 vi로 이름을 변경하여 실행하므로 vi와 따로 구분을 하지 않는 편.

CLI 환경만 사용할 수 있다면 어쩔 수 없이 사용해야 하는 편집기이지만, GUI 환경에서는 code 명령어를 활용하면 vscode로 파일을 열 수 있으니 vscode를 적극적으로 이용하도록 하자.
