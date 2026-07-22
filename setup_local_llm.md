# Setup Local LLM with docker compose, Ollama, web ui and Continue VS Code Extension

1. Install docker compose

2. Create a docker-compose.yaml :

E.g.

version: '3.8'

services:
  ollama:
    image: ollama/ollama:rocm
    container_name: ollama
    ports:
      - 11434:11434
    volumes:
      - ollama:/root/.ollama
      - ./modelfiles:/modelfiles
    environment:
      - OLLAMA_NUM_PARALLEL=4
      - OLLAMA_MAX_LOADED_MODELS=3
    tty: true
    pull_policy: always
    networks:
      - llm-network
    devices:
      # kfd and dri device access needed for AMDGPU ROCm support to work
      - /dev/kfd:/dev/kfd
      - /dev/dri:/dev/dri
    restart: unless-stopped

  open-webui:
    image: ghcr.io/open-webui/open-webui:main
    container_name: open-webui
    ports:
      - 3000:8080
    depends_on:
      - ollama
    volumes:
      - open-webui:/app/backend/data
    environment:
      - 'OLLAMA_BASE_URL=http://ollama:11434'
    extra_hosts:
      - host.docker.internal:host-gateway
    networks:
      - llm-network
    restart: unless-stopped

volumes:
  ollama: {}
  open-webui: {}

networks:
  llm-network:
    driver: bridge


3. Run: docker compose up --build  -d

4. Open a browsers and go to http://localhost:3000 and setup a local open web ui account and login

5. To download models use: docker exec -it ollama ollama pull <model-name>

6. Install Code OSS (VS code open source) and then the Continue - open source AI code extension

7. Setup the config.yaml for the continue extension:

E.g.

name: Main Config
version: 1.0.0
schema: v1
models:
  # Chat and reasoning (quality model)
  - name: Llama 3.1 8B
    provider: ollama
    model: llama3.1:8b
    apiBase: http://localhost:11434
    roles:
      - chat
      - edit
      - apply
    defaultCompletionOptions:
      temperature: 0.7
      contextLength: 8192
  - name: Qwen Coder 1.5B       # tab autocomplete
    provider: ollama
    model: qwen2.5-coder:1.5b
    apiBase: http://localhost:11434
    roles: [autocomplete]
  - name: Nomic Embed           # @codebase search
    provider: ollama
    model: nomic-embed-text
    apiBase: http://localhost:11434
    roles: [embed]



