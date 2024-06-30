1. 22번 포트에서 돌아가고 있는지 확인한다
```
netstat -nlp | grep ssh
```

2.  /etc/ssh/sshd_config 파일의 포트 부분의 주석을 해제하고 원하는 포트 번호로 수정한다

3. ssh를 재시작한다
```
systemctl restart sshd
```

아마 여기서 재시작이 되지 않는다면 SEL