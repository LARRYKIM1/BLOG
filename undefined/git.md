# Git 명령어

## init 

* git reset — hard HEAD && git pull : git 코드 강제로 모두 받아오기

## branch

* git checkout branch\_name: 브랜치 선택하기
* git checkout -t remote\_path/branch\_name : 원격 브랜치 선택하기
* git branch -r : 원격 브랜치 목록보기
* git branch -a : 로컬 브랜치 목록보기
* git branch -m 1 2: 브랜치 이름 바꾸기
* git branch -d : 브랜치 삭제하기
* git push remote\_name — delete branch\_name : 원격 브랜치 삭제하기  \( git push origin — delete gh-pages \)

## commit, pull, fetch, push, add  

* git commit -am "설명" : 코드 add하고 커밋  \(git add가 포함\)
* git pull : 최신 코드 받아와 merge 하기 
* git fetch : 최신 코드 받아오기
* git push romote\_name branch\_name : add하고 commit한 코드 git server에 보내기  \(git push origin master\)



## reset

* git reset — hard HEAD^ : commit한 이전 코드 취소하기
* git reset — soft HEAD^ : 코드는 살리고 commit만 취소하기
* git reset — merge : merge 취소하기

## stash

* git stash / git stash save “description” : 작업코드 임시저장하고 브랜치 바꾸기
* git stash pop : 마지막으로 임시저장한 작업코드 가져오기













푸시 

git push romote\_name branch\_name : add하고 commit한 코드 git server에 보내기   
예시: \(git push origin master\)





## 참고 자료 

{% embed url="https://medium.com/@pks2974/%EC%9E%90%EC%A3%BC-%EC%82%AC%EC%9A%A9%ED%95%98%EB%8A%94-%EA%B8%B0%EC%B4%88-git-%EB%AA%85%EB%A0%B9%EC%96%B4-%EC%A0%95%EB%A6%AC%ED%95%98%EA%B8%B0-533b3689db81" %}

