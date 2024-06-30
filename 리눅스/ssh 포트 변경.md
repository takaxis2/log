1. 22번 포트에서 돌아가고 있는지 확인한다
``` sh
netstat -nlp | grep ssh
```

2.  /etc/ssh/sshd_config 파일의 포트 부분의 주석을 해제하고 원하는 포트 번호로 수정한다

3. ssh를 재시작한다
``` sh
systemctl restart sshd
```

아마 여기서 255 에러 코드가 뜬다면 SELinux 설정을 바꿔줘야 한다

``` sh
semanage port -l | grep xxxx //해당 포트와 겹치는 포트 있는지 확인
semanage port -a -t ssh_port_t -p tcp xxxx //포트 적용, 설정파일에도 적혀 있음
semanage port -l | grep ssh_port_t //적용 되었는지 확인
```

방화벽