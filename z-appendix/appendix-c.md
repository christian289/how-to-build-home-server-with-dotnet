# 부록 C: 트러블슈팅 가이드

## C.1 포트 충돌 해결

### 증상
```
Error: bind: address already in use
```

### 해결 방법

1. 사용 중인 포트 확인:
```bash
sudo lsof -i :8080
sudo netstat -tulpn | grep :8080
```

2. 프로세스 종료:
```bash
kill -9 <PID>
```

3. Docker 포트 변경:
```yaml
ports:
  - "8081:8080"  # 외부 포트 변경
```

## C.2 권한 문제 해결

### 증상
```
Permission denied
```

### 해결 방법

1. 파일 소유권 확인:
```bash
ls -la /path/to/file
```

2. 소유권 변경:
```bash
sudo chown -R $USER:$USER /path/to/directory
```

3. Docker 볼륨 권한:
```yaml
environment:
  - PUID=1000
  - PGID=1000
```

4. 실행 권한:
```bash
chmod +x script.sh
```

## C.3 네트워크 진단

### Docker 네트워크 문제

```bash
# 네트워크 목록
docker network ls

# 네트워크 상세 정보
docker network inspect bridge

# 컨테이너 IP 확인
docker inspect <container> | grep IPAddress

# 네트워크 재생성
docker network rm <network>
docker network create <network>
```

### 컨테이너 간 통신 안될 때

1. 같은 네트워크에 있는지 확인
2. 컨테이너 이름으로 ping 테스트:
```bash
docker exec container1 ping container2
```

3. 방화벽 확인:
```bash
sudo ufw status
```

### DNS 문제

```bash
# DNS 테스트
nslookup google.com
dig google.com

# DNS 서버 변경 (/etc/resolv.conf)
nameserver 1.1.1.1
nameserver 8.8.8.8
```

## C.4 성능 병목 찾기

### CPU 병목

```bash
# CPU 사용률 확인
top
htop

# 프로세스별 CPU 사용률
ps aux --sort=-%cpu | head -10

# Docker 컨테이너 리소스
docker stats
```

### 메모리 병목

```bash
# 메모리 사용률
free -h
vmstat 1

# 메모리 많이 사용하는 프로세스
ps aux --sort=-%mem | head -10

# 메모리 누수 확인
watch -n 1 free -h
```

### 디스크 I/O 병목

```bash
# 디스크 I/O 확인
iostat -x 1

# 디스크별 사용률
iotop

# 느린 디스크 찾기
sudo hdparm -Tt /dev/sda
```

### 네트워크 병목

```bash
# 네트워크 사용률
iftop
nload

# 연결 상태
netstat -an | grep ESTABLISHED | wc -l

# 패킷 손실
ping -c 100 8.8.8.8
```

## C.5 일반적인 Docker 문제

### 컨테이너가 계속 재시작

```bash
# 로그 확인
docker logs <container>

# 마지막 100줄만
docker logs --tail 100 <container>

# 실시간 로그
docker logs -f <container>
```

### 이미지 pull 실패

```bash
# Docker 재시작
sudo systemctl restart docker

# 다른 레지스트리 사용
docker pull ghcr.io/...

# 수동 다운로드
docker pull --platform linux/amd64 <image>
```

### 볼륨 데이터 손실

```bash
# 볼륨 확인
docker volume ls
docker volume inspect <volume>

# 볼륨 백업
docker run --rm -v <volume>:/source -v $(pwd):/backup alpine \
  tar czf /backup/volume-backup.tar.gz -C /source .

# 볼륨 복원
docker run --rm -v <volume>:/target -v $(pwd):/backup alpine \
  tar xzf /backup/volume-backup.tar.gz -C /target
```

---

대부분의 문제는 로그를 확인하면 해결 실마리를 찾을 수 있습니다!
