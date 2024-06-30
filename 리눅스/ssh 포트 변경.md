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

방화벽 설정
``` sh 
firewall-cmd --permanent --zone=public --addport=1022/tcp //포트 추가
firewall-cmd --reload //방화벽 재시작
firewall-cmd --zone=public --list-all //방화벽 list 확인
```

SFTP는 SSH를 기반으로 동작하므로, SSH 포트를 변경하면 SFTP 포트도 변경된다. 하지만 SFTP에 대해 별도로 포트를 설정할 수도 있다

``` sh
sftp -oPort=2223 sftpuser@your_server_ip //ssh처럼 연결하는법
```