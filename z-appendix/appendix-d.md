# 부록 D: 하드웨어 업그레이드 가이드

## D.1 RAM 추가

### 현재 RAM 확인

```bash
# 설치된 RAM
free -h

# 상세 정보
sudo dmidecode --type memory

# 사용 가능한 슬롯
sudo lshw -class memory
```

### 업그레이드 절차

1. 서버 종료 및 전원 차단
2. 미니 PC 케이스 오픈
3. RAM 슬롯 확인 (보통 SO-DIMM)
4. RAM 장착 (딸깍 소리가 날 때까지 누름)
5. 부팅 후 인식 확인

### 권장 RAM

- **8GB**: 기본 서비스 운영
- **16GB**: 권장 (여러 서비스 + 개발)
- **32GB**: 가상화, 대규모 데이터베이스

N100 미니 PC는 보통 최대 16GB까지 지원.

## D.2 스토리지 확장

### 내장 스토리지 업그레이드

1. 기존 데이터 백업
2. M.2 NVMe SSD 구매
3. 클론 (Clonezilla 사용) 또는 새로 설치
4. 기존 드라이브 교체

### 외장 스토리지 추가

```bash
# 연결된 드라이브 확인
lsblk
sudo fdisk -l

# 파티션 생성
sudo fdisk /dev/sdb
# n (새 파티션) -> p (primary) -> Enter -> Enter -> w (저장)

# 파일시스템 생성
sudo mkfs.ext4 /dev/sdb1

# 마운트
sudo mkdir /mnt/data2
sudo mount /dev/sdb1 /mnt/data2

# 자동 마운트 설정 (/etc/fstab)
UUID=xxx /mnt/data2 ext4 defaults 0 2
```

## D.3 네트워크 카드 업그레이드

### USB 기가비트 어댑터

일부 N100 미니 PC는 USB 3.0 기가비트 어댑터 추가 가능.

```bash
# 네트워크 인터페이스 확인
ip addr show
```

### 듀얼 NIC 활용

듀얼 NIC 모델의 경우:
- NIC 1: 인터넷 연결
- NIC 2: 로컬 네트워크 (스위치 연결)

## D.4 쿨링 개선

### 팬리스 모델

- 정기적 먼지 제거
- 통풍이 잘 되는 곳에 배치
- 선택사항: 소형 USB 팬 추가

### 액티브 쿨링 모델

```bash
# 온도 확인
sensors

# CPU 온도
cat /sys/class/thermal/thermal_zone*/temp
```

### 온도가 높을 때

1. 먼지 제거 (압축 공기)
2. 써멀 페이스트 재도포
3. 케이스 팬 추가
4. 위치 변경 (통풍 개선)

---

**주의**: 하드웨어 작업 전 반드시 백업하고, 정전기 방지 조치를 취하세요.
