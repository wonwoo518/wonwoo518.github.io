---

layout: post

title: Windows 10 서브시스템 Linux (WSL)에서 GUI 프로그램 사용하기

description: >
  WSL을 사용하다 보면 콘솔에서 vi, vim등을 사용하는게 익숙하지 않아서 불편할 때가 많다. 
  이런 경우 GUI환경이 제공되는 텍스트 에디터를 쓰고 싶어진다. 이런 경우 어떻게 할까?  
  WSL 에서 GUI 프로그램을 사용하기 위해선 크게 두가지 작업을 해야한다.
  하나는 Windows 10에서의 작업, 다른 하나는 WSL의 작업이다. 
tags: [ubuntu]

---

## Windows 10에서의 작업
Xming을 설치한다. Linux의 GUI 프로그램을 화면에 보여주는 역할을 한다.
설치 - https://sourceforge.net/projects/xming/

## WSL에의 작업 
##### ssh 데몬 띄우기
~~~js
#sudo apt-get install  openssh-server
#sudo service ssh --full-restart
~~~

#### display export 하기
~~~js
#export DISPLAY=:0
~~~

#### GUI 프로그램 실행하기 > sublime text를 실행해보자


~~~js
#sudo apt-get update
#sudo apt-get install sublime-text
#subl
~~~
