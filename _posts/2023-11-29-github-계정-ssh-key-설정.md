---
layout: post
title: github 계정 ssh key 설정
date: '2023-11-29 16:14:42 +0900'
category: [Github, SSH]
tags: [github, ssh]
---

# ssh key 생성 및 github 계정에 등록
[npm 패키지 제작기 2](/posts/npm-패키지-제작기-2/) 포스트에서 언급한 대로, github에 사용중인 ssh key 에 중복문제가 발생했다.

그냥 새로운 키를 만들고 계정에 등록해서 사용하자.

> **ssh key 가 사용되고 있는 위치 확인**
>
> ```bash
> ssh -T git@github.com # github 연결 확인
> ssh -T -ai ~/.ssh/id_rsa git@github.com # 특정 key 연결 확인
> ```
> 명령어를 통해, key가 사용되고 있는 github 계정 / 레포를 확인할 수 있다.
>
> 출처 : <https://docs.github.com/en/authentication/troubleshooting-ssh/error-key-already-in-use>{:target="_blank"}
{: .prompt-tip }

---

## SSH Key 생성
도큐먼트 참조 : <https://docs.github.com/ko/authentication/connecting-to-github-with-ssh/generating-a-new-ssh-key-and-adding-it-to-the-ssh-agent>{: target="_blank"}
1. ed25519 알고리즘을 사용한 Key를 생성한다.
    ```bash
    ssh-keygen -t ed25519 -C "your_email@example.com"
    ```
    - 생성 도중 passphrase 설정을 하면, 2차 보안까지 설정 가능하다.
2. 생성된 private key와 public key를 ~/.ssh/ 경로로 옮겨준다.(혹은 생성할 때 경로까지 지정하면 됨)

---

## ssh-agent에 SSH Key 등록
ssh-agent[^fn1] 에 생성한 ssh key를 추가하자.

```bash
ssh-add privateKey경로
```

---

## Windows용 Git에서 ssh-agent 자동 시작
내가 사용하는 VSCode의 기본 터미널을 `git bash`로 설정해놓았기 때문에, 이런 `git bash`를 열 때 마다 `ssh-agent`를 자동실행되게 설정할 수 있다.

아래의 코드블록을 복사해 `~/.profile` 혹은 `~/.bashrc` 에 붙여넣자.

```bash
env=~/.ssh/agent.env

agent_load_env () { test -f "$env" && . "$env" >| /dev/null ; }

agent_start () {
    (umask 077; ssh-agent >| "$env")
    . "$env" >| /dev/null ; }

agent_load_env

# agent_run_state: 0=agent running w/ key; 1=agent w/o key; 2=agent not running
agent_run_state=$(ssh-add -l >| /dev/null 2>&1; echo $?)

if [ ! "$SSH_AUTH_SOCK" ] || [ $agent_run_state = 2 ]; then
    agent_start
    ssh-add
elif [ "$SSH_AUTH_SOCK" ] && [ $agent_run_state = 1 ]; then
    ssh-add
fi

unset env
```

이제 `git bash`를 켤 때 마다 자동 실행된다.

---

## Github 계정에 등록
github repo가 아닌 계정 설정에 들어가서 key를 등록하자.

Github 계정의 [Settings](https://github.com/settings){: target="_blank"} 메뉴(우측 상단 프로필 클릭 시 확인 가능)에 들어가서 [SSH and GPG keys](https://github.com/settings/keys){: target="_blank"} 항목으로 들어간다.

첫번째 항목인 SSH keys 우측에 New SSH key 버튼을 클릭한다.

- Title에는 이름을 임의로 설정한다.
- Key type은 그대로 두고, Key 에는 ssh public key의 내용을 그대로 복사 붙여넣기한다.
- Add SSH key 버튼을 눌러 완료.


---

## 각주
[^fn1]: ssh key에 passphrase를 사용하면 원격 연결시마다 passphrase를 입력해야 하는데, 이런 과정을 ssh-agent 가 생략해줄 수 있다.