# Untitled

### 아래의 명령들이 무엇인지 생각해보자.

* git --version
* git config --list
* git config --global --list
* git config --global alias.ss 'status -s'
* git config --global --unset alias.co
* gitk
* git commit -am "설명"
* git remote add origin 주소
* git log --branches --decorate --graph
* git config --global user.name &lt;이름&gt;
* git config --global user.email &lt;이메일&gt;
* git log --branches --decorate --graph
* git remote -v
* git branch -r
* git merge --abort

## 2 Git의 기초

### 2.4 되돌리기

```text
$ git commit -m '커밋메시지'
$ git add forgotten_file
$ git commit --amend
======================================
//실수로 모든 것을 add 하였다고 가정
$ git add * 
//스테이징된 파일을 복구시키면 된다.
$ git reset HEAD 클래스.java
​
```

### 2.5

### 2.6 태그

태그 - 보통 릴리즈할 때 사용

```text
//태그 확인 
$ git tag 
//태그생성 - Annotated 방식(Lightweight도 있음)
git tag -a 'v1.0.1' -m 'Release version 1.0.1'
//보기
$ git show 'v1.0.1'
//특정버전 확인
$ git tag -l "v1.*"
//원격저장소에 모든 태그 올리기
git push origin --tags 
//특정버전 올리기
git push origin v.1.5
//로컬태그 삭제
git tag -d v1.0.0
//원격태그 삭제
git push origin :v1.0.0
```

### 3 Git 브랜치

![image-20200627234633149](file://C:/Users/youwa/AppData/Roaming/Typora/typora-user-images/image-20200627234633149.png?lastModify=1593509645)

```text
git checkout -b iss53
//이슈53처리하는중... 급하게 수정해야되는 것이 왔네...
git checkout master 
git checkout -b hotfix 
//급한건 처리해줬다 가정
git commit -a -m '급수정파일 커밋'
git checkout master
git merge hotfix
git branch -d hotfix
git checkout iss53 //하던일로..
```

 위에 것에서는 fast merge 가 실행된다. 반면, 아래는 3-way Merge를 한다.

```text
git checkout -b iss53
git commit -am "1"
git commit -am "2"
​
git checkout master
git merge iss53
​
git log --branches --decorate --graph
```

![image-20200627234419990](file://C:/Users/youwa/AppData/Roaming/Typora/typora-user-images/image-20200627234419990.png?lastModify=1593509645)

위와 같은 경우 충돌이 발생할 수 있는데, git status로 어떤 파일이 충돌났는지 확인후 `=====` 위쪽이나 아래쪽 내용 중에서 고르거나 새로 작성하여 Merge 한다.

`git merge --abort` 머지 취소

## 5 분산 환경에서의 Git

유연성을 살려 저장소를 운영하는 3가지 방식

* 중앙집중식 워크플로
* Integration-Manager 워크플로
* Dictator and Lieutenants 워크플로

프로젝트에 기여하는 방식을 설명하는데 방식이 매우 다양하다. 3가지 변수를 봐야한다.

* 첫 번째: 살펴볼 변수는 활발히 활동하는 개발자의 수
* 두 번째: 선택한 워크플로우
  * 개발자 모두가 메인 저장소에 쓰기 권한을 갖는 중앙집중형 방식인가?
  * 프로젝트에 모든 Patch를 검사하고 통합하는 관리자가 따로 있는가?
  * 모든 수정사항을 개발자끼리 검토하고 승인하는가?
  * 자신이 그저 돕는게 아니라 어떤 책임을 맡고 있는지?
  * 중간 관리자가 있어서 그들에게 먼저 알려야 하는가?
* 세 번째: 접근 권한

### 커밋 가이드라인

* 공백문자를 깨끗하게 정리하고 커밋해야 한다. `git diff --check`
  * 불필요하게 커밋되는 것을 막아 다른 개발자들이 신경 쓰는 일을 방지할 수 있다.
* 각 커밋은 논리적으로 구분되는 Changeset이다.
* 최대한 수정사항을 한 주제로 요약할 수 있어야 한다.
* 여러 가지 이슈에 대한 수정사항을 하나의 커밋에 담지 않아야 한다.
  * 여러 가지 이슈를 한꺼번에 수정했다고 하더라도 Staging Area를 이용하여 한 커밋에 이슈 하나만 담기도록 한다.
  * 같은 파일의 다른 부분을 수정하는 경우에는 `git add -patch` 명령을 써서 한 부분씩 나누어 Staging Area에 저장해야 한다. 7장 참조.
* 커밋 메시지를 작성할 때 사용하는 규칙이 있다.
  * 첫라인 - 50자가 넘지 않는 간략한 메시지를 쓴다.
  * 첫라인에서 한칸 뛰고 자세히 설명한다.
  * "I added tests for \(테스트를 추가함\)"보다는 "Add tests for \(테스트 추가\)"와 같은 메시지를 작성

```text
영문 50글자 이하의 간략한 수정 요약
​
자세한 설명. 영문 72글자 이상이 되면
라인 바꿈을 하고 이어지는 내용을 작성한다.
특정 상황에서는 첫 번째 라인이 이메일
메시지의 제목이 되고 나머지는 메일
내용이 된다. 빈 라인은 본문과 요약을
구별해주기에 중요하다(본문 전체를 생략하지 않는 한).
​
이어지는 내용도 한 라인 띄우고 쓴다.
​
  - 목록 표시도 사용할 수 있다.
​
  - 보통 '-' 나 '*' 표시를 사용해서 목록을 표현하고
    표시 앞에 공백 하나, 각 목록 사이에는 빈 라인
    하나를 넣는데, 이건 상황에 따라 다르다.
```

[깃프로젝트](https://github.com/git/git) 에 잘쓰여진 커밋메시지들을 확인해보자.\(영어\) `git log --no-merges`

### 비공개 소규모 팀

* 보통 중앙집중형 버전 관리 시스템에서 사용하던 방식을 사용
* 서버가 아닌 클라이언트 쪽에서 Merge 한다.
* 같은 파일을 수정한 것도 아닌데 왜 Push가 거절되는 걸까? \(존-제시카 푸시 사례\)
  * Subversion에서는 서로 다른 파일을 수정하는 Merge 작업은 자동으로 서버가 처리한다.
  * 하지만 Git은 로컬에서 먼저 _Merge_ 해야 한다.
  * 즉, 로컬에서 먼저 Fetch 하고 Merge한 후 원격저장소에 Push한다.
* `git log --no-merges issue54..origin/master`
  * `origin/master` 에 속한 커밋 중 `issue54`에 속하지 않은 커밋을 검색하는 문법
* 토픽 브랜치란, 기능 추가나 버그 수정과 같은 단위 작업을 위한 브랜치이다. \(=Feature branch\)
* 토픽 브랜치에서 특정 작업이 완료되면 다시 통합 브랜치에 병합하는 방식으로 진행됩니다.
* ^\(캐럿\), ~\(틸드\) -&gt; HEAD~3

> #### merge와 rebase차이
>
> merge: 변경 내용의 이력이 모두 그대로 남아 있기 때문에 이력이 복잡해짐. rebase: 이력은 단순해지지만, 원래의 커밋 이력이 변경됨. 정확한 이력을 남겨야 할 필요가 있을 경우에는 사용하면 안됨.
>
> 둘다 충돌된 파일을 수정후 add 한다. 그러나 merge는 commit을 하는데 rebase는 rebase --continue 로 처리한후 master로 이동후 merge 한다.

> 참고사이트: [https://backlog.com/git-tutorial/kr/stepup/stepup1\_4.html](https://backlog.com/git-tutorial/kr/stepup/stepup1_4.html)

![merge](../../.gitbook/assets/image%20%2888%29.png)

![rebase](../../.gitbook/assets/image%20%2887%29.png)

### 비공개 대규모 팀

* 보통 팀을 여러 개로 나눈다.
* \(가정\) John과 Jessica는 “featureA” 기능을 함께 작업하게 됐다. Jessica는 Josie와 함께 “featureB” 기능도 작업하고 있다.
* Integration-manager 워크플로를 선택하는 게 좋다.
* 팀마다 브랜치를 하나씩 만들고 Integration-Manager는 그 브랜치를 Pull 해서 Merge 한다.

### 공개 프로젝트 Fork

* 모든 개발자가 프로젝트의 공유 저장소에 직접적으로 쓰기 권한을 가지지는 않는다. 그래서 프로젝트의 관리자는 몇 가지 일을 더 해줘야 한다.
* Fork를 지원하는 Git 호스팅 \(GitHub, BitBucket, repo.or.cz 등\)
* 관리자는 보통 Fork 하는 것으로 프로젝트를 운영한다. 다른 방식으로 이메일과 Patch를 사용하는 방식도 있다.
* 웹사이트로 가서 Fork 버튼을 누르면 원래 프로젝트 저장소에서 갈라져 나온, 쓰기 권한이 있는 저장소가 하나 만들어진다.
* 작업하던 것을 로컬 저장소의 `master` 브랜치에 Merge 한 후 Push 하는 것\(fork사용x 방식\)보다 리모트 브랜치에 바로 Push를 하는 방식이 훨씬 간단하다.

```text
$ git remote add myfork <url>
$ git push -u myfork featureA
$ git request-pull origin/master myfork  //Pull 요청
// 토픽 브랜치가 어느 시점에 갈라져 나온 것인지, 어떤 커밋이 있는지, 
// Pull 하려면 어떤 저장소에 접근해야 하는지에 대한 내용이 들어 있다.
```

* 다른 주제의 일을 하려고 할 때는 앞서 Push 한 토픽 브랜치에서 시작하지 말고 주 저장소의 `master` 브랜치로부터 만들어야 한다.

```text
$ git checkout -b featureB origin/master  //master를 기준으로 featureB 토픽 브랜치 생성
  ... work ...
$ git commit
$ git push myfork featureB 
$ git request-pull origin/master myfork  
  ... email generated request pull to maintainer ...
$ git fetch origin
```

## 7 Git 도구

### 7.6 히스토리 단장하기

* 로컬 커밋 히스토리를 수정해야 할 때가 있다
* 이미 커밋해서 결정한 내용을 수정할 수 있다.
* 커밋들의 순서도 변경할 수 있고 커밋 메시지와 커밋한 파일도 변경할 수 있다.
* 커밋을 하나로 합치거나 반대로 커밋 하나를 여러 개로 분리할 수도 있다.
* 커밋 전체를 삭제할 수도 있다.
* 이 모든 것은 다른 사람과 코드를 공유하기 전에 해야 한다.
  * Push된 데이터는 수정에 대해선 완전이 끝난 것이다. 고쳐야 할 이유가 생겼더라도 새로 수정작업을 추가해야지 이전 커밋 자체를 수정할 수는 없다.
  * 그렇기에 온전하게 수정 작업을 마무리했다는 확신 없이 작업 내용을 공유하는 저장소로 보내는\(Push\) 것은 피해야 할 행동이다.

```text
// 마지막 커밋 메시지를 열어준다. 
// 텍스트창에서 수정하면 된다.
$ git commit --amend
=========================================================
// 프로젝트 내용을 수정한 경우
// 파일을 수정하고 
$ git add
$ git commit --amend
// 기존 커밋 메시지가 충분할 경우 수정 안하게 --no-edit
$ git commit --amend --no-edit
=========================================================
// 실질적으로 가리키게 되는 것은 수정하려는 커밋의 부모인 네 번째 이전 커밋이다.
$ git rebase -i HEAD~3
// 다시 강조하지만 이미 중앙서버에 Push한 커밋은 절대 고치지 말아야 한다. 
// 이유,  Push 한 커밋을 Rebase 하면 결국 같은 내용을 
// 두 번 Push 하는 것이기 때문에 다른 개발자들이 혼란스러워 할 것이다.
```

rebase 텍스트창에 순서는 log 와는 반대이다. \(SHA-1 값\)

```text
// rebase 순서
pick f7f3f6d changed my name a bit  //가장 오래된것
pick 310154e updated README formatting and added blame
pick a5f4a0d added cat-file
​
// log 순서
a5f4a0d added cat-file
310154e updated README formatting and added blame
f7f3f6d changed my name a bit  //가장 오래된것
```

## 10 Git의 내부

```text
// 모두 같다.
// 서버에 있는 master 브랜치에 접근
$ git log origin/master
$ git log remotes/origin/master
$ git log refs/remotes/origin/master
```

