# 제20장: AI와 머신러닝 서비스

## 20.1 로컬 LLM 실행 (Ollama)

[Ollama](https://ollama.ai/)로 로컬에서 ChatGPT 같은 LLM 실행:

```yaml
version: '3.8'

services:
  ollama:
    image: ollama/ollama:latest
    container_name: ollama
    ports:
      - 11434:11434
    volumes:
      - ./ollama:/root/.ollama
    restart: unless-stopped
```

```bash
# 모델 다운로드 및 실행
docker exec -it ollama ollama run llama2

# API 사용
curl http://localhost:11434/api/generate -d '{
  "model": "llama2",
  "prompt": "Why is the sky blue?"
}'
```

### Open WebUI 설치 (ChatGPT UI)

```yaml
services:
  open-webui:
    image: ghcr.io/open-webui/open-webui:main
    container_name: open-webui
    ports:
      - 3004:8080
    environment:
      - OLLAMA_BASE_URL=http://ollama:11434
    volumes:
      - ./open-webui:/app/backend/data
    restart: unless-stopped
```

## 20.2 Stable Diffusion WebUI

AI 이미지 생성 (N100은 CPU 생성이라 느림):

```yaml
version: '3.8'

services:
  stable-diffusion:
    image: ghcr.io/abhigya/stable-diffusion-webui:latest
    container_name: stable-diffusion
    ports:
      - 7860:7860
    volumes:
      - ./stable-diffusion/models:/app/models
      - ./stable-diffusion/outputs:/app/outputs
    restart: unless-stopped
```

**참고**: GPU가 없으면 매우 느립니다. 관심있다면 GPU 서버 고려.

## 20.3 음성 인식 서버 (Whisper)

[OpenAI Whisper](https://github.com/openai/whisper)로 음성-텍스트 변환:

```bash
# Whisper 설치
pip install openai-whisper

# 음성 파일 변환
whisper audio.mp3 --model medium --language Korean
```

## 20.4 얼굴 인식 시스템 구축

Immich (제8장)가 이미 얼굴 인식 기능 포함.

추가로 [Frigate](https://frigate.video/)로 CCTV 얼굴 인식:

```yaml
version: '3.8'

services:
  frigate:
    image: ghcr.io/blakeblackshear/frigate:stable
    container_name: frigate
    privileged: true
    ports:
      - 5000:5000
      - 8971:8971
    volumes:
      - ./frigate/config.yml:/config/config.yml
      - /etc/localtime:/etc/localtime:ro
      - ./frigate/media:/media/frigate
    devices:
      - /dev/dri:/dev/dri
    restart: unless-stopped
```

---

**다음 장에서는**: 이메일 서버, 웹 분석 등 전문 서비스를 알아보겠습니다.
