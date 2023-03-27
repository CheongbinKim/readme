# readme

# Nginx 설치
```
$ sudo apt-get install nginx -y
```
vue 4200 포트로 proxy 설정
```
$ sudo vi /etc/nginx/sites-available/default
location / {
        # First attempt to serve request as file, then
        # as directory, then fall back to displaying a 404.
        #try_files $uri $uri/ =404;
        add_header Access-Control-Allow-Origin *;
        proxy_pass http://127.0.0.1:4200;
        proxy_http_version 1.1;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection "Upgrade";
        # allow socket.io
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header Host $host;
        proxy_set_header Origin "";
}
```
# Node.js 12.x 설치
```
$ curl -sL https://deb.nodesource.com/setup_12.x | sudo -E bash - 
```
# Vue 설치
```
sudo npm install -g vue
sudo npm isntall -g vue-cli
```
# POSTGRESQL 13 설치 (npm pg도 설치 )
```
$ sudo npm install pg -g 
$ sudo sh -c 'echo "deb http://apt.postgresql.org/pub/repos/apt $(lsb_release -cs)-pgdg main" > /etc/apt/sources.list.d/pgdg.list'
$ wget --quiet -O - https://www.postgresql.org/media/keys/ACCC4CF8.asc | sudo apt-key add -
$ sudo apt-get update
$ sudo apt-get -y install postgresql-13
$ sudo systemctl enable postgresql
$ sudo systemctl start postgresql
$ sudo systemctl status postgresql
```

설정 시작
```
$ sudo -i -u postgres
$ psql 

postgres=# create user cjy password 'choimaster' superuser;
postgres=# \du

postgres=# create database conntest owner cjy;
postgres=# \l
                              List of databases
   Name    |  Owner   | Encoding | Collate |  Ctype  |   Access privileges
-----------+----------+----------+---------+---------+-----------------------
 conntest  | cjy      | UTF8     | C.UTF-8 | C.UTF-8 |
 
postgres=# \q

$ postgres@ip-172-31-16-170:~$ exit
```

postgresql 접속 권한 부여
```ubuntu@ip-172-31-16-170:~$ sudo adduser cjy
Adding user `cjy' ...
Adding new group `cjy' (1001) ...
Adding new user `cjy' (1001) with group `cjy' ...
Creating home directory `/home/cjy' ...
Copying files from `/etc/skel' ...
New password:
Retype new password:
passwd: password updated successfully
Changing the user information for cjy
Enter the new value, or press ENTER for the default
        Full Name []: cjy
        Room Number []:
        Work Phone []:
        Home Phone []:
        Other []:
Is the information correct? [Y/n] y

ubuntu@ip-172-31-16-170:~$ sudo -i -u cjy
cjy@ip-172-31-16-170:~$ psql -U cjy -d conntest
psql (13.10 (Ubuntu 13.10-1.pgdg20.04+1))
Type "help" for help.

conntest=# \q

```
pg_hba.conf 설정 (peer -> md5 변경)
```
$ sudo vi /etc/postgresql/13/main/pg_hba.conf
local   all             all                                     md5
# IPv4 local connections:
host    all             all             127.0.0.1/32            md5
# IPv6 local connections:
host    all             all             ::1/128                 md5
```
```
$ sudo systemctl restart postgresql
```

# Express 설치 및 프로젝트 생성 
```
$ sudo npm install -g express-generator
$ express app
$ cd app
$ npm install
``` 

# vue 프로젝트 생성 (app 폴더 안에 위치)
```
ubuntu@ip-172-31-16-170:~/app$ vue init webpack vue-front
? Project name vue-front
? Project description A Vue.js project
? Author cjy
? Vue build standalone
? Install vue-router? Yes
? Use ESLint to lint your code? Yes
? Pick an ESLint preset Standard
? Set up unit tests Yes
? Pick a test runner noTest
? Setup e2e tests with Nightwatch? Yes
? Should we run `npm install` for you after the project has been created? (recommended) npm
```
포트변경 8080 -> 4200
```
$ vi config/index.js
module.exports = {
  dev: {
  ...
    port: 4200,
  ...
  }
}    
```
실행
```
ubuntu@ip-172-31-16-170:~/app$ cd vue-front
ubuntu@ip-172-31-16-170:~/app/vue-front$ npm run dev
```

# PM2 로 데몬관리
```
$ sudo npm install -g pm2
# pm2 log파일 자동관리
$ pm2 install pm2-logrotate   
```

# 실행
```
ubuntu@ip-172-31-16-170:~/app$ pm2 start "npm start" --name back
ubuntu@ip-172-31-16-170:~/app$ cd vue-front/
ubuntu@ip-172-31-16-170:~/app/vue-front$ pm2 start "npm run dev" --name front
```

# vscode server 설치
```
$ sudo apt-get install -y build-essential
$ wget -q https://github.com/cdr/code-server/releases/download/3.4.1/code-server_3.4.1_amd64.deb
$ sudo dpkg -i code-server_3.4.1_amd64.deb
$ echo "export PASSWORD='choimaster'" >> ~/.bashrc
$ source ~/.bashrc
$ sudo ufw allow 8080/tcp
$ vi  ~/.config/code-server/config.yaml
bind-addr: localhost:8080
$ pm2 start "code-server --bind-addr=0.0.0.0:8080" --name code-server
```
