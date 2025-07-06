# Hosting services


## RestAPI
Representational state transfer
way for programs to talk to one another over the web using standard HTTP methods
 | HTTP Method| Purpose | Example     |
|----------|-----|----------------|
| GET   |Read data  |/ping to check service       |
| POST      | send data  |/asir to upload audio for ASR|

- it accepts and return JSON responses

## FastAPI
modern python web framework to build RESTful APIs quickly and efficiently.
- known to be fast
- easy to write and test
- self-documenting


## Flask

## Uvicorn




# Docker
- packages code and dependencies into container
- app run the same anywhere
- Lightweight, portable, efficient alternative to VMs

---

## Key concepts
### **Image**
- A **blueprint** of your application environment
- Built from a **Dockerfile**
- Contains layers: base OS, installed packages, code, etc.

### **Container**
- A **running instance** of an image
- Isolated process using **host kernel**, own **filesystem + network + PID namespace**
- Starts fast, minimal overhead, shares host kernel namespace**


### Docker Hub:
- The "GitHub of Docker"
- Hosts public and private images
- Example: `docker pull python:3.9-slim`

### Volume
- persisted data shared between host and container
- Useful for saving logs, DBs, and intermediate output

- 
### **Network**
- Virtual bridge connecting containers together
- Containers can talk over virtual IPs
- Created and managed using `--network` or `docker-compose`

## Dockerfile (image blueprint)
- script that defines how image is built

**Base image:**
- 
example: From:python:3.9-slim
- starts with minimal Debian + Python 3.9 preinstalled
- shared base filesystem for your container

**WORKDIR:**
```bash
WORKDIR /app
```
- set the current working directory inside the container
- all subsequent commands (COPY, RUN, etc) will run as if inside /app

**COPY and RUN**
```bash
COPY requirements.txt .
RUN pip install --no-cache-dir -r requirements.txt
```
- copy: Transfers files from your host machine into the image at the current WORKDIR (here, /app)
- in this case, copy the requirements list
- RUN: execute shell command in the container during the image huild phase (part of image's file system)
- run once only
- In this case, it is to install all dependencies in requirements.txt
- -no-cache-dir: Prevents pip from caching downloaded packages → keeps image smaller

**Model Preload**
```bash
RUN python - <<'EOF'
from transformers import Wav2Vec2Processor, Wav2Vec2ForCTC
Wav2Vec2Processor.from_pretrained("facebook/wav2vec2-large-960h")
Wav2Vec2ForCTC.from_pretrained("facebook/wav2vec2-large-960h")
EOF
```

- pre-downloads models into cache during build time
- avoids runtime downloads and delays
- 
 **CMD**
 ```bash
CMD ["uvicorn", "asr.asr_api:app", "--host", "0.0.0.0", "--port", "8001"]
```
- Defines the default runtime command executed when container starts
- This is the main application the container is built to run
- Can be overridden by docker run <new-command>

**EXPOSE**
```bash
EXPOSE 8001
```
- A declaration: this container listens on port 8001
-❗ Purely metadata — does not actually open the port
-  use -p in docker run to expose it

In docker CLI, need to explicitly publish:
```bash
docker run -p 1234:8001 my-asr-app
```

In docker compose, this works with the ports: field
```bash
ports:
- "1234:8001"
```


**ENV**
- set environement variables during build and runtime
- Available to your app via os.environ
- Useful for settings, model paths, credentials (non-secret)

## Docker build
- build the docker image according to the dockerfile (eg. my-asr-app)
```bash
docker build -t my-asr-app .
```
docker build image and take the name as -t my-asr-app., the container is not built yet!
- -t: tag, names the image you're building
- if ommited, dafualts to latest.

 ## Docker run
 ```bash
docker run -p 1234:8001 my-asr-app
```
- Starts a new container from the image
- -p <host>:<container>:
-   Left: port on your host machine (e.g. localhost:1234)
-   Right: port inside the container where app listens (8001)

Enables communication between your browser/curl and the container


## 📦 Docker Compose

Used for defining and running multi-container applications (e.g., web + database) using a `docker-compose.yml` file.

---

### 📝 Example `docker-compose.yml`

```yaml
version: "3.9"

services:
  asr:
    build: .
    container_name: asr-service
    ports:
      - "8001:8001"  # format: "host:container"
    environment:
      - PYTHONUNBUFFERED=1
    volumes:
      - ./data:/app/data
    restart: always

