# uWSGI

WSGI의 한 종류로 'Web Server Gateway Interface'

웹 서비스와 파이썬 통신을 위한 게이트 웨이









```
pip install uwsgi
```





시작 스크립트 (start.sh)

```
#/bin/bash

uwsgi --ini uwsgi.ini
```





종료 스크립트 (stop.sh)

```
#/bin/bash

uwsgi --stop ./uwsgi.pid
```



uwsgi 설정 파일 (uwsgi.ini)

```
[uwsgi]
module=wsgi:app

processes=2
socket=./uwsgi.sock
chmod-socket=666
pidfile=./uwsgi.pid
daemonize=./logs/uwsgi.log

plugins-dir=/usr/local/Cellar/uwsgi/2.0.19.1_1/libexec/uwsgi
plugin=python3

log-reopen=true
die-on-term=true
master=true
vacuum=true
```

1) module 설정 의미는 파이썬 wsgi.py 에서 app 이라는 모듈을 실행하겠다는 의미
2) processes : 게이트 웨이 프로세스 2개를 띄우겠다는 의미 (실제로 프로세스 확인해보면 2개 띄워져 있다.)
3) socket, chmod-socket : 소켓 저장 경로와 권한을 설정한다.
4) pidfile : pid 파일 저장 경로 지정 한다.
5) daemonize : 로그 저장 경로 지정한다.
6) plugins-dir, plugin 은 파이썬 여러버전 설치 되어 있다면 지정 해줘야 실행이 되기 때문에 플러그인 지정 필요



wsgi.py

- 파이썬 코드를 실행시키는 코드

app.py

- 실제 서비스할 flask 앱



실행

```
sh start.sh
```



프로세스 확인

```
ps -ef | grep uwsgi
```





### mind



- eval2.py

```
....



if __name__=="__main__":
    app.run(host='0.0.0.0',port='5003')
```





- wsgi.py

```
# wsgi.py


from eval2 import app

if __name__ == "__main__":
    app.run(host='0.0.0.0',port='5003')
```





- mind.ini 파일

```
# mind.ini


[uwsgi]
module = wsgi:app

pulgin = python3
master = true
processes = 2
thread = 2
enable-threads = true

#chdir = /data/kjjeong78/mind/

socket = mind.sock
chmod-socket = 666
pidfile=./uwsgi.pid
ignore-sigpipe = true
ignore-write-error=true
disable-write-exception=true
vacuum = true

die-on-term = true

#logto = /var/log/uwsgi/%n.log
```









- 서비스 작성

```
sudo vi /etc/systemd/system/mind.service
```

```
[Unit]
Description=uWSGI Instance to server mind
After=network.target

[Service]
User=ubuntu
Group=www-data
WorkingDirectory=/data/kjjeong78/mind
Environment="PATH=/home/ubuntu/anaconda3/envs/pt_py38/bin"
ExecStart=/home/ubuntu/anaconda3/envs/pt_py38/bin/uwsgi --ini mind.ini

[Install]
WantedBy=multi-user.target
```



nginx 설정 파일 경로

- /etc/nginx/sites-available/
- /etc/nginx/sites-enabled/



- nginx mind 파일

```

server {
	listen 5003;
	
	location / {
		include uwsgi_params;
		uwsgi_pass unix:/data/kjjeong78/mind/mind.sock;
	}
}

```







- 포트 상태 확인

```
netstat -tulpn
```



- 링크파일 생성

```
sudo ln -s /etc/nginx/sites-available/mind /etc/nginx/sites-enabled/
```



- 시작

```
sudo systemctl start mind
```



- 재시작

```
sudo systemctl restart mind
```



- 상태 확인

```
sudo systemctl status mind
```



- uwsgi 확인

```
ps -ef | grep uwsgi
```









------------------------

## Reference

- https://co-de.tistory.com/27
- https://blog.ppuing.me/2

- https://velog.io/@dong3789/nginx-%ED%8F%AC%ED%8A%B8%EB%B3%84-%EB%8F%84%EB%A9%94%EC%9D%B8%EC%9D%84-%EB%8B%AC%EC%95%84%EB%B3%B4%EC%9E%90
- https://developer0809.tistory.com/6
- https://postitforhooney.tistory.com/entry/ServerNginX-NginX-%EC%A3%BC%EC%9A%94-%EC%84%A4%EC%A0%95-%EC%A0%95%EB%B3%B4-%ED%8D%BC%EC%98%B4
