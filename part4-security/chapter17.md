# 제17장: 유지보수와 문제 해결

## 17.1 정기 유지보수 체크리스트

### 주간 작업

- [ ] 디스크 공간 확인 (`df -h`)
- [ ] Docker 로그 확인
- [ ] 서비스 가동 상태 확인
- [ ] 백업 성공 여부 확인

### 월간 작업

- [ ] 시스템 업데이트 (`sudo apt update && sudo apt upgrade`)
- [ ] Docker 이미지 업데이트
- [ ] 사용하지 않는 Docker 리소스 정리
- [ ] 백업 복원 테스트
- [ ] 보안 로그 검토

### 분기별 작업

- [ ] UPS 배터리 테스트
- [ ] 하드웨어 청소 (먼지 제거)
- [ ] 비밀번호 변경
- [ ] 재해 복구 훈련

## 17.2 시스템 업데이트 전략

### 안전한 업데이트 절차

```bash
# 1. 백업
~/scripts/backup.sh

# 2. 업데이트 확인
sudo apt update
sudo apt list --upgradable

# 3. 업데이트 실행
sudo apt upgrade -y

# 4. Docker 이미지 업데이트
docker compose pull
docker compose up -d

# 5. 재부팅 필요 시
sudo reboot
```

## 17.3 로그 로테이션과 디스크 정리

### Logrotate 설정

`/etc/logrotate.d/homeserver`:
```
/var/log/homeserver/*.log {
    daily
    rotate 7
    compress
    delaycompress
    missingok
    notifempty
}
```

### 디스크 정리 스크립트

```bash
#!/bin/bash
# ~/scripts/cleanup.sh

# Docker 정리
docker system prune -af --volumes

# 오래된 로그 삭제
find /var/log -type f -name "*.log" -mtime +30 -delete

# APT 캐시 정리
sudo apt autoremove -y
sudo apt clean

echo "정리 완료!"
```

## 17.4 일반적인 문제와 해결 방법

### 네트워크 문제

**증상**: 서비스에 접속 안됨

```bash
# 1. 컨테이너 상태 확인
docker ps -a

# 2. 네트워크 확인
docker network ls
docker network inspect bridge

# 3. 포트 확인
sudo netstat -tulpn | grep LISTEN

# 4. 방화벽 확인
sudo ufw status
```

### 디스크 공간 부족

```bash
# 1. 공간 사용량 확인
df -h
ncdu /

# 2. 큰 파일 찾기
du -sh /* | sort -rh | head -10

# 3. Docker 정리
docker system df
docker system prune -a
```

### 컨테이너 충돌

```bash
# 1. 로그 확인
docker logs <container-name>

# 2. 컨테이너 재시작
docker restart <container-name>

# 3. 포트 충돌 확인
sudo lsof -i :<port>

# 4. 컨테이너 재생성
docker compose down
docker compose up -d
```

### 성능 저하

```bash
# 1. 리소스 사용량 확인
htop
docker stats

# 2. 디스크 I/O 확인
iostat -x 1

# 3. 네트워크 확인
iftop

# 4. 메모리 누수 확인
free -m
```

---

**다음 장에서는**: K3s를 활용한 쿠버네티스 입문을 알아보겠습니다.
