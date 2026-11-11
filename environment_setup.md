# Environment Setup Guide

## What the kit provides and what you build

| Provided to you | Built by you |
| --- | --- |
| This setup guide for Docker, minikube, kubectl, Helm, Python 3.12 and uv | Analysis code, dashboard code and choice of framework |
| minikube cluster setup and SeaweedFS (S3) manifests | Dockerfiles, Kubernetes or Compose manifests for your own services |
| Report templates | All report and README content |

## Save your progess

At the end of each task remember to push the relative tag, for example, once finished task 1 you will commit and push the `task1` tag.

```bash
git add -A && git commit -m "task1: final state at timebox"
git tag task1
git push origin main --tags
```

## Your environment at a glance

You need seven tools: Git, Docker, minikube, kubectl, Helm, uv and Python 3.12 (installed through uv). Install them at home and run the checks in *Verify your setup* before Part 1; no installation scripts are provided.

**Minimum machine:** 4 CPU cores, 16 GB RAM, 50 GB free disk (recommended), 64-bit OS, no GPU needed. Supported systems: Windows 10 22H2 or Windows 11 with hardware virtualisation enabled, macOS 14 or later (Intel or Apple silicon), Ubuntu 22.04 or 24.04 LTS.

| Tool | Version | What you use it for |
| --- | --- | --- |
| Git | latest | Version control, commits every 30 min, task tags |
| Docker | Docker Desktop (Windows, macOS) or Docker Engine (Linux), latest | Building images; also runs the minikube node |
| minikube | latest | Single-node local Kubernetes cluster |
| kubectl | within one minor version of the cluster | Talking to the cluster: apply manifests, inspect pods, read logs |
| Helm | 3.x or 4.x | Installing packaged services such as SeaweedFS |
| uv | latest | Installing Python, managing virtual environments and dependencies |
| Python | 3.12 | Analysis, app, model and agent code |

You also need a GitHub account and an editor (VS Code is a good default). You do **not** need to install an LLM: the endpoint and a personal API key for Task 3 are provided.

&#91;embedded content: your local environment · host tools, Docker, minikube, LLM endpoint\]

You drive the cluster from host tools; Docker runs the minikube node, where SeaweedFS and your own services run as pods. Only the LLM endpoint sits outside your laptop.

## Setup on Windows

On Windows, Docker Desktop runs on WSL 2, and every other tool installs with `winget` from PowerShell. Plan about 45 minutes, including up to one reboot.

1. **Check for WSL 2 and enable it if needed.** Open PowerShell as Administrator and run:

   ```powershell
   wsl --version
   ```

   **If it prints version details** (for example `WSL version: 2.x.x`), WSL is already installed. Run `wsl --update` to make sure it's current, then go straight to step 2. No reboot is needed.

   **If you get an error or a list of usage options instead**, WSL is missing or outdated. Install it, then reboot:

   ```powershell
   wsl --install
   wsl --update
   ```

   If the install reports that virtualisation is disabled, enable Intel VT-x or AMD-V in your BIOS/UEFI and run the commands again. You can check this beforehand in Task Manager → Performance → CPU, where "Virtualization" should say *Enabled*.

2. **Install the tools.** Close VS Code, Git Bash and any other app that uses Git, then open a normal PowerShell window and run:

   ```powershell
   winget install -e --id Git.Git
   winget install -e --id Docker.DockerDesktop
   winget install -e --id Kubernetes.kubectl
   winget install -e --id Helm.Helm
   winget install -e --id astral-sh.uv
   winget install -e --id ezwinports.make   # GNU make, for the starter repo Makefile
   ```

   Accept the administrator prompts when they appear. If a tool is already installed, winget upgrades it instead.

   Then open PowerShell **as Administrator** and install minikube, which needs admin rights:

   ```powershell
   winget install -e --id Kubernetes.minikube
   ```

   If Git fails with `exit code: 4`, a program is still using Git. Close it and run the Git command again.

3. **Restart PowerShell.** Close every PowerShell window and open a new one so the new `PATH` entries take effect. Otherwise commands like `kubectl` or `uv` will say "not recognized".

   If a command is still "not recognized" later, reinstall that tool and restart PowerShell again. If it's Git and an upgrade failed earlier, the old version was removed before the error: close any app that uses Git, then reinstall from an Administrator PowerShell with `winget install -e --id Git.Git --force`.

   If a command prints an older version than winget installed, another copy comes first in your `PATH`. Run `where.exe <tool>` (for example `where.exe kubectl`) to see every copy. Docker Desktop includes its own `kubectl`, so an older `kubectl` version is expected and fine.

4. **Start Docker Desktop** from the Start menu (not with "Run as administrator"), accept the terms, and check that *Settings → General → Choose Virtual Machine Manager (VMM)* is set to **WSL 2**. Older versions show this as a *Use the WSL 2 based engine* checkbox, which should be ticked.

   If Docker Desktop says the *Privileged helper service is not running*, click *Start* and accept the administrator prompt. If it asks every time, run `Set-Service com.docker.service -StartupType Automatic` in an Administrator PowerShell.

   If Docker Desktop closes with `open \\.\pipe\docker_engine: Access is denied`, another program is using Docker's connection. Restart Windows and start Docker Desktop again. If you also have Podman Desktop or Rancher Desktop, quit them first, since they conflict with Docker Desktop.

5. **Give WSL enough memory.** With the WSL 2 engine, Docker's memory is set by WSL, not by Docker Desktop. Open the config file in Notepad:

   ```powershell
   notepad "$env:USERPROFILE\.wslconfig"
   ```

   If Notepad asks to create the file, choose *Yes*. Add the content below and save. Set `memory` to about half your computer's RAM, for example `8GB` on a 16 GB laptop:

   ```ini
   [wsl2]
   memory=8GB
   processors=4
   ```

   Then run `wsl --shutdown` and restart Docker Desktop. This stops Docker and any running containers, so do it before you start working.

6. **Install Python 3.12 through uv:**

   ```powershell
   uv python install 3.12
   ```

7. **Configure Git** (use the email of your GitHub account):

   ```powershell
   git config --global user.name "Your Name"
   git config --global user.email "you@example.com"
   ```

Then continue with *Verify your setup*.

**Shell scripts:** Git for Windows includes *Git Bash*. Use it for any `.sh` script in the starter repository.

**Prefer Linux tools?** You can work inside an Ubuntu distribution in WSL. If you had WSL before this setup, you may not have Ubuntu yet: check with `wsl --list`, and install it with `wsl --install -d Ubuntu` if it's missing. Keep Docker Desktop on Windows, enable *Settings → Resources → WSL integration* for Ubuntu, then follow the Ubuntu section inside WSL, skipping the Docker Engine step. Keep your repository inside the WSL file system (`~/`), not under `/mnt/c`, or file access becomes slow.

**Licence note:** Docker Desktop is free for personal use, education and small businesses.

## Setup on macOS

On macOS everything installs with Homebrew in about 20 minutes. The same commands work on Intel and Apple silicon Macs.

1. **Install Homebrew** if `brew --version` fails. Run the installer, then run the two *Next steps* lines it prints to add `brew` to your `PATH`. It also installs the Xcode Command Line Tools, which include `git` and `make`.

   ```bash
   /bin/bash -c "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/HEAD/install.sh)"
   ```
2. **Install the tools:**

   ```bash
   brew install git minikube kubectl helm uv
   brew install --cask docker-desktop
   ```
3. **Start Docker Desktop** from Applications and accept the terms. In *Settings → Resources*, set at least 4 CPUs and 8 GB of memory, then *Apply & restart*.
4. **Install Python 3.12 through uv:**

   ```bash
   uv python install 3.12
   ```
5. **Configure Git** (use the email of your GitHub account):

   ```bash
   git config --global user.name "Your Name"
   git config --global user.email "you@example.com"
   ```

Then continue with *Verify your setup*.


## Setup on Linux (Ubuntu)

On Ubuntu you install Docker Engine from Docker's own apt repository and the other tools as single binaries, in about 20 minutes. Commands are for x86-64; on ARM64 machines replace `amd64` with `arm64`.

1. **Base packages:**

   ```bash
   sudo apt update
   sudo apt install -y git make curl ca-certificates
   ```
2. **Docker Engine.** Do not use the snap or Ubuntu's `docker.io` package; remove them first if installed.

   ```bash
   sudo install -m 0755 -d /etc/apt/keyrings
   sudo curl -fsSL https://download.docker.com/linux/ubuntu/gpg -o /etc/apt/keyrings/docker.asc
   sudo chmod a+r /etc/apt/keyrings/docker.asc
   echo "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.asc] https://download.docker.com/linux/ubuntu $(. /etc/os-release && echo "$VERSION_CODENAME") stable" | sudo tee /etc/apt/sources.list.d/docker.list > /dev/null
   sudo apt update
   sudo apt install -y docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin
   ```
3. **Run Docker without sudo.** minikube's Docker driver refuses to run as root, so add yourself to the `docker` group, then log out and back in.

   ```bash
   sudo usermod -aG docker $USER
   ```
4. **minikube:**

   ```bash
   curl -LO https://github.com/kubernetes/minikube/releases/latest/download/minikube-linux-amd64
   sudo install minikube-linux-amd64 /usr/local/bin/minikube && rm minikube-linux-amd64
   ```
5. **kubectl:**

   ```bash
   curl -LO "https://dl.k8s.io/release/$(curl -L -s https://dl.k8s.io/release/stable.txt)/bin/linux/amd64/kubectl"
   sudo install -o root -g root -m 0755 kubectl /usr/local/bin/kubectl && rm kubectl
   ```
6. **Helm:**

   ```bash
   curl -fsSL https://raw.githubusercontent.com/helm/helm/main/scripts/get-helm-4 | bash
   ```
7. **uv and Python 3.12.** Open a new terminal after installing uv so it is on your `PATH`.

   ```bash
   curl -LsSf https://astral.sh/uv/install.sh | sh
   uv python install 3.12
   ```
8. **Configure Git** (use the email of your GitHub account):

   ```bash
   git config --global user.name "Your Name"
   git config --global user.email "you@example.com"
   ```

Then continue with *Verify your setup*.

## Verify your setup

Your setup is ready when every command below succeeds and the minikube node reports `Ready`. The commands are the same on all three systems (PowerShell, macOS Terminal or Linux shell).

1. **Check versions.** Every command should print a version, not an error.

   ```bash
   git --version
   docker version
   docker compose version
   minikube version
   kubectl version --client
   helm version
   uv --version
   make --version
   ```
2. **Check Docker can run containers:**

   ```bash
   docker run --rm hello-world
   ```
3. **Start the cluster** (the first start downloads about 1 GB, so do it at home):

   ```bash
   minikube start --driver=docker --cpus=4 --memory=6144
   kubectl get nodes          # STATUS should be Ready
   kubectl get pods -A        # system pods Running
   helm list -A               # empty list, no error
   ```
4. **Deploy and reach a test service**, then remove it. On Windows and macOS, `minikube service` keeps the terminal busy while the tunnel is open; press Ctrl+C when done.

   ```bash
   kubectl create deployment hello --image=nginx:alpine
   kubectl expose deployment hello --port=80 --type=NodePort
   minikube service hello --url
   kubectl delete service,deployment hello
   ```
5. **Check Python and uv** in a throwaway project:

   ```bash
   uv init env-check --python 3.12
   cd env-check
   uv add pandas
   uv run python -c "import sys, pandas; print(sys.version, pandas.__version__)"
   ```
6. **Run the starter repository check** once you have cloned it: `make check`.
7. **Pre-pull images and data.** Pull the base images you plan to use (for example `docker pull python:3.12-slim`) and any images listed in the starter repository README, and download your dataset.

### Getting your own images into minikube

minikube runs its own container runtime, so an image you build with plain `docker build` is not visible to the cluster. Use one of these:

- `minikube image build -t myapp:dev .` builds straight into the cluster.
- `minikube image load myapp:dev` copies an image you already built.

In your manifests, set `imagePullPolicy: IfNotPresent` (or `Never`) for local images, and use a tag other than `latest`.

### Stopping and resetting

- `minikube stop` pauses the cluster and keeps its state; `minikube start` resumes it.
- `minikube delete` wipes the cluster completely. Use it only when you want a clean slate.

## Official installation docs

If a command in this guide stops working, the official pages below have the current version.

- [Docker Engine on Ubuntu](https://docs.docker.com/engine/install/ubuntu/) and [Docker Desktop (Homebrew cask)](https://formulae.brew.sh/cask/docker-desktop)
- [minikube start](https://minikube.sigs.k8s.io/docs/start/)
- [Install kubectl on Linux](https://kubernetes.io/docs/tasks/tools/install-kubectl-linux) (macOS and Windows pages linked from there)
- [Installing Helm](https://helm.sh/docs/intro/install/)
- [Installing uv](https://docs.astral.sh/uv/getting-started/installation/)

