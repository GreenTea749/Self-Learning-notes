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
- Instead of weriting docker run commands and managing many containers
- define everything inside a YAML file **docker-compose.yml**
- avoids writing long `docker run` commands and handles orchestration, networks, and volumes easily.

  
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
```
## **services**
- each key under services: defines a container
- either build from Dockerfile (build:)
- or from Docker Hub or another registry (image:)
- Can be defined by either
  - build: from a Dockerfile in current directory
  - image: <name> pull from docker hub or a custom registry

  ## **ports**
  - eg 8000: 1234
  - expose internal container ports to external host machine
  - allow host to access the app
  - map machine port (8000) to container (1234)


  ## **volume**

  2 types:
  ### Bind mount
  ```yaml
  volumes:
  - ./data:/app/data
  ```
   - sync host folder (./data) with container path (/app/data)
   - basically allow the container to contain the data from that host folder

### Named volumes (persistant, managed by Docker)
```yaml
volumes:
- esdata01:/usr/share/elasticsearch/data
```
- persists data even when container is deleted
- Declared under top-level volumes
 
  ## **environment**
  - sets envrionment variables in the container
  ```yaml
  environment:
  - DEBUG=true
  - MODEL_PATH=/app/models
  - PYTHONUNBUFFERED=1
  ```
  - can be accessed in python using os.environ['KEY']
  - can use .env files instead for cleaner management
    
  ## **communication in compose**
  - all services in the same yaml file are placed in an **default isolated bridge network**
  - can talk to one another using service names
  - no need to expose internal ports
  - If no have ports: , the container can only be reached **internally** by other containers

  ## **restart**
  - sets the container restarts policy:
  - no: default, do not restart
  - always: always restart the container
  - on-failure: restart only if the container exits with error
  - unless-stopped: restart unless explicitly stopped
 
  ## **depends_on:
  - specifies startup condition and order:
  ```yaml
  depends_on:
  es01:
    condition: service_healthy
  ```
  - condition: service_healhy waits for target's health check to pass
  - usefull for dependent services such as indexers and web app
 
  ## **healthcheck**:
  - let docker monitor the health of a container:
  ```yaml
  healthcheck:
  test: ["CMD-SHELL", "curl -fs http://localhost:9200/_cluster/health || exit 1"]
  interval: 20s
  timeout: 5s
  retries: 10
  ```
  - runs periodic checks on the container
  - mark as unealthy if test fails *retries* number of times
  - required for the *depends_on* conditon
 
  ## **networks**
  - controls how containers communicate with each other
  - by default, Compose creates a bridge network
  - all containers in this same file share this network
  - containers can reach each other via their service names, so no need for EXPOSe to access between containers
 
  ## **Common commands**
  1) build and start all containers (rebuild if it is needed)
  ```bash
  docker compose up --build
  ```
  2) stop and remove all containers
  ```bash
  docker compose down
  3) check running services
  ```bash
  docker compose ps
  ```
  4) rebuild image without restarting
  ```bash
  docker compose build
  ```
  5) execute command inside the container
  ```bash
  docker compose exec ___ ____
  ```
  6) restart a specific service only
  ``` bash
  docker compose restart asr
  ```
  
