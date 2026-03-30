[Back](./README.md)

---

## 📑 Table of Contents

> Docker Demo >> [HERE](https://github.com/hmza-smha/docker-demo)

| Section | Topic |
|---------|-------|
| [1](#-what-is-docker-in-simple-terms) | 🐳 What is Docker (in simple terms)? |
| [2](#-the-actual-problem-docker-solves) | ❗ The actual problem Docker solves |
| [3](#-why-do-we-need-docker) | 🤔 Why do we need Docker? |
| [4](#-core-concepts-the-important-building-blocks) | 🧱 Core Concepts |
| [5](#-simple-analogy-best-way-to-remember) | ⚡ Simple analogy |
| [6](#example) | Example - FastAPI + Angular with Docker |
| [7](#-do-we-need-venv-if-we-are-using-docker) | Do we need `.venv` with Docker? |
| [8](#files-explanation) | Files Explanation |
| [9](#-commands) | Docker Commands Cheatsheet |
| [10](#docker--ubuntu) | Docker & Ubuntu (Server Setup) |
| [11](#docker-vs-kubernetes) | Docker vs Kubernetes |

---

# 🐳 What is Docker (in simple terms)?

Docker is a tool that lets you package an application *with everything it needs* (code, libraries, settings) and run it anywhere consistently.

Think of it like:

> 📦 “A sealed box that contains your app and everything required to run it.”

---

# ❗ The actual problem Docker solves

Before Docker, developers constantly faced this issue:

### 😩 “It works on my machine… but not on yours”

Why?

* Different OS versions
* Different library versions
* Missing dependencies
* Conflicting software

Example:

* Your app needs Python 3.10
* Another app needs Python 3.8
* Installing both can break things

---

### 💡 Docker solves this by:

* Isolating apps from each other
* Packaging dependencies with the app
* Ensuring the app runs the same everywhere

So:

> ✅ Same behavior on your laptop, server, or cloud

---

# 🤔 Why do we need Docker?

### 1. Consistency

Run the exact same environment everywhere

### 2. Isolation

Apps don’t interfere with each other

### 3. Easy setup

No more “install these 20 things first”

### 4. Scalability

Run multiple copies of your app easily

### 5. Deployment speed

Start apps in seconds instead of minutes

---

# 🧱 Core Concepts (the important building blocks)

---

## 📄 Dockerfile

A **recipe** for building your app environment.

It tells Docker:

* What **base** system to use
* What **dependencies** to install
* What **commands** to run

Example idea:

```
FROM python:3.10
COPY . /app
RUN pip install -r requirements.txt
CMD ["python", "app.py"]
```

👉 Think of it as:

> 🧾 Instructions to build your app box

---

## 📦 Image

A **built, ready-to-use package** created from a Dockerfile.

* Contains app + dependencies
* Read-only (doesn’t change)

👉 Think of it as:

> 📸 A snapshot/template of your app

---

## 🚀 Container

A **running instance of an image**

* This is the actual running app
* Can start, stop, restart

👉 Think of it as:

> 📦➡️🏃 A live version of the image

---

## 💾 Volume

A way to **persist data outside the container**

Why?

* Containers are **temporary**
* Data inside them disappears if removed

👉 Think of it as:

> 💽 External storage connected to your container

Example:

* Database data stored safely even if container restarts

---

## 🧩 Docker Compose

A tool to run **multiple containers together**

Docker Compose lets you define everything in one file:

Example:

* App container
* Database container
* Redis container

All started with:

```
docker-compose up
```

👉 Think of it as:

> 🎬 A script that runs your whole system

---

## ☁️ Docker Hub

Docker Hub is an online registry where you:

* Store images
* Share them
* Download ready-made images

Examples:

* `node`
* `mysql`
* `nginx`

👉 Think of it as:

> 🌐 App Store for Docker images

---

# 🧠 Putting it all together (big picture)

1. You write a **Dockerfile**
2. Build it → get an **Image**
3. Run it → get a **Container**
4. Use **Volumes** to save data
5. Use **Docker Compose** to run multiple containers
6. Store/share images on **Docker Hub**

---

# ⚡ Simple analogy (best way to remember)

| Concept    | Real-life analogy |
| ---------- | ----------------- |
| Dockerfile | Recipe            |
| Image      | Frozen meal       |
| Container  | Cooked meal       |
| Volume     | Refrigerator      |
| Compose    | Full meal plan    |
| Docker Hub | Grocery store     |

---

# 🚀 Final takeaway

Docker isn’t just a tool — it’s a solution to a **huge pain in software development**:

> 👉 Making sure software runs the same everywhere without headaches.

---

# Example


* 2 FastAPI apps (backend1, backend2)
* 1 Angular app (frontend)
* Full development with live reload
* Using only terminal on Windows 11

We’ll use:

* Docker
* Docker Compose

---

# 🧱 1. Project Structure

Create something like:

```
project/
│
├── backend1/
│   ├── app/
│   ├── Dockerfile
│   └── requirements.txt
│
├── backend2/
│   ├── app/
│   ├── Dockerfile
│   └── requirements.txt
│
├── frontend/
│   ├── src/
│   ├── Dockerfile
│   └── package.json
│
└── docker-compose.yml
```

* With docker, you do not need `.venv`

---

# ⚙️ 2. FastAPI Dockerfile (for BOTH backends)

Example for `backend1/Dockerfile` (same for backend2):

```dockerfile
FROM python:3.11

WORKDIR /app

COPY requirements.txt .
RUN pip install -r requirements.txt

COPY . .

CMD ["uvicorn", "app.main:app", "--host", "0.0.0.0", "--port", "8000", "--reload"]
```

👉 Important:

* `--reload` enables auto-restart on code changes

---

# 🌐 3. Angular Dockerfile (dev mode)

`frontend/Dockerfile`:

```dockerfile
FROM node:20

WORKDIR /app

COPY package.json package-lock.json ./
RUN npm install

COPY . .

CMD ["npm", "run", "start", "--", "--host", "0.0.0.0"]
```

---

# 🧩 4. docker-compose.yml

This is the heart of everything:

```yaml
version: "3.9"

services:
  backend1:
    build: ./backend1
    ports:
      - "8001:8000"
    volumes:
      - ./backend1:/app
    command: uvicorn app.main:app --host 0.0.0.0 --port 8000 --reload

  backend2:
    build: ./backend2
    ports:
      - "8002:8000"
    volumes:
      - ./backend2:/app
    command: uvicorn app.main:app --host 0.0.0.0 --port 8000 --reload

  frontend:
    build: ./frontend
    ports:
      - "4200:4200"
    volumes:
      - ./frontend:/app
      - /app/node_modules
    command: npm run start -- --host 0.0.0.0
```

---

# ▶️ 5. First Run (VERY IMPORTANT)

From project root:

```bash
docker compose build
docker compose up
```

Now:

* Backend1 → [http://localhost:8001](http://localhost:8001)
* Backend2 → [http://localhost:8002](http://localhost:8002)
* Angular → [http://localhost:4200](http://localhost:4200)

---

# 🔄 6. Development Workflow

## ✅ When you change FastAPI code

👉 Nothing extra needed

Because:

* `--reload` is enabled
* Volume is mounted

So:

> Edit file → container auto-restarts → changes applied

---

## ✅ When you change Angular code

👉 Nothing extra needed

Because:

* Angular dev server auto reloads
* Volume mount syncs files

So:

> Edit → browser refreshes automatically

---

## ⚠️ When you change dependencies

### Backend (requirements.txt)

```bash
docker compose build backend1
docker compose up
```

### Frontend (package.json)

```bash
docker compose build frontend
docker compose up
```

---

## ⚠️ When you change Dockerfile

You MUST rebuild:

```bash
docker compose build
docker compose up
```

---

# 🧠 Key Idea (very important)

### Volumes = live sync

```yaml
volumes:
  - ./backend1:/app
```

👉 This means:

* Your local files override container files
* No rebuild needed for code changes

---

# 🧹 7. Useful Commands (terminal only)

### Stop everything

```bash
docker compose down
```

### Run in background

```bash
docker compose up -d
```

### Rebuild one service

```bash
docker compose build backend1
```

### View logs

```bash
docker compose logs -f
```

---

# 🪟 Windows 11 Notes (IMPORTANT)

### 1. Use Docker Desktop (but CLI only)

Even if you don’t use GUI, you still need it installed.

### 2. Use WSL2 backend (recommended)

Much faster file syncing

### 3. Line endings issue

If you see weird errors:

* Use LF instead of CRLF

---

# 🚀 Real Developer Flow (simple)

1. Start project:

```bash
docker compose up
```

2. Edit code normally in VS Code

3. Changes reflect instantly:

* FastAPI → auto reload
* Angular → hot reload

4. Only rebuild if:

* Dependencies change
* Dockerfile changes

---

# 🔥 Final Mental Model

* Docker = environment
* Volumes = live code sync
* Reload flags = automatic restart
* Compose = runs everything together

---

# Do we need `.venv.` if we are using docker?
Short answer: **No, you usually don’t need `.venv` when using Docker** — but there are a couple of important nuances.

---

# 🧠 What `.venv` was solving

A `.venv` (virtual environment) in Python is used to:

* Isolate dependencies
* Avoid conflicts between projects
* Keep your system Python clean

👉 Basically, it solves the same kind of problem Docker solves.

---

# 🐳 What Docker does instead

Docker already gives you:

* A completely isolated environment
* Its own Python installation
* Its own installed packages

So:

> Each container = its own “virtual environment” (but stronger)

---

# ⚖️ So do you need `.venv` with Docker?

## ✅ Inside Docker → NO

You **do NOT create or use `.venv` inside containers**

Why?

* The container itself *is* the environment
* Adding `.venv` = unnecessary complexity

---

## ✅ Outside Docker (on your machine) → MAYBE

You have two options:

---

### 🟢 Option 1 (Recommended for your setup)

👉 **No `.venv` at all**

* You run everything through Docker
* You never install dependencies locally

✔ Clean
✔ Consistent
✔ No conflicts

---

### 🟡 Option 2 (Hybrid workflow)

👉 Use `.venv` locally **only for development tools**

Example:

* Running scripts without Docker
* Using linters (flake8, black)
* Running tests quickly

---

# 🚫 What you should NOT do

❌ Don’t copy `.venv` into Docker image
❌ Don’t mount `.venv` into container

Example of what to avoid:

```yaml
- ./backend:/app   # OK
- ./backend/.venv:/app/.venv   # ❌ WRONG
```

---

# 🔥 Best Practice for your FastAPI + Docker setup

### Your backend folder should look like:

```
backend/
├── app/
├── requirements.txt
├── Dockerfile
```

👉 No `.venv` inside

---

# ⚡ How dependencies work instead

In Dockerfile:

```dockerfile
RUN pip install -r requirements.txt
```

👉 That’s your “virtual environment”

---

# 🧠 Mental Model (important)

| Tool    | Isolation Level |
| ------- | --------------- |
| `.venv` | Python-only     |
| Docker  | Full OS-level   |

👉 Docker is basically:

> `.venv` + OS + system libraries + runtime

---

# 🚀 When `.venv` is still useful

Use it if:

* You’re NOT using Docker
* You’re doing quick scripts
* You want faster local testing without containers

---

# 🔑 Final takeaway

👉 If you're fully using Docker (like your setup):

> ❌ You don’t need `.venv`
> ✅ Docker replaces it completely

---


# Files Explanation

We’re working with:

* Docker
* Docker Compose

---

# ✅ 1. Cleaned-Up Dockerfile (Frontend)

```dockerfile
# Use official Node.js version 20 image
FROM node:20

# Set working directory inside container
WORKDIR /app

# Copy dependency files first (for caching)
COPY package.json package-lock.json ./

# Install dependencies
RUN npm install

# Copy the rest of the project files
COPY . .

# Start Angular dev server and expose it to outside
CMD ["npm", "run", "start", "--", "--host", "0.0.0.0"]
```

---

# 🔍 Explanation (Line by Line)

### `FROM node:20`

* Use a ready-made image that already has Node.js installed
* Saves you from installing Node manually

👉 Think: “Start from a machine that already has Node”

---

### `WORKDIR /app`

* Sets the working folder inside the container
* All commands after this run inside `/app`

👉 Equivalent to:

```bash
cd /app
```

---

### `COPY package.json package-lock.json ./`

* Copies only dependency files first

👉 Why?

* Docker caches this step
* If dependencies don’t change → faster rebuilds

---

### `RUN npm install`

* Installs all frontend dependencies

👉 Runs inside the container

---

### `COPY . .`

* Copies ALL project files into container

👉 Includes:

* src/
* angular.json
* everything else

---

### `CMD ["npm", "run", "start", "--", "--host", "0.0.0.0"]`

* Starts Angular dev server

👉 Important part:

```bash
--host 0.0.0.0
```

* Makes app accessible from outside container
* Without it → won’t open in browser

---

# ✅ 2. Cleaned docker-compose.yml

```yaml
version: "3.9"

services:
  backend1:
    build: ./backend1
    ports:
      - "8001:8000"
    volumes:
      - ./backend1:/app
    command: uvicorn app.main:app --host 0.0.0.0 --port 8000 --reload

  backend2:
    build: ./backend2
    ports:
      - "8002:8000"
    volumes:
      - ./backend2:/app
    command: uvicorn app.main:app --host 0.0.0.0 --port 8000 --reload

  frontend:
    build: ./frontend
    ports:
      - "4200:4200"
    volumes:
      - ./frontend:/app
      - /app/node_modules
    command: npm run start -- --host 0.0.0.0
```

---

# 🔍 Explanation (Line by Line)

---

## `version: "3.9"`

* Defines Docker Compose file format version
* Mostly for compatibility

---

## `services:`

* Defines all containers (apps)

---

# 🔹 backend1

### `backend1:`

* Name of the service (container)

---

### `build: ./backend1`

* Build image using Dockerfile inside `backend1` folder

---

### `ports:`

```yaml
- "8001:8000"
```

* Maps ports:

| Your PC | Container |
| ------- | --------- |
| 8001    | 8000      |

👉 Access via:

```
localhost:8001
```

---

### `volumes:`

```yaml
- ./backend1:/app
```

* Sync local folder with container

👉 Meaning:

* Edit code locally → reflected instantly in container

---

### `command:`

```bash
uvicorn app.main:app --host 0.0.0.0 --port 8000 --reload
```

* Starts FastAPI server

👉 Important flags:

* `--reload` → auto restart on code changes
* `--host 0.0.0.0` → accessible from outside

---

# 🔹 backend2

Same as backend1 but:

```yaml
- "8002:8000"
```

👉 Runs on:

```
localhost:8002
```

---

# 🔹 frontend

---

### `frontend:`

* Angular app container

---

### `build: ./frontend`

* Uses frontend Dockerfile

---

### `ports:`

```yaml
- "4200:4200"
```

👉 Open in browser:

```
http://localhost:4200
```

---

### `volumes:`

```yaml
- ./frontend:/app
- /app/node_modules
```

#### First line:

* Sync your Angular code

#### Second line:

```yaml
- /app/node_modules
```

👉 VERY IMPORTANT:

* Prevents node_modules from being overwritten
* Keeps container-installed dependencies safe

---

### `command:`

```bash
npm run start -- --host 0.0.0.0
```

* Starts Angular dev server

---

# 🧠 Big Picture (What’s happening)

When you run:

```bash
docker compose up
```

👉 Docker:

1. Builds images
2. Starts 3 containers:

   * backend1
   * backend2
   * frontend
3. Connects ports
4. Syncs code using volumes

---

# 🔥 What happens when you change code?

### Backend:

* `--reload` detects change → restarts server

### Frontend:

* Angular dev server auto-refreshes

👉 No rebuild needed ✅

---

# ⚡ When do you rebuild?

Only if you change:

* `package.json`
* `requirements.txt`
* Dockerfile

```bash
docker compose build
```

---

# 🚀 Final takeaway

This setup gives you:

* Live development 🔄
* Multiple services 🧩
* Zero environment conflicts 🧼

---


# Commands

Here’s a **practical, no-BS command cheat sheet** to manage everything in Docker using only terminal (CMD / PowerShell).

---

# 🐳 1. Container Management

## ▶️ Run / Start / Stop

```bash
docker run <image>
```

Run a new container

```bash
docker start <container>
```

Start stopped container

```bash
docker stop <container>
```

Stop running container

```bash
docker restart <container>
```

Restart container

---

## 📋 List Containers

```bash
docker ps
```

Running containers only

```bash
docker ps -a
```

All containers (including stopped)

---

## ❌ Remove Containers

```bash
docker rm <container>
```

Force remove:

```bash
docker rm -f <container>
```

Remove ALL stopped containers:

```bash
docker container prune
```

---

## 🧠 Inspect / Logs

```bash
docker logs <container>
```

Live logs:

```bash
docker logs -f <container>
```

Inspect details:

```bash
docker inspect <container>
```

---

## 💻 Execute Inside Container

```bash
docker exec -it <container> bash
```

If bash not available:

```bash
docker exec -it <container> sh
```

---

# 🧱 2. Image Management

## 📋 List Images

```bash
docker images
```

---

## 📦 Build Image

```bash
docker build -t myapp .
```

---

## ❌ Remove Images

```bash
docker rmi <image>
```

Force:

```bash
docker rmi -f <image>
```

Remove unused images:

```bash
docker image prune
```

Remove EVERYTHING unused:

```bash
docker system prune -a
```

---

# 💾 3. Volume Management

## 📋 List Volumes

```bash
docker volume ls
```

---

## 🔍 Inspect Volume

```bash
docker volume inspect <volume>
```

---

## ❌ Remove Volume

```bash
docker volume rm <volume>
```

Remove unused volumes:

```bash
docker volume prune
```

---

# 🌐 4. Network Management

## List networks

```bash
docker network ls
```

## Inspect network

```bash
docker network inspect <network>
```

## Remove network

```bash
docker network rm <network>
```

---

# 🧩 5. Docker Compose (MOST IMPORTANT for your setup)

Using Docker Compose

---

## ▶️ Start everything

```bash
docker compose up
```

Detached (background):

```bash
docker compose up -d
```

---

## 🛑 Stop everything

```bash
docker compose down
```

---

## 🔄 Rebuild

```bash
docker compose build
```

Rebuild one service:

```bash
docker compose build backend1
```

---

## 📋 View containers

```bash
docker compose ps
```

---

## 📜 Logs

```bash
docker compose logs
```

Follow logs:

```bash
docker compose logs -f
```

Specific service:

```bash
docker compose logs -f backend1
```

---

## 🔁 Restart service

```bash
docker compose restart backend1
```

---

## 🧹 Remove everything (containers + networks)

```bash
docker compose down -v
```

👉 Also removes volumes

---

# 🔥 6. System Cleanup (VERY USEFUL)

## Clean everything unused

```bash
docker system prune
```

Aggressive cleanup:

```bash
docker system prune -a --volumes
```

---

# ⚡ 7. Real Workflow (what you’ll actually use daily)

### Start project

```bash
docker compose up
```

### Stop project

```bash
docker compose down
```

### Rebuild after changes

```bash
docker compose build
docker compose up
```

### Check logs

```bash
docker compose logs -f
```

### Enter container

```bash
docker exec -it <container> bash
```

---

# 🧠 Pro Tips (important)

### 1. Container names

Get them with:

```bash
docker ps
```

---

### 2. If something is “stuck”

```bash
docker compose down -v
docker system prune -a
```

---

### 3. Windows tip

Use:

* PowerShell or CMD ✅
* Avoid Git Bash (sometimes breaks paths)

---

# 🚀 Final takeaway

You really only need to memorize:

```bash
docker compose up
docker compose down
docker compose build
docker compose logs -f
docker ps
docker exec -it <container> bash
```

Everything else is occasional.

---

# Docker & Ubuntu

Now you’re moving from *dev (Docker + hot reload)* to *server (Ubuntu + systemd)*. I’ll give you a **real production-style flow**, step by step.

We’ll still use:

* Docker
* Docker Compose
* And manage it with Linux service manager: systemd

---

# 🧱 0. Big Picture

On Ubuntu server:

👉 You will:

1. Copy project files
2. Install Docker
3. Run with Docker Compose
4. Use systemd to auto-start on boot

---

# 📦 1. What files to move?

From your project, move ONLY:

```
project/
├── backend1/
│   ├── app/
│   ├── Dockerfile
│   └── requirements.txt
│
├── backend2/
│   ├── app/
│   ├── Dockerfile
│   └── requirements.txt
│
├── frontend/
│   ├── src/
│   ├── Dockerfile
│   ├── package.json
│   └── package-lock.json
│
└── docker-compose.yml
```

---

## ❌ Do NOT move:

* `.venv`
* `node_modules`
* `__pycache__`
* `.git` (optional)

---

## ✅ Best way to transfer

From your local machine:

```bash
scp -r project user@your-server-ip:/home/user/
```

---

# 🖥️ 2. Prepare Ubuntu Server

SSH into server:

```bash
ssh user@your-server-ip
```

---

## Install Docker

```bash
sudo apt update
sudo apt install -y docker.io
```

---

## Install Docker Compose

```bash
sudo apt install -y docker-compose-plugin
```

---

## Enable Docker on boot

```bash
sudo systemctl enable docker
sudo systemctl start docker
```

---

## (Optional) Run Docker without sudo

```bash
sudo usermod -aG docker $USER
```

Then logout/login again.

---

# ▶️ 3. Run your project manually first

Go to project:

```bash
cd ~/project
```

Run:

```bash
docker compose up -d
```

Check:

```bash
docker ps
```

---

## 🌐 Test in browser

* `http://server-ip:4200`
* `http://server-ip:8001`
* `http://server-ip:8002`

---

# ⚙️ 4. Create systemd service

Now the important part 🔥

Create service file:

```bash
sudo nano /etc/systemd/system/myapp.service
```

---

## Paste this:

```ini
[Unit]
Description=My Docker Compose App
Requires=docker.service
After=docker.service

[Service]
WorkingDirectory=/home/user/project
ExecStart=/usr/bin/docker compose up -d
ExecStop=/usr/bin/docker compose down
Restart=always
TimeoutStartSec=0

[Install]
WantedBy=multi-user.target
```

---

# 🔍 Explanation

### `[Unit]`

* Defines dependencies

`Requires=docker.service`
👉 Start Docker first

---

### `[Service]`

`WorkingDirectory`
👉 Where your docker-compose.yml is

---

`ExecStart`
👉 Runs your app

---

`ExecStop`
👉 Stops containers

---

`Restart=always`
👉 Auto restart if it crashes

---

### `[Install]`

* Run on system boot

---

# ▶️ 5. Enable & start service

Reload systemd:

```bash
sudo systemctl daemon-reexec
sudo systemctl daemon-reload
```

---

Enable service:

```bash
sudo systemctl enable myapp
```

Start it:

```bash
sudo systemctl start myapp
```

---

# 🔍 6. Check status

```bash
sudo systemctl status myapp
```

Logs:

```bash
journalctl -u myapp -f
```

---

# 🔄 7. Updating your app

When you change code:

```bash
cd ~/project
docker compose build
docker compose up -d
```

OR restart service:

```bash
sudo systemctl restart myapp
```

---

# 🧹 8. Stopping app

```bash
sudo systemctl stop myapp
```

Disable auto-start:

```bash
sudo systemctl disable myapp
```

---

# ⚠️ Production Notes (IMPORTANT)

## ❌ Don’t use `--reload` in production

Remove from backend command:

```bash
--reload
```

---

## ❌ Angular dev server is NOT production

Instead build it:

```bash
npm run build
```

Then serve with nginx (optional step if you want later)

---

## 🔐 Open firewall ports

```bash
sudo ufw allow 4200
sudo ufw allow 8001
sudo ufw allow 8002
```

---

# 🚀 Final Flow

1. Copy files
2. Install Docker
3. `docker compose up -d`
4. Create systemd service
5. Enable → auto start on boot

---

# 🧠 Mental Model

* Docker → runs your app
* Compose → manages multiple services
* systemd → keeps everything running forever

---

# Docker vs Kubernetes

Let’s build this from the ground up so it’s clear and practical.

---

# ☸️ What is Kubernetes?

Kubernetes is a system that **automatically manages, deploys, scales, and heals containers across multiple machines**.

👉 In simple words:

> Kubernetes runs your apps *reliably at scale* without you manually controlling everything.

---

## 🧠 Why Kubernetes exists

When apps grow, you run into problems like:

* You have 10+ containers running
* Some crash → you must restart them
* Traffic increases → you need more instances
* Servers go down → apps should still work

👉 Kubernetes solves all of that automatically:

* Restarts crashed containers
* Scales apps up/down
* Distributes load
* Keeps your app alive

---

# 🐳 What is Docker (quick recap)

Docker is a tool to:

* Package your app + dependencies
* Run it inside containers

👉 Docker = “create and run containers”

---

# 🔑 Key difference (one sentence)

> Docker runs containers
> Kubernetes manages containers at scale

---

# 🆚 Kubernetes vs Docker (Table)

| Feature              | Docker                     | Kubernetes                       |
| -------------------- | -------------------------- | -------------------------------- |
| **What it is**       | Container runtime/platform | Container orchestration system   |
| **Main job**         | Build & run containers     | Manage many containers           |
| **Scope**            | Single machine (mostly)    | Cluster (multiple machines)      |
| **Scaling**          | Manual                     | Automatic                        |
| **Self-healing**     | ❌ Limited                  | ✅ Yes (auto restart, reschedule) |
| **Load balancing**   | ❌ No                       | ✅ Built-in                       |
| **Deployment**       | Simple (`docker run`)      | Declarative (`kubectl apply`)    |
| **Networking**       | Basic                      | Advanced (service discovery)     |
| **Use case**         | Dev, small apps            | Production, large systems        |
| **Learning curve**   | Easy                       | Hard                             |
| **Handles failures** | ❌ You manage it            | ✅ Automatic recovery             |
| **Works together?**  | —                          | ✅ Uses Docker images             |

---

# 🧩 Simple analogy

| Concept    | Real-world meaning                                    |
| ---------- | ----------------------------------------------------- |
| Docker     | A single delivery truck                               |
| Kubernetes | A full logistics company managing thousands of trucks |

---

# ⚙️ How they work together

1. You create app → Docker image
2. Kubernetes runs and manages many copies of that image

```text
Docker → builds containers
Kubernetes → orchestrates them
```

---

# 🚀 When to use what

## Use Docker only:

* Local development
* Small projects
* Single server (like your Ubuntu + systemd setup)

---

## Use Kubernetes when:

* You have many services (microservices)
* Need auto scaling
* Need zero downtime
* Running in cloud environments

---

# ⚠️ Reality for you

Based on your setup (FastAPI + Angular + one server):

👉 Kubernetes is unnecessary complexity right now

👉 Stick with:

* Docker Compose
* systemd (what you already learned)

---

# 🧠 Final takeaway

* Docker = packaging & running apps
* Kubernetes = running apps **at scale and reliably**

---
[Back](./README.md)