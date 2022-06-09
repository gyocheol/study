# Docker & mysql

`docker pull mysql` 를 입력하면 가장 최신버전의 MySQL Docker 이미지를 다운받을 수 있다.

`dorker images` 를 입력하면 다운로드한 Docker의 이미지를 확인할 수 있다.

---
```
Error response from daemon: Ports are not available: exposing port TCP 0.0.0.0:3306 -> 0.0.0.0:0: listen tcp 0.0.0.0:3306: bind: Only one usage of each socket address (protocol/network address/port) is normally permitted.
```

위와 같은 오류가 뜨는 것은 포트 때문이다. 포트의 실행여부는 cmd에서 `netstat -ano` 를 입력하면 다 볼 수 있으며 특정 번호를 찾기 위해선 `netstat -ano | findstr 포트번호` 를 입력하면 된다.

`taskkill /f /pid 포트번호` 를 입력하면 실행중인 포트번호가 종료된다.

---

`docker run -d -p 9876:3306 -e MYSQL_ROOT_PASSWORD=<password> mysql` 에서 password는 원하는 것을 입력하면 윈도우 Docker에서 mysql을 실행할 수 있다. 포트때매 고생 좀 함..ㅠ

`docker exec -it container id /bin/bash` 를 입력하면 mysql컨테이너로 들어감

 container id는 `docker ps` 를 입력해서 볼 수 있음!

들어가서 아래 명령어 입력



**SQL은 문장이 끝나면 ;를 꼭 붙여주기!**

`mysql -u root -p` : 입력하여 mysql로 들어가기

`CREATE DATABASE TEST;`  : test란 이름의 database만들기

`SHOW DATBASES;` : 만들어진 DB보기

`exit` : 나가기



MySQL Docker 컨테이너 중지 : `docker stop objective_montalcini ` objective_montalcini는 내 컨테이너 NAMES

MySQL Docker 컨테이너 시작 : `docker start objective_montalcini `

MySQL Docker 컨테이너 재시작 : `docker restart objective_montalcini`



fail

다른 방법으로 들어가는 방법

`docker inspect <container id>`  : docker container의 세부정보를 볼 수 있음

`mysql -u root -p --host 172.17.0.2 --port 3306` : 위 세부정보에서 IPAddress를 보고 들어가는 것 > 현재 이건 ip가 막혀있어서(?) 안들어가짐..ㅠ

mysql이 안 깔려있으면 `sudo apt install mysql-client-core-8.0` 를 해주면 된다.







