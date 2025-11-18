# 제16장: 성능 최적화

## 16.1 시스템 리소스 최적화

### CPU 거버너 설정

```bash
# 현재 거버너 확인
cat /sys/devices/system/cpu/cpu0/cpufreq/scaling_governor

# 성능 모드 (최대 성능)
echo performance | sudo tee /sys/devices/system/cpu/cpu*/cpufreq/scaling_governor

# 절전 모드 (저전력)
echo powersave | sudo tee /sys/devices/system/cpu/cpu*/cpufreq/scaling_governor
```

### 메모리 관리

```bash
# 메모리 사용량 확인
free -h
htop

# 캐시 정리 (필요시)
sudo sync && sudo sysctl -w vm.drop_caches=3
```

### 스왑 최적화

```bash
# 스왑 사용 경향 조정 (0-100, 기본 60)
sudo sysctl vm.swappiness=10

# 영구 설정
echo "vm.swappiness=10" | sudo tee -a /etc/sysctl.conf
```

## 16.2 스토리지 성능 튜닝

### SSD TRIM 활성화

```bash
# TRIM 지원 확인
sudo fstrim -v /

# 자동 TRIM 활성화
sudo systemctl enable fstrim.timer
sudo systemctl start fstrim.timer
```

### 파일시스템 마운트 옵션

`/etc/fstab`:
```
UUID=xxx / ext4 defaults,noatime,nodiratime 0 1
```

- `noatime`: 파일 접근 시간 기록 안함 (성능 향상)
- `nodiratime`: 디렉토리 접근 시간 기록 안함

## 16.3 네트워크 최적화

### TCP 최적화

```bash
# /etc/sysctl.conf 추가
cat >> /etc/sysctl.conf <<EOF
net.core.rmem_max=16777216
net.core.wmem_max=16777216
net.ipv4.tcp_rmem=4096 87380 16777216
net.ipv4.tcp_wmem=4096 65536 16777216
EOF

sudo sysctl -p
```

## 16.4 Docker 컨테이너 리소스 제한

### CPU/메모리 제한

```yaml
services:
  app:
    image: myapp:latest
    deploy:
      resources:
        limits:
          cpus: '2.0'
          memory: 2G
        reservations:
          cpus: '0.5'
          memory: 512M
```

### 로그 크기 제한

```yaml
services:
  app:
    image: myapp:latest
    logging:
      driver: "json-file"
      options:
        max-size: "10m"
        max-file: "3"
```

---

**다음 장에서는**: 정기 유지보수와 일반적인 문제 해결 방법을 알아보겠습니다.
