# 제19장: 게임 서버 호스팅

## 19.1 Minecraft 서버 구축

```yaml
version: '3.8'

services:
  minecraft:
    image: itzg/minecraft-server:latest
    container_name: minecraft
    environment:
      EULA: "TRUE"
      TYPE: "PAPER"
      VERSION: "LATEST"
      MEMORY: "4G"
      DIFFICULTY: "normal"
      OPS: "player1,player2"
      MAX_PLAYERS: "20"
      MOTD: "우리 홈서버 마인크래프트!"
    ports:
      - 25565:25565
    volumes:
      - ./minecraft-data:/data
    restart: unless-stopped
```

## 19.2 Valheim 서버 운영

```yaml
version: '3.8'

services:
  valheim:
    image: lloesche/valheim-server:latest
    container_name: valheim
    environment:
      - SERVER_NAME=MyValheimServer
      - WORLD_NAME=MyWorld
      - SERVER_PASS=secretpassword
      - SERVER_PUBLIC=false
    ports:
      - 2456-2458:2456-2458/udp
    volumes:
      - ./valheim-data:/config
      - ./valheim-data:/opt/valheim
    restart: unless-stopped
```

## 19.3 Pterodactyl Panel로 게임 서버 관리

[Pterodactyl](https://pterodactyl.io/)은 게임 서버 관리 패널입니다.

설치는 공식 문서 참조: https://pterodactyl.io/project/introduction.html

## 19.4 Steam 게임 스트리밍 (Sunshine/Moonlight)

[Sunshine](https://github.com/LizardByte/Sunshine) + [Moonlight](https://moonlight-stream.org/)로 게임 스트리밍:

```yaml
version: '3.8'

services:
  sunshine:
    image: lizardbyte/sunshine:latest
    container_name: sunshine
    network_mode: host
    environment:
      - PUID=1000
      - PGID=1000
    volumes:
      - ./sunshine:/config
    devices:
      - /dev/dri:/dev/dri
    restart: unless-stopped
```

---

**다음 장에서는**: 로컬 LLM과 AI 서비스를 알아보겠습니다.
