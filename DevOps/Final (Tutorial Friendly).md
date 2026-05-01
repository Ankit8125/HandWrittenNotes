# DevOps Master Guide

**From Zero to Production: a complete, beginner-first walkthrough of the modern DevOps toolchain**

---
## 📖 Foreword — How to read this guide

This guide is built directly from Ankit's raw notes (Parts 1, 2, 3, 4).

Reading rules:
1. **Read top to bottom on the first pass.** Topics build on each other — Docker is needed for CI/CD, CI/CD is needed for Jenkins, Jenkins introduces the same socket trick we re-use in Kubernetes, etc. Skipping ahead leaves holes.
2. **Every section follows the same teaching pattern:**
    - First, the **problem** we are facing (why does this tool need to exist?)
    - Then the **manual / old-school approach** (what would we do without it?)
    - Then the **tool's solution** (how it fixes the problem)
    - Then a **high-level overview** (the architecture in plain English)
    - Then a **low-level walkthrough** (commands, code, configs — heavily commented)
    - Then a **practical MERN example** (so you see it work in a real stack)
    - Then **common mistakes & interview questions**
3. **Code blocks have inline comments** explaining what each line does..
4. **`🔁 Recap:` boxes** intentionally repeat earlier ideas. Repetition is a teaching tool, not laziness — concepts return because they're load-bearing later.
5. **`⚙️ Added Professional Context`** boxes mark content not in your raw notes but standard knowledge for an industry DevOps engineer.

---
## 🗺️ Master Table of Contents

```
SECTION 0 — INTRODUCTION & THE BIG PICTURE
   0.1  What is DevOps? (the problem it solves)
   0.2  Lifecycle of DevOps — the 7 Pillars
   0.3  The complete tool ecosystem (why every category exists)
   0.4  How the parts connect end-to-end

SECTION 1 — DEVOPS FUNDAMENTALS & CORE TOOLS
   1.1  Linux in DevOps
        1.1.1  Why Linux
        1.1.2  Quick command reference
        1.1.3  User management
        1.1.4  SSH (how it works + key-based auth)
        1.1.5  Process management
        1.1.6  File & permission management (chmod, chown, umask)
        1.1.7  Disk management (df, du, mount, fdisk)
   1.2  Shell Scripting
   1.3  Git & GitHub
   1.4  Environment Management
   1.5  Docker
   1.6  Docker Compose
   1.7  Continuous Integration (CI) — the philosophy
   1.8  GitHub Actions, ESLint and Prettier
   1.9  AWS EC2 Deployment
   1.10 Continuous Deployment (CD)
   1.11 Kubernetes (introduction)

SECTION 2 — DEEP DIVE INTO DOCKER & JENKINS
   2.1  Docker Volumes (persistence deep dive)
   2.2  Docker Networking (bridges, DNS, the localhost trap)
   2.3  Docker Registry
   2.4  Docker Volumes + Networking + Registry — combined MERN example
   2.5  Jenkins Intro
   2.6  Jenkins Flow Architecture
   2.7  Jenkins Setup with Docker and Docker Compose
   2.8  Jenkins Admin Setup and Plugin Installation
   2.9  Jenkins Web UI Walkthrough
   2.10 Jenkins Freestyle Job
   2.11 Jenkins Pipeline & MERN Implementation

SECTION 3 — ADVANCED ORCHESTRATION & NETWORKING
   3.1  Docker — Container-to-Container Communication
   3.2  NGINX
   3.3  Kubernetes Declarative
   3.4  Configuration and Secrets in Kubernetes
   3.5  Ingress
   3.6  Storage and Persistence in Kubernetes

SECTION 4 — INFRASTRUCTURE, AUTOMATION & MONITORING
   4.1  Tools Overview & Project Architecture
   4.2  Sublyzer Integration
   4.3  Terraform
   4.4  Ansible
   4.5  Caddy
   4.6  Monitoring and Uptime Kuma
   4.7  Prometheus and Grafana
   4.8  Grafana k6 — Load Testing

BONUS — INDUSTRY DEPTH (added context, not in raw notes)
   B.1  Latency, p50/p95/p99 — what you actually optimize for
   B.2  Alerting — how alerts are sent when a server goes down
   B.3  ArgoCD & GitOps
   B.4  Helm — package manager for Kubernetes
   B.5  Logging stacks: ELK / EFK
   B.6  APM & distributed tracing (Jaeger, Tempo, OpenTelemetry)
   B.7  Datadog & Splunk
   B.8  AWS networking deep (VPC, subnets, NACL, SG, NAT, Bastion)
   B.9  Cloud comparison (AWS vs GCP vs Azure)
   B.10 Security essentials (firewalls, secrets, image scanning, RBAC)
   B.11 Master interview question bank
   B.12 Master command cheat-sheet
```

---
---
# SECTION 0 — INTRODUCTION & THE BIG PICTURE
## 0.1 — What is DevOps? (And what was the world like before it?)

Before naming a single tool, we need to understand the **problem** DevOps was invented to solve. Tools without context are just noise.
### The world before DevOps

In a typical company in the early 2000s:

- **Developers** wrote code on their laptops.
- When code was "done," they zipped it up and emailed/uploaded it to a separate **Operations team** ("Ops").
- Ops would then SSH into a production server, manually install dependencies, copy files, restart services, and pray nothing broke.
- The two teams sat in different rooms with different KPIs (Key Performance Indicators):
    - **Devs** were rewarded for _shipping features fast_.
    - **Ops** were rewarded for _keeping production stable_.

These goals are in direct conflict. Devs wanted to push 10 changes a day. Ops wanted to push _zero_ (every change risks an outage). So they hated each other. Releases happened once every 3 months, took whole weekends, and routinely broke production.

This was called the **"throw it over the wall"** problem.
![[Pasted image 20260430201716.png | 400]]
### Classic symptoms of the pre-DevOps era

1. **"Works on my machine"** — code that ran fine on the dev's laptop crashed on the server because the server had a different Node.js version, a different OS, or a missing library.
2. **Manual deployments** — someone had to remember the _exact_ sequence: `git pull`, `npm install`, edit `.env`, `pm2 restart`. Forget step 4 → outage. Type the wrong DB URL → data loss.
3. **No visibility** — if the site went down at 3 AM, nobody knew until customers complained on Twitter.
4. **No rollback** — if a deploy went bad, "rollback" meant manually undoing every step.
5. **Snowflake servers** — every server in the company was slightly different, because they were configured manually by different people on different days. Reproducing them was impossible.
6. **Friction = fewer releases = more bugs per release.** Because devs deployed less often, every deploy carried _more_ changes, which made every deploy _more_ dangerous, which led to _fewer_ deploys. A vicious cycle.
### What DevOps actually is

> **DevOps = Development + Operations**

But more importantly, it is a **cultural + technical movement** that says:

> "The same team owns the code from `git push` all the way to running in production. The whole pipeline must be **automated**, **version-controlled**, **observable**, and **reversible**."

It is **not a job title.** It is **not a tool.** It is a _philosophy_ that produces a set of _practices_ which require a set of _tools_.

The three layers, in order:

```
    PHILOSOPHY            "Devs and Ops share ownership; automate everything"
        ↓
    PRACTICES             CI, CD, IaC, Monitoring, Configuration as Code
        ↓
	  TOOLS                 Git, Docker, Jenkins, Terraform, Prometheus, etc.
```

When somebody says "I'm a DevOps engineer," they really mean: _I implement the practices in the middle layer using the tools in the bottom layer, in service of the philosophy at the top._
### The benefits, concretely

| Pre-DevOps                             | DevOps                                         |
| -------------------------------------- | ---------------------------------------------- |
| Quarterly releases                     | Multiple deploys per day                       |
| Manual deploys (hours)                 | Automated deploys (minutes)                    |
| Outages discovered by users            | Outages caught by alerts before users notice   |
| Rollback = panic                       | Rollback = one button                          |
| "How is this server configured?" → ??? | Server config is in Git, version-controlled    |
| Incident postmortem = blame            | Incident postmortem = fix the system           |
| One environment differs from another   | Dev = staging = prod (identical, reproducible) |

---
## 0.2 — Lifecycle of DevOps — The 7 Pillars

Every DevOps process follows the same loop. **Memorize it.** Every interview question about "your CI/CD experience" is really just a question about _which tool you used in which pillar_.
![[Pasted image 20260430202124.png | 400]]
```
  ┌──────────────────────────────────────────────────────────┐
  │                                                          │
  │   ① Plan ──→ ② Develop ──→ ③ Build ──→ ④ Test            │
  │                                                  ↓       │
  │   ⑦ Monitor ←── ⑥ Deploy ←── ⑤ Release ←─────────┘       │
  │       │                                                  │
  │       └─────────── (feedback loop) ──────→ ① Plan        │
  │                                                          │
  └──────────────────────────────────────────────────────────┘
```

|#|Pillar|What happens|Typical tools|
|---|---|---|---|
|1|**Plan**|Define what to build (tickets, roadmap, scope)|Jira, Linear, Notion|
|2|**Develop**|Write the actual code; manage versions|VS Code, Git, GitHub|
|3|**Build**|Compile/package code into a deployable artifact|Jenkins, GitHub Actions, Maven, npm, `docker build`|
|4|**Test**|Run automated tests, lint, security scans, load tests|Jest, ESLint, Trivy, k6, JUnit|
|5|**Release**|Tag a stable version, push artifact to a registry|Docker Hub, GHCR, JFrog Artifactory, Nexus|
|6|**Deploy**|Promote the artifact to staging/production servers|Ansible, ArgoCD, Helm, kubectl, GitHub Actions CD|
|7|**Monitor**|Track logs, metrics, traces, errors; alert on problems|Prometheus, Grafana, ELK, Datadog, Sublyzer|

The arrow back from Monitor to Plan is the most important arrow on the diagram. It is the **feedback loop**: production telemetry tells you what to fix and what to build next. A pipeline without that arrow is just a one-way conveyor belt; with it, it becomes a learning system.

> 🔁 **Recap:** Every tool you'll meet in this guide lives inside one of these 7 pillars. When something feels confusing later, ask "which pillar is this in?" — that single question usually clears the fog.

---
## 0.3 — The Complete Tool Ecosystem — Why Every Category Exists

Beginners look at job descriptions and see a wall of tools — Jenkins, Docker, Kubernetes, Terraform, Ansible, Prometheus, Grafana, ELK, ArgoCD, Helm, Datadog, etc. — and panic. The trick is to realize: every tool fits a **category**, and there are only ~7 categories.

```
┌─────────────────────────────────────────────────────────────┐
│                  DEVOPS TOOL CATEGORIES                     │
├─────────────────────────────────────────────────────────────┤
│                                                             │
│  1. SOURCE CONTROL  ────→  Git, GitHub, GitLab, Bitbucket   │
│                                                             │
│  2. BUILD / CI      ────→  Jenkins, GitHub Actions,         │
│                            GitLab CI, CircleCI, Travis      │
│                                                             │
│  3. ARTIFACT STORE  ────→  Docker Hub, GHCR, ECR,           │
│                            JFrog Artifactory, Nexus         │
│                                                             │
│  4. DEPLOYMENT      ────→  Docker / Compose, Kubernetes,    │
│                            Helm, ArgoCD, Spinnaker          │
│                                                             │
│  5. INFRA AS CODE   ────→  Terraform, Pulumi,               │
│     (provisioning)         CloudFormation                   │
│                                                             │
│  6. CONFIG MGMT     ────→  Ansible, Chef, Puppet, SaltStack │
│                                                             │
│  7. OBSERVABILITY   ────→  Prometheus + Grafana,            │
│     ├─ Metrics             Datadog, New Relic               │
│     ├─ Logs                ELK / EFK, Splunk, Loki          │
│     ├─ Traces              Jaeger, Tempo, OpenTelemetry     │
│     └─ Alerting            Alertmanager, PagerDuty, Opsgenie│
│                                                             │
└─────────────────────────────────────────────────────────────┘

Cross-cutting concerns underneath all 7 categories:
   • CLOUD       (AWS, GCP, Azure)
   • NETWORKING  (VPC, Subnets, Load Balancers, DNS)
   • SECURITY    (Secrets, RBAC, image scanning, firewalls)
   • LINUX       (it all runs on Linux at the bottom)
```
### Why each category exists

**1. Source Control (Git/GitHub):** Without it, "the code" isn't really a thing — it's whatever is on someone's laptop. Source control gives you a single source of truth, history, branching, and the ability to roll back.

**2. Build / CI (Jenkins, GitHub Actions):** The instant you have a team, you need to _automatically_ build and test every code change. Manual builds get skipped, get done wrong, or get done on someone's laptop with weird env vars. CI removes the human.

**3. Artifact Store (Docker Hub / Artifactory / Nexus):** Once code is built, you need a place to store the _output_ — the Docker image, the JAR file, the npm tarball — so deployment systems can pull it. Without an artifact store, every deploy has to rebuild from source.

> ⚙️ **Added Professional Context — Artifactory and Nexus:** They're enterprise-grade artifact repositories that store **all** kinds of build outputs: Docker images, Maven JARs, npm packages, Python wheels, Helm charts, Debian packages, generic tarballs. Companies use them for two reasons:
> 
> 1. **Caching upstream registries** — instead of every CI build pulling from npmjs.com (which can rate-limit you and is a security risk), they pull from your internal Artifactory which has cached the package.
> 2. **Storing private artifacts** — a private internal library you don't want public. Docker Hub / GHCR / ECR are _Docker-image-specific_ registries. Artifactory and Nexus are _universal_. Treat them as the same idea, generalized.

**4. Deployment (Docker / Kubernetes / Helm / ArgoCD):** After building and storing the artifact, _something_ has to actually run it on a server. That's deployment. Docker runs one container; Compose runs a few containers on one machine; Kubernetes runs thousands of containers across many machines.

**5. Infrastructure as Code — IaC (Terraform):** Before you can deploy _to_ a server, the server has to _exist_. IaC tools provision servers, networks, DNS records, load balancers — all from text files in Git, instead of clicking buttons in a cloud console.

**6. Configuration Management (Ansible / Chef / Puppet):** Once a server exists, it needs to be configured — Docker installed, app folders created, environment variables set. Config management tools turn a raw provisioned server into a "ready to run the app" server.

> 💡 **The Terraform vs Ansible question (one of the most common interview questions):**
> 
> - **Terraform** = _provisioning_ = "what should exist?" (the server, the network, the DNS record). Talks to **cloud APIs**.
> - **Ansible** = _configuration_ = "what should be installed and configured on it?" (Docker, the app, the secrets). Talks to **the server over SSH**. The output of Terraform (the new server's IP) becomes the input to Ansible (the inventory target).

**7. Observability (Prometheus, Grafana, ELK, Datadog):** Once your app is running, you need to see what it's doing — its CPU, its errors, its response times, its logs. Without observability you are flying blind. We'll dedicate a huge chunk of this guide to splitting observability into its three sub-pillars: **Metrics**, **Logs**, **Traces**, plus **Alerting**.
![[Pasted image 20260430203542.png | 1000]]

---
## 0.4 — How the Parts Connect End-to-End

Here is the whole pipeline you will eventually build. Don't worry if names are unfamiliar — every box is covered later. Save this diagram; you'll keep coming back to it.
```
   ┌──────────────────────────────────────────────────────────────┐
   │                        DEVELOPER LAPTOP                      │
   │                                                              │
   │   [Code in VS Code]  ──→  git commit  ──→  git push          │
   └──────────────────────────────────────────────────────────────┘
                                   │
                                   ▼
   ┌──────────────────────────────────────────────────────────────┐
   │                            GITHUB                            │
   │                                                              │
   │   PR opened  ──→  GitHub Actions runs CI                     │
   │                  (lint + test + docker build)                │
   │                                                              │
   │   PR merged to main  ──→  triggers CD job                    │
   └──────────────────────────────────────────────────────────────┘
                                   │
                                   ▼
   ┌──────────────────────────────────────────────────────────────┐
   │                      ARTIFACT REGISTRY                       │
   │                                                              │
   │   Built Docker image is pushed to Docker Hub / GHCR / ECR    │
   └──────────────────────────────────────────────────────────────┘
                                   │
                                   ▼
   ┌──────────────────────────────────────────────────────────────┐
   │                     INFRASTRUCTURE LAYER                     │
   │                                                              │
   │   Terraform has already provisioned the VPS / EC2 instance   │
   │   Ansible has already installed Docker + Git + setup files   │
   └──────────────────────────────────────────────────────────────┘
                                   │
                                   ▼
   ┌──────────────────────────────────────────────────────────────┐
   │                   THE PRODUCTION SERVER                      │
   │                                                              │
   │   docker compose up -d --build       ← deployment            │
   │      │                                                       │
   │      ├── caddy / nginx (port 80/443) ← reverse proxy         │
   │      ├── frontend container                                  │
   │      ├── backend container                                   │
   │      └── mongo container (persistent volume)                 │
   └──────────────────────────────────────────────────────────────┘
                                   │
                                   ▼
   ┌──────────────────────────────────────────────────────────────┐
   │                      OBSERVABILITY LAYER                     │
   │                                                              │
   │   Prometheus scrapes node-exporter / cAdvisor / blackbox     │
   │      every 15s   →   stores time-series data                 │
   │                                                              │
   │   Grafana queries Prometheus    →   renders dashboards       │
   │   Uptime Kuma probes endpoints  →   alerts on outage         │
   │   Sublyzer captures app errors  →   AI suggests fixes        │
   │   k6 simulates traffic          →   validates capacity       │
   │                                                              │
   │   Alertmanager (or PagerDuty/Slack) → notifies on-call eng   │
   └──────────────────────────────────────────────────────────────┘
```

That single diagram is the entire job description of a DevOps engineer. The rest of this guide is unpacking each box.
![[Pasted image 20260430204007.png | 600]]

---
---
# SECTION 1 — DEVOPS FUNDAMENTALS & CORE TOOLS

We start with the foundations. Every fancy tool above runs on top of Linux, version-controlled in Git, configured through environment variables. Get these solid first.

---
## 1.1 — Linux in DevOps
### 1.1.1 Why Linux?

Question first: why does _every_ DevOps tool you'll meet (Docker, Kubernetes, Jenkins, Terraform, Ansible, Prometheus, Nginx, Caddy) run on Linux? And why are 95%+ of cloud servers Linux machines?

Reasons:

1. **Free and open-source** — no licensing cost per server. At a 10,000-server scale, this is decisive.
2. **Stable** — Linux servers commonly run for years without reboot.
3. **Designed for servers** — Linux was built around the multi-user, multi-process server model from day one.
4. **Container kernels are Linux kernels.** A Docker container is fundamentally a Linux process with namespaces and cgroups around it. There's no such thing as a "Windows-native Docker container" in the same lightweight way — Docker on Windows secretly runs a tiny Linux VM under the hood.
5. **Scriptable end-to-end** — every operation in Linux can be done from a terminal, which means it can be scripted, which means it can be automated, which is what DevOps _is_.

So: if you can't navigate Linux, you can't do DevOps. This sub-section turns you into someone comfortable on a Linux server.
### 1.1.2 Accessing Linux on Windows (WSL)

If you're on Windows, the best way to learn Linux is **WSL — Windows Subsystem for Linux** — which gives you a real Ubuntu shell inside Windows.

```bash
wsl --install              # Installs Ubuntu by default (the WSL2 version)
wsl --list --verbose       # See what distros are installed and their version
wsl                        # Start the default Ubuntu shell
```

> 💡 In WSL, your Windows D:\ drive appears as `/mnt/d/`. So `D:\Coding\app` becomes `/mnt/d/Coding/app` from inside Ubuntu. We use this exact convention later when running Ansible.
### 1.1.3 Linux Quick-Reference Table

The minimum daily-driver vocabulary. Memorize these — they appear in every screencast, every Stack Overflow answer, every Dockerfile.

|Command|Purpose|
|---|---|
|`pwd`|"Print working directory" — where am I right now?|
|`ls`|List files & folders in the current directory|
|`ls -l`|Long listing — permissions, size, owner, modified date|
|`ls -a`|Show hidden files (those starting with `.`, like `.env`, `.gitignore`)|
|`ls -lah`|Long + hidden + human-readable sizes (KB/MB instead of raw bytes)|
|`cd <dir>`|Change directory|
|`cd ~`|Go to your home directory (`~` = home)|
|`cd /`|Go to root of filesystem|
|`cd -`|Jump back to your previous directory|
|`touch <file>`|Create an empty file (or update its modification time if it exists)|
|`rm <file>`|Delete a file. **No recycle bin** — this is permanent.|
|`rm -r <dir>`|Recursively delete a directory and everything inside|
|`rm -rf <dir>`|Same, but force (no confirmation). Be very careful — `rm -rf /` will wipe a server.|
|`mkdir <dir>`|Create a directory|
|`mkdir -p a/b/c`|Create a nested chain of directories|
|`rmdir <dir>`|Delete a directory only if it's empty|
|`cp file1 file2`|Copy file1 to file2|
|`cp -r dir1 dir2`|Recursively copy a directory|
|`mv file newloc/`|Move (or rename) a file|
|`cat <file>`|Print the entire file contents to the terminal|
|`head -20 <file>`|Print first 20 lines|
|`tail -20 <file>`|Print last 20 lines|
|`tail -f <file>`|"Follow" a file — print new lines as they arrive (used for live logs)|
|`find <path> -name "<pattern>"`|Recursively search for filenames|
|`grep "<text>" <file>`|Search for a string inside a file|
|`grep -r "<text>" .`|Recursively grep through every file in the current directory|
|`vi <file>` / `nano <file>`|Edit a file in place. `nano` is friendlier; `vi`/`vim` is more powerful but has a learning curve.|
|`history`|Show your previously executed commands|
|`chmod +x <file>`|Make a file executable (covered in detail in 1.1.6)|
|`clear` (or Ctrl+L)|Clear the terminal screen|
|`man <command>`|Show the manual page for a command. `man ls` is your best friend.|

Now we go deeper into the five Linux topics your senior specifically called out.
### 1.1.4 User Management

A Linux server is **multi-user**. Every running process belongs to a user; every file has an owning user. So managing users is non-negotiable for a server administrator.

**Why does it matter for DevOps?**

- The application should run as a **non-root** user (so a bug can't take over the whole system).
- Different teammates SSH in as different users — auditable.
- Some tools (Jenkins, Ansible) need their own dedicated user.
- The classic `ec2-user`, `ubuntu`, `root` users you've seen in cloud examples are exactly this concept.

**Key commands:**

```bash
# Show who you currently are
whoami                    # → e.g., "ec2-user" or "root"

# Show every user logged in right now
who

# Show a user's UID, GID, group memberships
id ankit

# Create a new user named "deploy"
sudo useradd -m -s /bin/bash deploy
#       │  │  │
#       │  │  └── shell to use after login
#       │  └───── create their home directory at /home/deploy
#       └──────── needs sudo because we're modifying the system

# Set a password for them
sudo passwd deploy

# Delete a user (and their home dir)
sudo userdel -r deploy

# Add an existing user to a group (e.g., the docker group so they can run docker without sudo)
sudo usermod -aG docker deploy
#                │  │
#                │  └── group name
#                └───── -a = append, -G = supplementary groups
#                       (without -a, you'd OVERWRITE their other groups)

# List all groups a user belongs to
groups deploy

# Switch to another user
su - deploy               # "su" = "switch user". The "-" gives them a fresh login shell.

# Run a single command as another user (or as root)
sudo apt update           # the most common form: run one command as root
sudo -u deploy whoami     # run "whoami" as the user "deploy"
```

**Where users are stored:**

|File|Contents|
|---|---|
|`/etc/passwd`|Every user account (username, UID, home dir, shell). World-readable.|
|`/etc/shadow`|Hashed passwords. Readable only by root.|
|`/etc/group`|Every group and its members.|
|`/etc/sudoers`|Who is allowed to run `sudo`. Edit ONLY with `visudo`, never directly.|

**The `root` user vs `sudo`:**

- `root` (UID 0) is god. Every command runs unchecked. Don't log in as root for daily work.
- A regular user with `sudo` privileges can elevate a single command to root: `sudo apt install nginx`. This is the safer pattern.

> 💡 **Real DevOps gotcha:** When you spin up a brand new Ubuntu EC2 instance, you log in as `ubuntu` (not `root`). When you spin up Amazon Linux, you log in as `ec2-user`. Both have passwordless `sudo`. Memorize this — first-time SSH failures are usually wrong-username failures.
### 1.1.5 SSH — Secure Shell

**SSH** is _the_ protocol for remotely controlling a Linux server. Every cloud server, every Hostinger VPS, every Kubernetes node — you reach them all through SSH.

#### How SSH works (high level)

```
   Your laptop                                    Remote server
   ───────────                                    ─────────────
   [SSH client]  ──── encrypted TCP ──→ port 22  [SSH daemon (sshd)]
                       (TLS-like)
                                                  authenticates you,
                                                  drops you into a shell
```

When you type `ssh ankit@server.example.com`:

1. Your SSH client connects to port 22 on the server.
2. The server presents its public host key. Your client checks if it has seen this key before (`~/.ssh/known_hosts`). If new, it prompts: _"The authenticity of host can't be established. Continue connecting?"_ — answering "yes" stores the key for future trust.
3. Your client sends authentication credentials (either a password or a private key).
4. If the server accepts, you get a shell on the remote machine. Every keystroke is encrypted.
#### Password auth vs Key-based auth

**Password auth** is what you use the first time. It's discouraged for production:

- People pick weak passwords.
- Bots constantly brute-force port 22.
- If you have 50 servers, you have 50 passwords to remember (or, more likely, the same password reused everywhere — even worse).

**Key-based auth** is the gold standard. You generate a _key pair_:

- A **private key** stays on your laptop. NEVER leaves it. NEVER shared. This is your secret.
- A **public key** is placed on every server you want to access. Servers freely share it.

These keys are mathematically linked — anything signed by the private key can be verified by the public key, but you cannot derive one from the other.

**The login dance with keys:**

```
   Server:    "Prove you have the matching private key for this public
               key in /home/ankit/.ssh/authorized_keys."
   Laptop:    [signs a challenge using my private key]
   Server:    [verifies signature with the public key]  →  ✓ Login granted
```

No password ever crosses the wire. No password to brute-force.
#### Generating keys (the modern way)
```bash
# On your laptop:
ssh-keygen -t ed25519 -a 64 -C "ankit-laptop-2026"
#           │           │     │
#           │           │     └── comment / label embedded into the key
#           │           └─── number of key derivation rounds (higher = harder to brute-force the passphrase)
#           └─── algorithm: ed25519 is a modern elliptic-curve algo,
#                fast, short, secure. Older tutorials use RSA — still fine,
#                but ed25519 is preferred today.

# When prompted:
#   - Where to save (default: ~/.ssh/id_ed25519)        ← press Enter
#   - Optional passphrase                                ← strongly recommended

# Two files produced:
#   ~/.ssh/id_ed25519        ← PRIVATE KEY (never share)
#   ~/.ssh/id_ed25519.pub    ← PUBLIC KEY (give to servers)
```
#### Authorizing the public key on a server

The server needs your public key inside `~/.ssh/authorized_keys`. The simplest way:

```bash
# This copies your public key to the remote server and appends it to ~/.ssh/authorized_keys
ssh-copy-id -i ~/.ssh/id_ed25519.pub ankit@187.127.157.4
```

After this, you can log in passwordlessly:

```bash
ssh ankit@187.127.157.4    # no password prompt; the key authenticates you
```
#### SSH config file (the productivity unlock)

If you SSH into many servers, typing the full `ssh -i ~/.ssh/id_ed25519 ec2-user@3.91.x.x` every time is painful. Create `~/.ssh/config`:

```
Host prod-vps
    HostName 187.127.157.4
    User root
    IdentityFile ~/.ssh/id_ed25519
    Port 22

Host bastion
    HostName 54.231.10.20
    User ec2-user
    IdentityFile ~/.ssh/aws-bastion.pem
```

Now you just type `ssh prod-vps`. Done.
#### Where SSH appears in the rest of this guide

|Where|What for|
|---|---|
|AWS EC2 deployment|First step is `ssh -i keypair.pem ec2-user@public-ip`|
|GitHub Actions → EC2|The CD job uses `ssh-keyscan` + `ssh -i ec2_key.pem` to deploy|
|Ansible|Ansible is fundamentally "SSH automation with idempotency"|
|Hostinger VPS bootstrap|`ssh-copy-id` puts your public key on the VPS so Ansible can connect|
|`git clone git@github…`|GitHub itself uses SSH for authenticated git operations|

> 🔁 **Recap:** SSH is so central to DevOps that you'll meet it in literally every section. Master keys _now_; don't put it off.
### 1.1.6 Process Management

Every running program is a **process**. Understanding processes lets you debug "why is the server slow?", "what's eating my CPU?", "why did my app crash?".

```bash
# Show every process on the system
ps -ef          # "every" + "full format"
ps aux          # alternative; shows CPU/memory %

# Live, top-CPU-first view (refreshes every 2s). Press q to quit.
top

# Nicer, color-coded version of top (you may need to install: sudo apt install htop)
htop

# Find a process by name
ps -ef | grep node
pgrep -fa node     # cleaner — searches by name and shows full cmdline

# Kill a process by PID (Process ID)
kill 12345              # sends SIGTERM (polite — "please shut down")
kill -9 12345           # sends SIGKILL (force — "die now"). Last resort.
killall node            # kill every process named "node"

# How much CPU/memory a process uses
top -p 12345

# Run a command in the background
node server.js &
#               └── the "&" means "run in background and give me my prompt back"

# See background jobs
jobs

# Bring a job back to foreground
fg %1
```

**Signals (a key concept):**

When you "kill" a process you're really sending it a _signal_. Signals are how the OS tells processes things.

|Signal|What it means|Default behavior|
|---|---|---|
|`SIGTERM`|"Please terminate gracefully"|Process can clean up, then exit|
|`SIGKILL`|"Die immediately"|Cannot be caught/ignored|
|`SIGINT`|What Ctrl+C sends|Usually = stop the process|
|`SIGHUP`|Terminal closed (used to mean "reload config" by convention)|Varies|

Why this matters: when Docker or Kubernetes shuts down a container, it sends SIGTERM first, waits 10 seconds (`stop_grace_period`), then SIGKILL if the container hasn't exited. Apps that don't handle SIGTERM lose in-flight requests on every deploy. This is why production apps register signal handlers. 
- (A **signal handler** is code that listens for OS signals like SIGTERM.)
### 1.1.7 File and Permission Management

This is where Linux beginners get burned. Every file/folder in Linux has **owner**, **group**, and **other** permissions for **read**, **write**, **execute**.

When you do `ls -l`, you see something like:

```
-rwxr-xr--  1 ankit  developers  1234 Apr 28 10:00 deploy.sh
│└┬┘└┬┘└┬┘  │   │        │
│ │  │  │   │   │        └── group name
│ │  │  │   │   └─────────── owner name
│ │  │  │   └─────────────── number of hard links (don't worry about this)
│ │  │  └─────────────────── permissions for OTHER (everyone else):  r--
│ │  └────────────────────── permissions for GROUP:                  r-x
│ └───────────────────────── permissions for OWNER:                  rwx
└─────────────────────────── file type ("-" = file, "d" = directory, "l" = symlink)
```

`r` = read (4) | `w` = write (2) | `x` = execute (1)

So `rwxr-xr--` means:

- Owner: read+write+execute (4+2+1 = **7**)
- Group: read+execute (4+1 = **5**)
- Other: read only (**4**)

That's permission **`754`** — a number you'll see constantly.

**Common permission shorthand:**

|Numeric|Symbolic|Meaning|
|---|---|---|
|`755`|`rwxr-xr-x`|Owner can do anything, others can read+execute. Standard for executables and directories.|
|`644`|`rw-r--r--`|Owner read+write, others read-only. Standard for normal files.|
|`600`|`rw-------`|Owner read+write, NOBODY else. Used for **secret files** like SSH private keys, `.env`.|
|`700`|`rwx------`|Owner-only directory. Used for `~/.ssh`.|
|`777`|`rwxrwxrwx`|Everyone can do everything. **Almost always wrong.** A red flag in code review.|
#### Changing permissions: `chmod`

```bash
# Symbolic form
chmod +x script.sh         # add execute for everyone
chmod u+x script.sh        # add execute for OWNER only (u=user, g=group, o=other, a=all)
chmod g-w report.txt       # remove write from group
chmod o=r private.txt      # set OTHER to exactly read

# Numeric form (the one you'll see in tutorials and CI scripts)
chmod 755 deploy.sh
chmod 600 ~/.ssh/id_ed25519
chmod 644 server/.env

# Recursive — apply to a directory and everything inside
chmod -R 755 /opt/myapp
```

> ⚠️ **The most common SSH error in the world:** "Permissions on the keypair are too open." Cause: you copied a `.pem` file to your laptop and it's `644` — SSH refuses, because anyone could have read your private key. **Fix:** `chmod 600 mykey.pem`.
#### Changing ownership: `chown`

```bash
# Change owner of a file
sudo chown ankit deploy.sh

# Change owner AND group
sudo chown ankit:developers deploy.sh

# Recursively
sudo chown -R deploy:deploy /opt/myapp
```

A common pattern: after creating a directory as root, hand it over to your app's user:

```bash
sudo mkdir /opt/myapp
sudo chown -R deploy:deploy /opt/myapp
```
#### `umask` — the permission-default

When you create a new file with `touch`, the default permission isn't 666 (read+write for everyone) — it's actually 644 on most distros. Why? Because of `umask`, which subtracts bits from the default.
```bash
umask                  # prints your current umask (often "0022")
umask 022              # set it for this shell session
```

You almost never need to change this manually. Just know that `umask` exists when you see it referenced in production scripts.
### 1.1.8 Disk Management

Servers run out of disk space. When they do, _everything_ breaks — Docker can't pull images, logs can't be written, apps crash. So disk is something you monitor and manage.

```bash
# Show disk usage of every mounted filesystem
df -h
#  -h = human-readable (GB, MB)

# Sample output:
# Filesystem      Size  Used Avail Use% Mounted on
# /dev/xvda1       30G   18G   12G  60% /
# tmpfs           487M     0  487M   0% /dev/shm
# /dev/xvdf       100G   45G   55G  45% /var/lib/docker

# Show disk usage of a specific directory (recursively)
du -sh /var/log
#  -s = summarize (one number, not every file)
#  -h = human-readable

# What are the biggest things in /var ?
du -h /var | sort -rh | head -20

# Find files larger than 100 MB
find / -type f -size +100M 2>/dev/null

# What's mounted where?
mount | column -t

# Mount/unmount a disk
sudo mount /dev/xvdf /mnt/data
sudo umount /mnt/data

# List block devices
lsblk

# Inspect partitions on a disk (root only)
sudo fdisk -l
```
#### What fills up a server's disk in production?

1. **Application logs** in `/var/log` — never rotated, grow forever. Solution: configure `logrotate`.
2. **Docker** — old images, stopped containers, volumes. `/var/lib/docker` can balloon to dozens of GB.
    - Cleanup: `docker system prune -af --volumes`
3. **Database growth** in `/var/lib/mysql`, `/var/lib/mongodb`, etc.
4. **Build artifacts** from CI runners.
#### Linux filesystem hierarchy (the names you must know)
![[Pasted image 20260430222601.png | 700]]

```
/                  ← root of everything
├── bin/           ← essential binaries (ls, cp, mv)
├── etc/           ← system-wide config files (passwd, ssh, nginx config)
├── home/          ← user home directories (/home/ankit)
├── opt/           ← optional / 3rd-party software (we'll put apps here)
├── tmp/           ← temporary files (cleared on reboot)
├── usr/           ← user programs (most installed apps live in /usr/bin or /usr/local/bin)
├── var/           ← variable data — logs, databases, mail, docker, anything that grows
│   ├── log/       ← system & app logs
│   └── lib/docker ← Docker images, containers, volumes
├── proc/          ← virtual filesystem reflecting the kernel state (CPU, memory, processes)
└── sys/           ← virtual filesystem reflecting hardware
```

You'll see `/proc` and `/sys` get mounted into the **node-exporter** container later in the monitoring section — that's how the exporter reads the host's CPU/memory.

> 🔁 **Recap (Linux section):** A DevOps engineer is, at heart, a Linux power-user with extra tools on top. The five sub-skills above (commands, users, SSH, processes, permissions, disk) are 80% of what you'll do daily on a server. We'll layer Docker, Kubernetes, Ansible on top — but they all eventually call into Linux.

---
## 1.2 — Shell Scripting
### What problem are we solving?

You just learned a bunch of Linux commands. But what if you have to run the same 10 commands every time you set up a new server? Or every time you deploy?

You could:

- Do them manually each time (slow, error-prone, undocumented).
- Or write them once in a file and run that file. ← this is shell scripting.

A **shell script** is a plain text file containing a series of shell commands, run by a shell interpreter (usually `bash`). It's the simplest possible automation — and despite the existence of fancy tools like Ansible and Jenkins, simple shell scripts are _still_ used everywhere because they're trivial to write and review.
### The 5-step workflow

```bash
# 1. Create the script file
nano hello.sh

# 2. Inside the file, write:
#!/bin/bash
echo "Hello, DevOps!"

# 3. Save and exit nano:  Ctrl+O → Enter → Ctrl+X

# 4. Make it executable
chmod +x hello.sh
#       └── ↑ remember from 1.1.7: "+x" adds execute permission

# 5. Run it
./hello.sh
# Output: Hello, DevOps!
```

> 💡 **Why `#!/bin/bash`?** This is the **shebang** (`#!`) line. It tells the OS _which interpreter_ should run this script. Without it, the OS guesses based on your current shell — which might be `zsh`, `dash`, or something else, and your script may behave differently. Always include the shebang.
### Variables

```bash
#!/bin/bash

name="Ankit"
echo "Hello $name"

# Command substitution: capture output of a command into a variable
today=$(date +%Y-%m-%d)
echo "Today is $today"

# Read user input
read -p "What's your name? " username
echo "Welcome, $username"
```

**Quoting rules (these trip up beginners):**

- Single quotes `'...'` — literal, no variable expansion. `'Hello $name'` prints exactly that.
- Double quotes `"..."` — variables expand. `"Hello $name"` prints `Hello Ankit`.
- Backticks/`$()` — execute and substitute. `$(date)` runs date and inserts its output.
### Conditionals

```bash
#!/bin/bash
number=6

if [ $number -gt 5 ]; then
    echo "Number greater than 5"
elif [ $number -eq 5 ]; then
    echo "Exactly 5"
else
    echo "Number less than 5"
fi
```

Run:

```
$ chmod +x conditional.sh
$ ./conditional.sh
Number greater than 5
```

**Comparison operators (memorize):**

|Operator|Meaning|
|---|---|
|`-eq`|equal|
|`-ne`|not equal|
|`-gt`|greater than|
|`-lt`|less than|
|`-ge`|greater than or equal|
|`-le`|less than or equal|
|`=`|string equal|
|`!=`|string not equal|
|`-z "$x"`|string is empty|
|`-f file`|file exists|
|`-d dir`|directory exists|
### Loops

```bash
# For loop
for service in mongo backend frontend; do
    echo "Restarting $service..."
    docker restart $service
done

# While loop
count=1
while [ $count -le 5 ]; do
    echo "Count: $count"
    count=$((count + 1))
done
```
### Practical DevOps shell script examples
#### Example 1: Start a MERN app locally

```bash
#!/bin/bash
# start-mern.sh — start backend & frontend together
echo "Starting MERN app..."

# Start backend in the background (& sends to background)
cd server && npm start &

# Start frontend in the foreground
cd ../client && npm run dev
```

Why this matters: before CI/CD pipelines exist, devs use exactly this kind of script during local development.
#### Example 2: Bootstrap a fresh EC2 server

```bash
#!/bin/bash
# bootstrap.sh — first-time setup of a fresh Ubuntu server
set -e   # ← KEY LINE: exit immediately if any command fails (don't continue silently)

echo "=== Updating package list ==="
sudo apt update -y

echo "=== Installing Docker ==="
sudo apt install -y docker.io
sudo systemctl start docker
sudo systemctl enable docker            # start on boot

echo "=== Letting current user run docker without sudo ==="
sudo usermod -aG docker $USER

echo "=== Installing Git ==="
sudo apt install -y git

echo "=== Done. Please log out and log back in for group changes to apply. ==="
```

That single script encapsulates what would otherwise be 8 manual commands you'd run on every new server.

> ⚠️ **`set -e` is the most underused line in shell scripts.** Without it, if a command fails, the script keeps running anyway — and you end up with a half-broken server because step 4 succeeded "on top of" a failed step 3. ALWAYS include `set -e` at the top of any script that does multiple steps.
#### Example 3: Backup a MongoDB database

```bash
#!/bin/bash
set -e
TIMESTAMP=$(date +%Y%m%d-%H%M%S)        # stamp for the backup filename
BACKUP_DIR="/var/backups/mongo"
DB_NAME="prod_db"

mkdir -p "$BACKUP_DIR"

echo "Backing up $DB_NAME to $BACKUP_DIR/$DB_NAME-$TIMESTAMP.gz"
mongodump --db="$DB_NAME" --archive="$BACKUP_DIR/$DB_NAME-$TIMESTAMP.gz" --gzip

# Delete backups older than 30 days
find "$BACKUP_DIR" -name "*.gz" -mtime +30 -delete

echo "Backup complete."
```

This is the kind of script you put in `cron` to run nightly. Real production work.
### Where shell scripts fit (and where they break down)

**Shell scripts are great for:**

- One-off setup tasks
- Simple deployment glue
- Wrapping CI/CD step commands
- Cron jobs (scheduled tasks)

**Where they break down:**

- More than ~100 lines → hard to maintain
- Idempotency is hard (re-running may break things)
- Error handling is awkward
- Cross-platform portability is poor

This is exactly why we move to Ansible later — Ansible playbooks are like shell scripts but **idempotent, structured, and testable**. But shell scripts remain a daily tool.

> 🔁 **Recap:** Shell scripting is the gateway drug to automation. You'll see snippets of bash inside Dockerfiles, Jenkinsfiles, GitHub Actions, and Ansible. Get comfortable.

---
## 1.3 — Git & GitHub
### Git vs GitHub (different things, often confused)

| **Git**      | **GitHub**                                 |                                                  |
| ------------ | ------------------------------------------ | ------------------------------------------------ |
| What it is   | A distributed version-control system (VCS) | A cloud-hosted service built around Git          |
| Lives        | On your laptop / on every server           | On the internet (a website + backend)            |
| Purpose      | Tracks code changes, history, branches     | Stores Git repos, enables collaboration, runs CI |
| Created by   | Linus Torvalds (the Linux guy), 2005       | A company, 2008 (now owned by Microsoft)         |
| Alternatives | (none — Git is the standard)               | GitLab, Bitbucket, Gitea, Codeberg               |

> 💡 **Analogy:** Git is the _typewriter_ (a tool that lives on your desk). GitHub is the _post office_ (a service that lets you send your typed pages to others and collaborate).
### Why version control exists

Imagine three teammates editing the same `app.js` file. Without Git:

- Bob saves his changes → emails it to Alice.
- Alice has already changed the file. Whose version wins?
- Carol joins later — which version is "the latest"?

Git solves this with a **distributed model**: every developer has the _complete_ history of the project on their laptop, and Git knows how to merge their independent changes intelligently.
### Git Quick-Reference

|Command|What it does|
|---|---|
|`git init`|Turn the current folder into a Git repo (creates `.git/`)|
|`git status`|What's changed? What's staged? Run this constantly.|
|`git add <file>`|Stage a file for the next commit|
|`git add .`|Stage everything in current directory|
|`git commit -m "message"`|Snapshot the staged changes with a message|
|`git log`|Show commit history|
|`git log --oneline --graph --all`|Beautiful one-line history with branches|
|`git diff`|What unstaged changes exist?|
|`git diff --staged`|What's about to be committed?|
|`git remote add origin <url>`|Connect this repo to a remote (usually GitHub)|
|`git remote -v`|List configured remotes|
|`git push origin <branch>`|Upload local commits to GitHub|
|`git push -u origin main`|Push AND set upstream (do this once per branch)|
|`git pull`|Fetch + merge remote changes into current branch|
|`git fetch`|Download remote changes WITHOUT merging|
|`git branch`|List local branches|
|`git branch <name>`|Create a new branch|
|`git checkout <branch>`|Switch to a branch|
|`git checkout -b <branch>`|Create + switch in one step|
|`git switch <branch>`|Modern alternative to `checkout` for branches only|
|`git merge <branch>`|Merge another branch into current one|
|`git branch -M main`|Rename current branch to "main"|
|`git reset --hard origin/main`|Discard local changes, match remote main|
|`git stash`|Temporarily shelve uncommitted changes|
|`git stash pop`|Reapply stashed changes|
|`git clone <url>`|Download a remote repo to your machine|
### The standard workflow (memorize this loop)

```bash
# 1. Make sure you're up-to-date with main
git checkout main
git pull

# 2. Create a feature branch
git checkout -b feature/add-login

# 3. Work on the feature, save files...

# 4. See what changed
git status
git diff

# 5. Stage and commit
git add .
git commit -m "feat: add user login endpoint"

# 6. Push the feature branch to GitHub
git push origin feature/add-login

# 7. On GitHub, open a Pull Request: feature/add-login → main
# 8. Teammates review, comments addressed, CI checks pass
# 9. Merge the PR; main is now up-to-date with your feature
```

This loop is repeated _thousands_ of times in your career. Burn it in.
### Branching strategy — why `main` is sacred

Rule: **never push directly to `main`.** All work happens on feature branches.

```
   main      ●────●────●────────●────●────●           ← always deployable
              \    \             \    \   ↑
               \    \             \    \  └─ merge after PR review
                ●─●─●               ●─●─●
                feature/login        feature/payments
```

Why?

- Multiple developers can work in parallel without stepping on each other.
- `main` always represents code that's safe to deploy.
- Pull Requests force code review, which catches bugs.
- PRs trigger **CI checks** (Section 1.7) — tests, linters — _before_ the bad code can pollute `main`.
- **Branch protection rules** on GitHub literally block the "Merge" button if CI fails. We'll set this up later.

> ⚙️ **Added Professional Context — branching strategies:** Three are common in industry:
> 
> 1. **Git Flow** (Vincent Driessen, 2010): main, develop, feature/_, release/_, hotfix/* branches. Heavyweight; suited to scheduled releases.
> 2. **GitHub Flow**: main + short-lived feature branches. Each merge to main can deploy. Lightweight; what most modern teams use.
> 3. **Trunk-based development**: everyone commits to `main` (or close to it) constantly, behind feature flags. Used at high-velocity companies (Google, Facebook). Most courses (and this one) use **GitHub Flow**.
### .gitignore — what NOT to push

GitHub is a _public-facing_ place (or at least a shared place). You must NOT commit:

- **Secrets** — `.env` files, API keys, private keys, certificates
- **Dependencies** — `node_modules/`, `vendor/`, `__pycache__/` (huge, regeneratable)
- **Build outputs** — `dist/`, `build/`, `*.log`
- **OS junk** — `.DS_Store` (macOS), `Thumbs.db` (Windows)
- **IDE files** — `.idea/`, `.vscode/` (personal config)

A typical Node.js `.gitignore`:

```gitignore
node_modules/
.env
.env.local
.env.*.local
dist/
build/
.DS_Store
*.log
.vscode/
.idea/
coverage/
```

> ⚠️ **The single most common DevOps disaster:** committing `.env` to a public repo. Bots scan GitHub _constantly_ for leaked AWS keys; within minutes, miners spin up $10,000 of EC2 on your card. **Always** put `.env` in `.gitignore` BEFORE the first commit, not after.
### Connecting a local repo to GitHub (the first-push ritual)

```bash
# Initialize repo
git init

# First commit
git add .
git commit -m "Initial commit: MERN project setup"

# Connect to remote GitHub repo (you create the empty repo on github.com first)
git remote add origin https://github.com/yourusername/your-repo.git

# Rename the default branch to "main" (modern convention; old default was "master")
git branch -M main

# Push and set upstream tracking
git push -u origin main
```

After this first push, future pushes are just `git push`.

![[Pasted image 20260430223409.png | 800]]
> 🔁 **Recap:** Git is your single source of truth. Every other DevOps tool — CI, CD, Terraform, Ansible, Kubernetes manifests — pulls from Git. If your code isn't in Git, it doesn't exist as far as DevOps is concerned.

---
## 1.4 — Environment Management
### The problem

Your app needs configuration: a database URL, a JWT secret, an API key for Stripe, a port to listen on.

The wrong instinct (every beginner does this once):
```javascript
// ❌ NEVER DO THIS
const dbUrl = "mongodb+srv://admin:Password123!@cluster.mongodb.net/prod";
const stripeKey = "sk_live_abc123XYZ...";
```

Why is this catastrophic?

1. **Secrets in source code.** The moment this hits GitHub, your DB and Stripe account are compromised.
2. **No way to differ across environments.** Dev should connect to a dev DB. Prod should connect to a prod DB. Hardcoded values can only be one thing at a time.
3. **Recompile to change config.** Changing the DB URL means editing source code → committing → redeploying. Insane.
### The solution: environment variables

Environment variables are key-value pairs the OS provides to running programs. Your code reads them at runtime; the values come from outside the codebase.

```javascript
// ✅ CORRECT
const dbUrl = process.env.MONGO_URI;
const stripeKey = process.env.STRIPE_SECRET_KEY;
```

Now the same code can be wired to a dev DB on your laptop and a prod DB in production — by setting different env vars.
### The three environment tiers

Almost every company has at least these three:

```
   Developer Laptop        Cloud Test Server       Cloud Live Server
   ──────────────         ─────────────────       ──────────────────
       DEV          ──→        STAGING        ──→        PROD
   localhost                  dev DB,                  real DB,
   MongoDB,                   test data                real users,
   no real users              QA tests here            real money
```

|Env|Purpose|Who uses it|
|---|---|---|
|**Development**|Local laptop work. Fast iteration.|Developers|
|**Staging**|Mirrors production exactly. Final sanity check before release.|QA, PMs|
|**Production**|Live system. Real users. Real consequences.|End users|

Some companies add **dev/qa/uat/staging/prod** for more safety. The idea is the same: every environment is a separate "instance" of your app, with its own config.
### `.env` files (the practical mechanism)

Convention: store env vars in a file called `.env`, one per line.

```bash
# server/.env (development)
PORT=5000
MONGO_URI=mongodb://localhost:27017/dev_db
NODE_ENV=development
JWT_SECRET=dev_only_secret_dont_use_in_prod
```

```bash
# server/.env.prod (production — kept on the server only)
PORT=80
MONGO_URI=mongodb+srv://app-user:STRONG_PWD@cluster.mongodb.net/prod
NODE_ENV=production
JWT_SECRET=actually_random_64_char_secret_generated_with_openssl
```

Add `.env*` to `.gitignore`:

```bash
echo ".env" >> .gitignore
echo ".env.*" >> .gitignore
```
### How the code reads `.env`

**Backend (Node.js + Express) — uses the `dotenv` package:**

```javascript
import dotenv from 'dotenv';
dotenv.config();   // reads .env from current directory and populates process.env

const PORT = process.env.PORT || 5000;          // fallback to 5000 if not set
const MONGO_URI = process.env.MONGO_URI;

if (!MONGO_URI) {
    console.error("MONGO_URI is required");
    process.exit(1);                            // fail fast — better than weird errors later
}

app.listen(PORT, () => console.log(`API on ${PORT}`));
```

**Frontend (React + Vite) — Vite reads `VITE_*` prefixed vars at _build time_:**

```bash
# client/.env.development
VITE_API_URL=http://localhost:5000/api

# client/.env.production
VITE_API_URL=https://api.your-prod-domain.com
```

In your React code:

```javascript
const apiUrl = import.meta.env.VITE_API_URL;
fetch(`${apiUrl}/tasks`).then(...);
```

Switch automatically:

```bash
npm run dev      # Vite uses .env.development
npm run build    # Vite uses .env.production
```

> ⚠️ **The Vite / build-time / Docker gotcha (CRITICAL — comes back in Section 3.2):**
> 
> Vite is a **build tool**. During `npm run build`, it reads `VITE_*` variables and _bakes their values directly into the compiled JS bundle_. Once built, the value is hardcoded in `dist/assets/index-abc123.js`. **Changing the env var after building does NOTHING** — the JS file already has the literal string in it.
> 
> Why does this matter for Docker?
> 
> - With backends, you set env vars at _container start_ — `docker run -e MONGO_URI=...` — and the Node process reads them.
> - With Vite frontends, you must pass the value at _image build_ — using a Docker `ARG` — because that's when `npm run build` runs.
> 
> Get this wrong and you'll spend hours wondering why your frontend is calling `localhost:5000` even though you set the env var. We'll fix this properly in the Docker + Nginx sections.
### Other ways to inject env vars (you'll meet all of these)

|Method|Where used|
|---|---|
|`.env` file with `dotenv`|Local dev|
|`export VAR=value` in shell|Quick testing|
|`docker run -e VAR=value`|Docker container at runtime|
|`env_file:` in docker-compose|Compose loads a file into a container|
|`--build-arg` for Vite|Image build time|
|GitHub Actions `secrets`|CI pipelines (encrypted at rest)|
|Kubernetes ConfigMap|Non-sensitive config in K8s|
|Kubernetes Secret|Sensitive config in K8s|
|AWS Secrets Manager / Vault|Enterprise secret stores (rotate keys automatically)|
### Summary
```
   CODE                              CONFIG (env vars)
   ────                              ─────────────────
   const url =          ←────────    MONGO_URI=mongodb://localhost
       process.env                   PORT=5000
       .MONGO_URI                    NODE_ENV=development
                                          ↑
                                          │
                                     Different per env
                                     Never in Git
```

> 🔁 **Recap:** Code is generic and committed. Configuration is environment-specific and _never_ committed. Treat secrets as radioactive — segregated, encrypted, and out of every diff.

---
## 1.5 — Docker
### What problem are we solving?

You've written a Node.js app on your laptop. Node 18, MongoDB running locally, everything works perfectly.

You upload it to a server. The server has Node 16. MongoDB is missing. You install Node 18 — but the version manager interferes with another team's app on the same box. You install MongoDB — but a _different_ version. Some weird OS library is missing. Five hours later, you're debugging a different machine, not your code.

This is the **"works on my machine" problem**, and it's universal.

The pre-Docker solutions were:

**Option 1 — Manual installation, document everything.** Doesn't scale. New servers fall behind. The wiki rots. People skip steps.

**Option 2 — Virtual Machines (VMs).** A VM is a complete simulated computer running its own OS. You can package your app + Node 18 + Mongo + Linux into one VM image and ship it. This works! But:

- VMs are **gigabytes** (you're shipping a whole OS).
- Boot times are **minutes**.
- 5–10 VMs per physical server is the practical density — hardware is heavily underutilized.
- They're heavy on RAM and CPU.

We need the **isolation** of VMs without the **overhead**.
### What Docker actually is

Docker packages an app + everything it depends on (libraries, runtime, configs) into a **container** — a lightweight, isolated, executable unit.

The key architectural insight: containers **share the host OS kernel** but are isolated by Linux features called **namespaces** (each container thinks it has its own filesystem, network, process list) and **cgroups** (each container can be capped on CPU/RAM use). Containers don't carry their own OS — they carry only the bits _above_ the kernel: the app and its libraries.

Therefore:

- **Container size:** MB (not GB)
- **Startup time:** milliseconds (not minutes)
- **Density:** hundreds of containers per machine (not 5–10 VMs)
- **Isolation:** still strong — separate namespaces, capped resources

VM vs Container, side by side:

| VM                 | Container                                   |                                      |
| ------------------ | ------------------------------------------- | ------------------------------------ |
| Carries an OS      | Yes — full guest OS                         | No — shares host kernel              |
| Image size         | GB                                          | MB                                   |
| Boot time          | Minutes                                     | Milliseconds                         |
| Density per server | 5–10                                        | Hundreds                             |
| Isolation          | Strongest (separate kernel)                 | Strong-enough (namespaces + cgroups) |
| Use case           | Run different OSes, strong tenant isolation | Package and run apps consistently    |

VMs and containers aren't enemies. Many real production setups run containers _inside_ VMs — the VM gives strong isolation between customers, the containers give density inside each customer's slice. But for application packaging, containers won.
![[Pasted image 20260501054733.png | 600]]
### Docker architecture

```
   ┌──────────────────────────────────────────────────────┐
   │                  HOST OPERATING SYSTEM               │
   │                  (Linux kernel)                      │
   └──────────────────────────────────────────────────────┘
                            ▲
                            │  uses kernel features
                            │  (namespaces, cgroups)
   ┌──────────────────────────────────────────────────────┐
   │                  DOCKER ENGINE                       │
   │                                                      │
   │   ┌──────────────────────────────────────────────┐   │
   │   │  Docker Daemon (dockerd)                     │   │
   │   │  • Manages images, containers, networks,     │   │
   │   │    volumes                                   │   │
   │   │  • Listens on /var/run/docker.sock           │   │
   │   └──────────────────────────────────────────────┘   │
   │                            ▲                         │
   │                            │ requests over socket    │
   │   ┌──────────────────────────────────────────────┐   │
   │   │  Docker CLI  (the `docker` command)          │   │
   │   └──────────────────────────────────────────────┘   │
   └──────────────────────────────────────────────────────┘
                            │ runs
                            ▼
   ┌──────────────────────────┐    ┌──────────────────────────┐
   │  Container A             │    │  Container B             │
   │  [App + dependencies]    │    │  [App + dependencies]    │
   │  isolated namespace      │    │  isolated namespace      │
   └──────────────────────────┘    └──────────────────────────┘
```

The components, in plain English:

- **Docker Daemon (`dockerd`)** — the _brain_. A long-running background process that does the actual work: building images, starting/stopping containers, managing networks and volumes. Listens on a Unix socket at `/var/run/docker.sock`.
- **Docker CLI** — the `docker` command you type in your terminal. It just sends instructions to dockerd over the socket and prints the response. The CLI itself doesn't run anything.
- **Image** — a read-only, layered snapshot containing your app + dependencies. Like a frozen blueprint or a `.zip` of an OS+app.
- **Container** — a _running instance_ of an image. The image gets a writable layer on top, plus its own isolated namespaces, and becomes a live process.
- **Docker Engine** — daemon + CLI together (the whole installation on your machine).
- **Registry** — a remote store for images. Docker Hub is the default public one. We'll cover registries fully in 2.3.

The relationship in one line:

```
Dockerfile  →  docker build  →  Image  →  docker run  →  Container
(recipe)        (compile the         (frozen template,    (live process,
                 recipe)              read-only)          writable)
```

> 💡 **The `/var/run/docker.sock` socket is the secret behind a clever Jenkins trick we'll see in Section 2.7.** When we install Jenkins as a Docker container and want Jenkins itself to run `docker build`, we mount this socket into the Jenkins container. That gives Jenkins a CLI that talks back to the _host's_ daemon — so Jenkins-in-a-container can build images on the host. Remember this; it returns.
### The Dockerfile — the recipe

A Dockerfile is a plain text file containing instructions to build an image. Each instruction creates a **layer** (we'll see why this matters in a moment).

**Backend Dockerfile (Node.js + Express):**

```dockerfile
# server/Dockerfile

# 1. FROM — start from a pre-built base image.
#    "node:18-alpine" = Alpine Linux + Node.js 18 baked in.
#    Alpine is a minimal Linux distro (~5 MB). Tiny attack surface; small images.
FROM node:18-alpine

# 2. WORKDIR — set the working directory inside the image.
#    All subsequent commands run with /app as their CWD.
#    If /app doesn't exist, Docker creates it.
WORKDIR /app

# 3. COPY only the package files first (key optimization — see "layer caching" below).
#    package*.json matches both package.json and package-lock.json.
COPY package*.json ./

# 4. RUN — execute a shell command at BUILD time.
#    npm install runs once, during build, and bakes node_modules into the image.
RUN npm install

# 5. Now copy the rest of the source code.
#    This is a SEPARATE step from the package.json copy — for caching reasons.
COPY . .

# 6. EXPOSE — document the port the app listens on.
#    This is metadata only; it doesn't actually open a port. That's `-p` at run time.
EXPOSE 5000

# 7. CMD — the command that runs when the container STARTS.
#    Unlike RUN (build time), CMD is the runtime entrypoint.
CMD ["node", "index.js"]
```

**Frontend Dockerfile (React/Vite):**

```dockerfile
# client/Dockerfile
FROM node:18-alpine
WORKDIR /app
COPY package*.json ./
RUN npm install
COPY . .
EXPOSE 5173
CMD ["npm", "run", "dev"]
```

> 🔁 **Recap from 1.4:** when we get to production, the frontend Dockerfile will gain an `ARG VITE_API_URL` because Vite bakes env vars into the build at _build time_, not run time. We'll wire that up in 1.6 / Section 3.

### Dockerfile instruction reference

The full vocabulary, in one place:

|Instruction|Runs at...|Purpose|
|---|---|---|
|`FROM`|build time|Base image to start from|
|`WORKDIR`|build time|Set the directory inside the image|
|`COPY`|build time|Copy files from host into image|
|`ADD`|build time|Like COPY but can also extract tarballs / fetch URLs (rarely needed; prefer COPY)|
|`RUN`|build time|Execute a shell command during build (install packages, compile, etc.)|
|`ENV`|build+runtime|Set an env variable inside the image (visible to RUN steps and to the running process)|
|`ARG`|build time|Define a variable that can be passed via `docker build --build-arg` (gone after build)|
|`EXPOSE`|metadata|Document a port (does NOT open it; that's `-p` at run time)|
|`VOLUME`|metadata|Declare a path that should be a volume|
|`CMD`|runtime|The default command when the container starts|
|`ENTRYPOINT`|runtime|The "main process"; CMD becomes its arguments|
|`USER`|build+runtime|Run subsequent steps and the final container as this user (security)|
|`HEALTHCHECK`|runtime|A command Docker runs to test if the container is healthy|

The trickiest distinction: **RUN vs CMD**.

- `RUN apt install nginx` happens **once during `docker build`** — it bakes nginx into the image.
- `CMD ["nginx", "-g", "daemon off;"]` runs **every time a container starts** from that image.

Get this wrong and you'll find yourself trying to install dependencies at every container start (slow, wrong) or trying to run your server during the build (the build hangs).
### Layer caching — the most important Docker optimization

Every Dockerfile instruction creates a **layer** in the resulting image. Docker caches layers and **reuses** them on rebuild _if the inputs to that layer are unchanged_.

This is why instruction order matters enormously.

**Bad order (cache-hostile):**

```dockerfile
COPY . .                # if ANY file in the project changes, this layer's hash changes
RUN npm install         # so this layer reruns from scratch every rebuild
```

Result: every code edit forces a full `npm install`. On a big project this is 60+ seconds wasted _every build_.

**Good order (the order in our example):**

```dockerfile
COPY package*.json ./   # only changes when dependencies change
RUN npm install         # cached unless package.json/package-lock.json changed
COPY . .                # changes on every code edit, but no expensive command runs after
```

Result: code edits = instant rebuilds (the slow `npm install` layer is reused). Dependency changes = full reinstall (acceptable, because rare).

This is interview gold. The question _"What's the most common Dockerfile optimization?"_ has exactly one expected answer: **"Order instructions so cache-friendly steps come first — copy `package.json`, install dependencies, _then_ copy the rest of the source."**

> ⚙️ **Added Professional Context — multi-stage builds:** Production Dockerfiles often have _two_ stages: a "builder" stage (with all build tools, compiler, dev dependencies) and a "runtime" stage (slim, only the compiled output). The builder produces artifacts; the runtime stage `COPY --from=builder` only what's needed. This shrinks final images dramatically. Example for a Vite frontend:
> 
> ```dockerfile
> # Stage 1 — build
> FROM node:18-alpine AS builder
> WORKDIR /app
> COPY package*.json ./
> RUN npm ci
> COPY . .
> RUN npm run build               # produces /app/dist
> 
> # Stage 2 — serve
> FROM nginx:alpine
> COPY --from=builder /app/dist /usr/share/nginx/html
> EXPOSE 80
> CMD ["nginx", "-g", "daemon off;"]
> ```
> 
> The final image contains only nginx + static files — typically <30 MB instead of 500+ MB.
### `.dockerignore` — the silent disaster preventer

Without a `.dockerignore`, `COPY . .` copies _everything_ in your project directory into the build context — including `node_modules` (huge), `.git/` (huge), `.env` (catastrophic to bake into an image), `dist/`, log files, IDE configs.

Always create a `.dockerignore` next to your Dockerfile:

```
node_modules
.git
.env
.env.*
dist
build
*.log
.DS_Store
.idea
.vscode
coverage
```

Effects:

- Build is faster (less to upload to the daemon).
- Final image is smaller.
- Secrets aren't accidentally embedded.
### Docker CLI — the daily commands

|Command|What it does|
|---|---|
|`docker --version`|Check Docker is installed|
|`docker info`|Detailed info about the daemon|
|`docker run hello-world`|Sanity-check container|
|`docker run -d -p 8080:80 nginx`|Run nginx in background, map host:8080 → container:80|
|`docker run -it ubuntu bash`|Interactive shell in a fresh ubuntu container|
|`docker run --name api -d -p 5000:5000 my-image:1.0`|Named, detached, port-mapped|
|`docker build -t my-image:v1 .`|Build image from Dockerfile in current dir, tag as `my-image:v1`|
|`docker build -t my-image:v1 ./server`|Build with context = ./server|
|`docker ps`|List **running** containers|
|`docker ps -a`|List **all** containers (incl. stopped)|
|`docker stop <id>`|Send SIGTERM, then SIGKILL after timeout|
|`docker start <id>`|Resume a stopped container|
|`docker restart <id>`|Stop + start|
|`docker rm <id>`|Remove a stopped container|
|`docker rm -f <id>`|Force remove (stops first if running)|
|`docker logs <id>`|View container's stdout/stderr|
|`docker logs -f <id>`|Follow logs live (like `tail -f`)|
|`docker logs --tail=100 <id>`|Last 100 lines|
|`docker exec -it <id> bash`|Open a shell _inside_ a running container|
|`docker exec -it <id> sh`|Same, but for Alpine images (which have `sh`, not `bash`)|
|`docker inspect <id>`|Detailed JSON: networks, mounts, env vars, IP|
|`docker images`|List local images|
|`docker rmi <image>`|Remove an image|
|`docker pull <image>`|Download image from registry without running|
|`docker push <image>`|Push tagged image to a registry|
|`docker tag <src> <new>`|Add another tag to an existing image|
|`docker login`|Authenticate to Docker Hub|
|`docker stats`|Live CPU/memory of running containers|
|`docker volume ls`|List all volumes|
|`docker volume create <name>`|Create a named volume|
|`docker volume inspect <name>`|Show volume details and host path|
|`docker network ls`|List all networks|
|`docker network create <name>`|Create a custom user-defined bridge|
|`docker network inspect <name>`|List connected containers and IPs|
|`docker image prune -f`|Remove dangling (unused) images|
|`docker system prune -af --volumes`|Nuke unused everything — frees disk fast (be careful)|
### Decoding `docker run` flags

A typical real command:

```bash
docker run -d \
  -p 5000:5000 \
  --name backend \
  -e NODE_ENV=production \
  -v /var/log/myapp:/app/logs \
  --network mern-net \
  --restart unless-stopped \
  my-image:1.0
```

|Flag|Meaning|
|---|---|
|`-d`|Detached — run in background (no log stream to terminal)|
|`-p HOST:CONTAINER`|Port mapping — host:5000 forwards to container:5000|
|`--name`|Give the container a stable name (default: random like `vibrant_lovelace`)|
|`-e KEY=VAL`|Set env variable inside the container|
|`-v HOST_PATH:CONTAINER_PATH`|Bind-mount a host directory or named volume|
|`--rm`|Auto-delete container when it exits (great for one-shot jobs)|
|`-it`|Interactive + tty — attach a real terminal (for shells)|
|`--network <name>`|Attach to a named network|
|`--restart unless-stopped`|Auto-restart policy: keep restarting unless I explicitly stopped it|
|`--restart always`|Restart always (even if I stopped it manually after reboot)|
### A complete example: build, run, debug, clean up

```bash
# Build the backend image (run from project root)
docker build -t mern-backend:1.0 ./server

# Run it
docker run -d \
  --name backend \
  -p 5000:5000 \
  -e MONGO_URI="mongodb://host.docker.internal:27017/dev" \
  -e NODE_ENV=development \
  mern-backend:1.0

# Check it's running
docker ps

# View logs
docker logs -f backend

# Open a shell to debug something inside
docker exec -it backend sh

# Inside the container — verify env vars, processes, etc.
# / # env | grep MONGO
# / # ps aux
# / # exit

# Stop and clean up
docker stop backend
docker rm backend

# Free disk
docker image prune -f
```

> 💡 `host.docker.internal` is a special DNS name Docker exposes to containers — it resolves to _the host machine_. Useful when a container needs to reach a service running on the host (like a local Mongo not in Docker). On Linux it usually requires `--add-host=host.docker.internal:host-gateway`.
### Common Docker mistakes

1. **Forgetting `-d`** — your terminal hangs because the container's logs stream to it. Easy fix: Ctrl+C, add `-d`, run again.
2. **Port already in use** — `docker run -p 5000:5000` fails because something else is on 5000. Use `lsof -i :5000` (or `netstat -tlnp | grep 5000`) to find the culprit, or pick a different host port (`-p 5001:5000`).
3. **Container exits immediately** — usually the main process crashed. `docker logs <name>` to see why.
4. **`COPY . .` copying too much** — without a `.dockerignore`, `node_modules`, `.git`, `.env` all sneak in. Final images are huge and may contain secrets.
5. **Using `latest` tag in production** — `latest` is mutable. Today's `nginx:latest` ≠ yesterday's. Pin: `nginx:1.27.3-alpine`.
6. **Running as root inside the container** — bad security practice. Add a non-root user:
    
    ```dockerfile
    RUN addgroup -S app && adduser -S -G app appUSER app
    ```
    
7. **Mounting a host directory over an image directory you needed** — `-v $(pwd):/app` overwrites everything in `/app` from the image, including the `node_modules` you installed during build. The fix is an "anonymous volume on top": `-v /app/node_modules` after the bind mount.

> 🔁 **Recap (Docker section):**
> 
> - **Dockerfile** = recipe.
> - **Image** = frozen, layered template.
> - **Container** = running instance with its own writable layer.
> - **Layer caching** is the magic that makes rebuilds fast — order instructions cache-friendly first.
> - **`RUN`** = build time, **`CMD`** = runtime, **`EXPOSE`** = metadata only.
> - **`.dockerignore`** is non-negotiable.
> - **`/var/run/docker.sock`** is the daemon's socket — remember the name; Jenkins uses it.

---
## 1.6 — Docker Compose
### What problem are we solving?

A real app is rarely a single container. The MERN stack alone is 3 services: MongoDB + Node backend + React frontend. Add Redis, an Nginx reverse proxy, a Caddy edge, an Uptime Kuma monitor — now you're at 7+ services.

With raw Docker, starting this stack means:

```bash
docker network create mern-net
docker volume create mongo-data
docker run -d --name mongo --network mern-net -v mongo-data:/data/db mongo:6.0
docker run -d --name backend --network mern-net \
  -e MONGO_URI=mongodb://mongo:27017/db -p 5000:5000 my-backend
docker run -d --name frontend --network mern-net -p 5173:5173 my-frontend
```

Five commands. Easy to forget the order. Easy to typo a port. No record of _why_ you ran them or _what config_ you used. Now imagine 12 services. Now imagine spinning the same stack on dev / staging / prod. Now imagine teaching it to a new developer.

We need a **declarative** way to describe the whole stack — write a file once, run one command.
### What Docker Compose is

Docker Compose is a tool that reads a YAML file (`docker-compose.yml` or `compose.yaml`) describing all services, networks, and volumes, and brings them up with a single command.

> **Analogy:** Docker CLI is a single musician playing alone. Compose is the conductor of an orchestra — one wave of the baton, every section plays in sync.

```bash
docker compose up -d --build
```

That single command replaces the 5+ `docker run` invocations above and gives you a versioned, reviewable, shareable definition of your stack — committable to Git.

> 💡 **Note on syntax:** in modern Docker, the command is `docker compose` (space — Compose v2, written in Go and bundled with Docker). The older `docker-compose` (hyphen — v1, written in Python) is a separate tool now in maintenance. Both still work; new docs use the v2 form. You may also see `compose.yaml` instead of `docker-compose.yml` — same thing.
### docker-compose.yml — the full MERN example

This is the single working YAML you can drop into a project.

```yaml
services:                          # all services we want to run together

  mongo:                           # service name = DNS name on the network!
    image: mongo:6.0               # use a prebuilt image (no Dockerfile needed)
    container_name: mongo          # optional explicit container name
    restart: unless-stopped        # if it crashes, restart it (unless I stopped it manually)
    volumes:
      - mongo-data:/data/db        # named volume mounted at /data/db (Mongo's data dir)
    networks:
      - mern-net

  backend:
    build:                         # build from a Dockerfile (no prebuilt image)
      context: ./server            # path to the directory with Dockerfile
      dockerfile: Dockerfile       # filename (default: "Dockerfile")
    container_name: backend
    restart: unless-stopped
    env_file:                      # load env vars from a file
      - ./server/.env
    ports:
      - "5000:5000"                # host:container — exposes 5000 on the host
    depends_on:                    # start order: wait for mongo before backend
      - mongo
    networks:
      - mern-net

  frontend:
    build:
      context: ./client
      dockerfile: Dockerfile
      args:                        # build-time args — used by Vite (see 1.4 gotcha)
        VITE_API_URL: http://localhost:5000/api
    container_name: frontend
    restart: unless-stopped
    ports:
      - "5173:5173"
    depends_on:
      - backend
    networks:
      - mern-net

volumes:
  mongo-data:                      # named volume — survives `docker compose down`

networks:
  mern-net:
    driver: bridge                 # user-defined bridge network → DNS resolution by service name
```
### How services find each other (DNS magic)

Within `mern-net`, every service can reach the others by **service name**. Compose puts an internal DNS resolver inside every container that translates service names to the current container IP.

- **Backend → Mongo:** `mongodb://mongo:27017/dev_db` — `mongo` resolves to the Mongo container's IP.
- **Backend → Frontend:** rarely needed, but `http://frontend:5173` would work.
- **Browser (host) → Backend:** `http://localhost:5000/api` — but only because we **published** port 5000 with `ports:`. Browsers run on the host; they don't speak Docker DNS.
- **Browser → Frontend:** `http://localhost:5173` — same reason.

The rule, written once for memory:

```
   Container talking to container       →  use the SERVICE NAME
   Browser/host talking to container    →  use LOCALHOST + PUBLISHED PORT
```

We'll go _very_ deep on this in Section 2.2 (Docker Networking) — including the classic `localhost` trap that 100% of beginners hit.
### `depends_on` — and its critical limit

`depends_on` ensures **start order** — Compose starts mongo, then backend, then frontend. But it does **NOT** wait for the service to be _ready_.

What's the difference?

- **Started:** the container's main process is running.
- **Ready:** Mongo has finished initializing and is accepting connections; the backend has finished loading and is listening on 5000.

A Mongo container can be running for 5 seconds before it accepts connections. The backend container starts immediately after — and crashes, because Mongo isn't accepting yet.

Three solutions, in order of quality:

1. **App-level retry logic (best).** Backend retries the DB connection forever, with exponential backoff, on startup. Production apps do this regardless of orchestrator. The `mongoose` library does this by default with `bufferCommands` + retry.
2. **`healthcheck:` + `condition: service_healthy` in Compose.** You define a healthcheck command for mongo, and depends_on waits until the healthcheck passes:
    
    ```yaml
	services:
	  mongo:
	    healthcheck:
	      test: ["CMD", "mongosh", "--eval", "db.adminCommand('ping')"]
	      interval: 5s
	      retries: 5
	
	  backend:
	    depends_on:
	      mongo:
	        condition: service_healthy
    ```
    
3. **`wait-for-it.sh`** scripts that block until a TCP port is open before the app starts.

Option 1 is the only one that survives in production (where Mongo restarts mid-day for failover, and your app shouldn't crash). Use it.
### `env_file` vs `environment`

Two ways to inject env vars:

```yaml
backend:
  # Inline — visible in YAML, fine for non-secrets
  environment:
    - NODE_ENV=production
    - LOG_LEVEL=info

  # From a file — file is gitignored, good for secrets
  env_file:
    - ./server/.env
```

You can use both. `environment` overrides `env_file` if a key appears in both.
### Build args vs runtime env (the Vite gotcha returns)

Recall from 1.4: Vite is a **build tool**. It bakes `VITE_*` env vars into the static bundle _during the build_. Setting them at _runtime_ is too late.

Compose handles this with `args` under `build:`:

```yaml
frontend:
  build:
    context: ./client
    args:
      VITE_API_URL: http://localhost:5000/api    # passed to Dockerfile ARG
```

And the Dockerfile receives it:

```dockerfile
ARG VITE_API_URL                    # declare the arg
ENV VITE_API_URL=$VITE_API_URL      # promote to ENV so `npm run build` sees it
RUN npm run build                   # bakes VITE_API_URL into dist/assets/index-*.js
```

If you put `VITE_API_URL` under `environment:` instead of `args:`, it would only exist when the container _runs_ — but `npm run build` already happened during the image build, with the ARG missing, so the bundle has the wrong value.

> ⚠️ **The most common Vite-in-Docker bug:** "I set VITE_API_URL but the frontend still calls the wrong URL!" → Cause: you set it as a runtime env var, not a build-time ARG. Fix: use `args:` and rebuild the image with `docker compose build --no-cache frontend`.
### Compose CLI — the daily commands

|Command|What it does|
|---|---|
|`docker compose up`|Build images (if needed) + start all services in **foreground**|
|`docker compose up --build`|Force rebuild then start|
|`docker compose up -d`|Detached (background)|
|`docker compose up -d --build`|The combo you'll type a thousand times|
|`docker compose down`|Stop and remove containers + networks (NOT volumes by default)|
|`docker compose down -v`|Also delete volumes — DESTRUCTIVE; drops your DB|
|`docker compose ps`|List services and their status|
|`docker compose logs`|All logs, combined|
|`docker compose logs -f backend`|Follow logs of one service|
|`docker compose stop`|Stop without removing|
|`docker compose start`|Restart stopped services|
|`docker compose build`|Build images, don't start|
|`docker compose build --no-cache`|Force a from-scratch rebuild (ignoring layer cache)|
|`docker compose exec backend sh`|Open a shell in a running service|
|`docker compose pull`|Pull the latest images (for `image:` services)|
|`docker compose config`|Validate the YAML and show the resolved config|
|`docker compose restart backend`|Restart one service|
### Dockerfile vs Compose — FAQ

> **Q: If Compose manages everything, why do Dockerfiles still exist?**

Because they operate at different levels:

- A **Dockerfile** is a recipe to build _one_ image. It operates at the image level. You write one Dockerfile per service that you build yourself (one for backend, one for frontend).
- **Compose** is an orchestrator for _multiple_ containers. It references Dockerfiles for the services it builds and pulls prebuilt images for services it doesn't (Mongo, Redis, Nginx).

Analogy: Dockerfiles are recipes. Compose is the kitchen manager that picks which recipe to cook when, in which order, on which station.
### Common Compose mistakes

1. **Editing source code without rebuilding.** Compose does NOT auto-rebuild on file change. After code changes, run `docker compose up -d --build`. (Bind-mounting source for hot-reload is a different setup, used in dev.)
2. **`latest` tags everywhere.** Same trap as raw Docker — pin versions in `image:` lines.
3. **Forgetting `-v` to remove volumes during reset.** `docker compose down` keeps your volume; you'll wonder why your DB still has yesterday's data after a "fresh" start. Use `docker compose down -v` for a true reset.
4. **Putting secrets directly in YAML.** YAML lives in Git. Use `env_file` (gitignored) or Docker secrets.
5. **Mixing v1 and v2 syntax.** Old tutorials say `version: '3.8'` at the top — modern Compose ignores that line. Don't worry about it.
6. **Same `container_name` on two services.** Compose refuses; the names must be unique.
7. **Mistyping a service name in `depends_on`.** Compose errors out clearly, but if you're new the message can be cryptic.

> 🔁 **Recap (Compose):** declarative multi-container orchestration. Service names become DNS names within the network. `depends_on` is start order, not readiness. Build args ≠ runtime env. Always gitignore `.env` files Compose loads.

---
## 1.7 — Continuous Integration (CI)
### What problem are we solving?

Three developers — Alice, Bob, Carol — work on the same project.

- **Day 1:** all three pull `main`. Each starts a feature branch.
- **Day 5:** Alice finishes, merges to main.
- **Day 7:** Bob finishes... but his code edited the same file Alice did. Merge conflicts. Four hours resolving them.
- **Day 10:** Carol finishes... her changes also conflict with Alice's _and_ Bob's. Eight hours resolving them.
- **Day 12:** Carol's code finally merges. Everything compiles. But there's a subtle bug — only triggered when Alice's and Carol's code interact. Found in production three days later by users.

This is **integration hell** — the longer code stays unmerged, the harder it is to integrate, and the more hidden bugs lurk in the unintegrated combination.

There's a second problem on top: even when code merges cleanly, was it tested? Did Bob actually run the linter? Did Carol forget the unit tests? In a manual world, the answer is: _sometimes_. And "sometimes" means production breakage.
### What CI is

**Continuous Integration** is the practice of:

1. Every developer integrates their work into the shared `main` (or `develop`) branch **frequently** (multiple times a day — not weeks of work in a private branch).
2. Every push triggers an **automated process** that builds the code, runs tests, checks formatting, scans for vulnerabilities — and reports pass/fail.
3. If the automated checks fail, the merge is **blocked** until the author fixes the problem.

The tools that run that "automated process" are GitHub Actions, Jenkins, GitLab CI, CircleCI, Travis CI, Buildkite, etc. The _practice_ — integrate often, automate verification — is what CI _means_. Tools just enable it.

> 💡 The "Continuous" in CI doesn't mean "running constantly." It means _frequent and automatic_ — every push triggers it; you never wait for "release day" to integrate.
### The CI workflow logic

```
   Developer pushes to feature branch
         │
         ▼
   GitHub detects the push (workflow trigger)
         │
         ▼
   CI Runner spins up (a fresh Ubuntu VM in the cloud)
         │
         ▼
   Steps execute, in order:
     1. Checkout the code
     2. Install Node.js + dependencies (npm ci)
     3. Run linter      (eslint, with --max-warnings=0)
     4. Run tests       (jest)
     5. Build Docker image (verify the Dockerfile actually works)
     6. (Optional)  Run security scan, push image to registry, etc.
         │
         ▼
		┌──────┴──────┐
		│             │
		▼ PASS        ▼ FAIL
		✅            ❌
		PR mergeable     PR blocked
					Developer fixes,
					pushes again,
					workflow restarts
					
   Once PR is merged to main:
         ▼
   (CD job triggers — covered in 1.10)
```
### The three-rule CI safety net

For CI to actually prevent broken code from reaching `main`, three things must be true. Miss any one and the system becomes advisory rather than mandatory.

**Rule 1 — Branching strategy.** Developers never push directly to `main`. All work happens on feature branches. (We covered this in 1.3 — without it, there's no PR, so no CI gate.)

**Rule 2 — Pull Requests (PRs).** Code can only enter `main` via a Pull Request — a GitHub feature where you propose a merge from your feature branch into `main`, others review, and CI checks run automatically.

**Rule 3 — Branch Protection.** A GitHub setting: _"require status checks to pass before merging."_ Once enabled, the green Merge button on the PR page is **disabled** until the CI workflow reports success. This is what makes CI mandatory rather than advisory — without branch protection, developers can ignore the red ❌ and merge anyway.

We will configure all three in Section 1.8.
### What CI specifically prevents

|Class of bug|How CI catches it|
|---|---|
|Syntax error / typo|The build/lint step fails|
|Forgot to run formatter|`prettier --check` step|
|Broke an existing feature|Unit tests|
|Dockerfile broken|"build the image" step|
|Vulnerable dependency|`npm audit` / `trivy` scan|
|Forgot to add an env var to docs|Build fails because config can't be read|
|Merge conflicts|GitHub blocks merge until resolved|
|Two PRs that pass alone but conflict|"Require branches up-to-date before merge"|
### CI vs CD — distinction matters

CI and CD are often combined as "CI/CD" but mean different things:

- **CI — Continuous Integration:** automate building + testing on every push. Confirms code is _correct_.
- **CD — Continuous Delivery:** every successful CI build is _ready_ to deploy with a click. Confirms code is _deployable_. The deploy itself still requires a human button-press.
- **CD — Continuous Deployment:** every successful CI build is _automatically_ deployed to production. No human in the loop.

So "CI/CD" can mean Continuous Integration + Delivery OR Continuous Integration + Deployment. Same acronym, two flavors. Most companies use:

- **Continuous Deployment** for staging (auto-deploy every push to a `staging` branch).
- **Continuous Delivery** for production (every commit produces a deployable artifact, but a human clicks "release").

We'll implement CI in Section 1.8 (GitHub Actions) and CD in Section 1.10 (auto-deploy to EC2).
### Why CI matters BEYOND just catching bugs

CI does more than save you from broken builds:

1. **Codifies team standards.** "We use ESLint with these rules" stops being a wiki page nobody reads — it's now enforced by a robot every time anyone pushes.
2. **Onboarding.** A new dev opens a PR; CI tells them what's wrong without anyone having to scold them. They learn the team's standards by interaction.
3. **Confidence to refactor.** Want to upgrade Node 18 → 20? Make the change, push the PR, CI tells you instantly if anything breaks.
4. **Audit trail.** Every commit has a recorded build artifact and test result. Compliance frameworks (SOC2, ISO 27001, HIPAA) all expect this.
5. **Faster releases.** With CI, you can confidently merge 20 PRs a day instead of 2 a week. Velocity scales with safety.
6. **Pre-production environments stay clean.** Staging and dev only get code that already passed CI — they aren't littered with half-broken commits.

> 🔁 **Recap (CI):** CI is a _practice_ (integrate often, automate verification). The _tool_ (GitHub Actions, Jenkins) is just the engine that enacts it. Branch protection is what turns CI from advisory to mandatory. CI is upstream — CD takes over after CI passes.

---
## 1.8 — GitHub Actions, ESLint and Prettier
### What problem are we solving?

Section 1.7 established _why_ we want CI. Now we need a concrete tool to actually execute the pipeline whenever someone pushes code. There are many CI engines (Jenkins, GitLab CI, CircleCI), but if your code is already on GitHub, **GitHub Actions** is the path of least resistance — it's free for public repos, generous for private repos, and lives directly inside the platform you already use.
### What GitHub Actions is

GitHub Actions is GitHub's built-in CI/CD engine. You commit a YAML file in a special folder, and GitHub watches your repo for events (a push, a PR opened, a release tagged). On a matching event, GitHub spins up a temporary Linux VM (a **runner**), executes the steps in your YAML, and reports pass/fail back to your PR.

Key vocabulary:

|Term|Meaning|
|---|---|
|**Workflow**|A YAML file that defines a CI pipeline. Lives in `.github/workflows/`|
|**Event**|Something that triggers a workflow (push, PR, schedule, manual)|
|**Job**|A group of steps that run on the same runner|
|**Step**|A single command or pre-built action invocation within a job|
|**Runner**|The temporary VM that executes the steps (Ubuntu, Windows, macOS available)|
|**Action**|A reusable, pre-built unit of work (`actions/checkout@v4`, etc.)|
|**Secret**|An encrypted variable injected into runners (DB URLs, API keys, SSH keys)|

> 💡 **Why YAML in `.github/workflows/`?** GitHub specifically scans this folder. Files there become workflows automatically — no separate registration step. Each `.yml` file is a separate workflow.
### A first workflow — lint + test on every PR

Create `.github/workflows/ci.yml`:

```yaml
name: CI                          # human-readable name shown on the Actions tab

on:                               # WHEN does this workflow run?
  pull_request:                   # any PR being opened or updated...
    branches: [ main ]            # ...that targets the `main` branch
  push:                           # also on direct pushes to main
    branches: [ main ]

jobs:                             # one workflow can have many jobs (parallel)
  lint-and-test:                  # job name
    runs-on: ubuntu-latest        # which runner to use (free Ubuntu VM)

    steps:                        # ordered list of actions
      - name: Checkout code
        uses: actions/checkout@v4 # pre-built action: clones the repo into the runner

      - name: Setup Node.js
        uses: actions/setup-node@v4
        with:
          node-version: 18
          cache: 'npm'            # cache ~/.npm between runs (huge speed-up)

      - name: Install dependencies
        run: npm ci               # `npm ci` = clean, lockfile-strict install (faster than npm install in CI)
        working-directory: ./server

      - name: Run linter
        run: npm run lint         # uses script defined in package.json
        working-directory: ./server

      - name: Run tests
        run: npm test
        working-directory: ./server
```

What happens after this is committed:

1. You push or open a PR.
2. GitHub spins up a fresh Ubuntu VM.
3. The 5 steps run sequentially.
4. Each step's pass/fail is reported on the PR page (those green ✓ and red ✗ marks).
5. If any step fails, the workflow fails. Branch protection (next subsection) blocks the merge.
### `uses:` vs `run:`

- **`run:`** — a shell command. Whatever you'd type in a terminal.
- **`uses:`** — invoke a pre-built reusable Action from the [GitHub Marketplace](https://github.com/marketplace?type=actions). The most-used ones:

|Action|What it does|
|---|---|
|`actions/checkout@v4`|Clone the repo into the runner|
|`actions/setup-node@v4`|Install Node.js|
|`actions/setup-python@v5`|Install Python|
|`actions/cache@v4`|Cache directories between runs|
|`docker/setup-buildx-action@v3`|Set up Docker Buildx for advanced builds|
|`docker/login-action@v3`|Log in to Docker Hub / GHCR|
|`docker/build-push-action@v5`|Build and push images|
|`appleboy/ssh-action@v1`|SSH into a server and run commands (used in CD)|
### `npm ci` vs `npm install` in CI

You'll always see `npm ci` (not `npm install`) in CI. Why?

|Behavior|`npm install`|`npm ci`|
|---|---|---|
|Reads `package.json`|Yes|Yes|
|Reads `package-lock.json`|Yes (loosely)|Yes (strictly)|
|Updates lockfile|May update|Never|
|If lockfile differs|Reconciles|Fails immediately|
|Speed|Slower|Significantly faster|
|Fresh `node_modules`|No (incremental)|Yes (deletes first, then installs)|

CI wants reproducibility. `npm ci` says: _"give me exactly what the lockfile says, or fail."_ That's exactly the contract you want.
### ESLint — the linter

A **linter** is a tool that statically analyzes your code (without running it) for problems: syntax errors, unused variables, suspicious patterns, style violations. ESLint is the JavaScript/TypeScript standard.

Setup in a Node project:

```bash
cd server
npm install --save-dev eslint
npx eslint --init       # interactive setup; produces .eslintrc.json
```

Add a script to `package.json`:

```json
{
  "scripts": {
    "lint": "eslint . --ext .js,.jsx,.ts,.tsx --max-warnings=0"
  }
}
```

The `--max-warnings=0` is critical: it makes the lint script _fail_ if there are any warnings, not just errors. Without it, warnings accumulate forever because nobody is forced to fix them.

A typical `.eslintrc.json`:

```json
{
  "env": { "node": true, "es2022": true, "jest": true },
  "extends": ["eslint:recommended"],
  "parserOptions": { "ecmaVersion": "latest", "sourceType": "module" },
  "rules": {
    "no-unused-vars": "error",
    "no-console": "warn",
    "eqeqeq": ["error", "always"]
  }
}
```

**What ESLint catches that the JS engine doesn't:**

- Variables you declared but never used (potential bugs / dead code).
- `==` vs `===` (loose equality has subtle bugs in JavaScript).
- Forgotten `await` on async calls.
- Reassigning `const`.
- Returning a value from a function that's supposed to be void.
- Many more — there are hundreds of available rules.
### Prettier — the formatter

ESLint catches _bugs_ and _style issues_; Prettier _fixes formatting_ automatically — indentation, line length, quotes, trailing commas. Together they cover both the substance and the look of code.

```bash
npm install --save-dev prettier
echo '{}' > .prettierrc          # default config
```

Add to `package.json`:

```json
{
  "scripts": {
    "format": "prettier --write .",          // rewrites files
    "format:check": "prettier --check ."     // CI mode: fails if anything is mis-formatted
  }
}
```

Add a step to your CI workflow:

```yaml
      - name: Check formatting
        run: npm run format:check
        working-directory: ./server
```

Now if anyone forgets to run Prettier locally, CI fails their PR with a list of files needing reformatting. Result: **the entire codebase has consistent formatting forever**, with zero arguments in code review.

> 💡 **The cultural win:** before Prettier, code reviews were 30% bickering about indentation. After Prettier, those debates evaporated industry-wide. Adopting it is one of the highest-leverage moves a team can make.
### ESLint vs Prettier — they're not competitors

|Tool|Catches|Fixes automatically?|
|---|---|---|
|ESLint|Bugs, unused vars, anti-patterns, style errors|Some (`--fix` flag)|
|Prettier|Pure formatting (whitespace, quotes, line length)|Yes (its whole job)|

They're complementary. Run both. The ESLint plugin `eslint-config-prettier` disables ESLint's formatting rules so they don't conflict with Prettier — install it and you're done.
### Branch Protection — the missing 1% that makes CI mandatory

Without branch protection, the CI workflow is **advisory** — devs can ignore the red ❌ and merge anyway. Turn it on:

GitHub → your repo → **Settings → Branches → Add branch protection rule**:

- **Branch name pattern:** `main`
- ✅ Require a pull request before merging
- ✅ Require approvals (at least 1)
- ✅ Require status checks to pass before merging
    - In the search box, type `lint-and-test` (or whatever your job name is) — pick it.
- ✅ Require branches to be up to date before merging (catches "passes alone but breaks combined")
- ✅ Do not allow bypassing the above settings (even admins)

Now the merge button is **disabled** until your CI workflow reports green. CI just became the law.
### Secrets — the right way to inject sensitive values

Some workflow steps need credentials: an API key, an SSH private key for deployment, a Docker Hub password. **Never hardcode these in YAML** — YAML is in Git, and Git is forever.

GitHub provides **encrypted secrets**. Settings → Secrets and variables → Actions → New repository secret:

```
EC2_HOST          → 3.91.123.45
EC2_USER          → ec2-user
EC2_SSH_KEY       → -----BEGIN OPENSSH PRIVATE KEY----- ...
DOCKERHUB_USER    → ankitsangwan
DOCKERHUB_TOKEN   → dckr_pat_xxxxxxxxxx
```

In the workflow, reference them as `${{ secrets.SECRET_NAME }}`:

```yaml
      - name: Login to Docker Hub
        uses: docker/login-action@v3
        with:
          username: ${{ secrets.DOCKERHUB_USER }}
          password: ${{ secrets.DOCKERHUB_TOKEN }}
```

Properties of GitHub Secrets:

- Encrypted at rest.
- Not printed to logs (GitHub auto-masks them).
- Not accessible in workflows triggered by PRs from forks (huge security feature — prevents random PRs from stealing your secrets).
- Scoped per-repo (or per-org / per-environment).

![[Pasted image 20260501060837.png | 600]]

> 🔁 **Recap (1.8):** GitHub Actions = YAML in `.github/workflows/`, runs on events, executes steps on a temporary runner. ESLint catches code issues; Prettier formats. `npm ci` for reproducibility. Branch protection makes the CI gate mandatory. Secrets keep credentials out of YAML.

---
## 1.9 — AWS EC2 Deployment
### What problem are we solving?

Your app works on your laptop and passes CI. But a laptop isn't a server — it sleeps, the IP changes, it isn't reachable from the internet. To put the app in front of real users, you need a server in the cloud that runs 24/7 with a public IP. The most common cloud server is an **AWS EC2 instance**.

This section is the manual, step-by-step deployment. Section 1.10 then shows how to _automate_ the same steps via GitHub Actions.
### What EC2 is

**EC2 = Elastic Compute Cloud** — Amazon's service for renting virtual machines. You pick:

- **AMI** (Amazon Machine Image) — the OS to start from (Amazon Linux, Ubuntu, RHEL, etc.).
- **Instance type** — the size (t2.micro = 1 vCPU + 1 GB RAM; m5.large = 2 vCPU + 8 GB RAM; etc.).
- **Storage** — typically EBS (Elastic Block Store) for persistent disk.
- **Security Group** — firewall rules controlling which ports are reachable.
- **Key Pair** — SSH key for login.

Within a minute, AWS provisions a real Linux machine and gives you a public IP. You SSH in and treat it like any Linux server.

> 💡 **"Elastic" means you can resize.** Today's t2.micro can become tomorrow's m5.large with a stop / change / start. You're not locked to the size you picked initially.
### High-level deployment flow

```
   ┌──────────────────────────────────────────────────────────┐
   │  1. AWS Console — launch EC2 instance                    │
   │     (pick AMI, size, key pair, security group)           │
   └──────────────────────────────────────────────────────────┘
                              │
                              ▼
   ┌──────────────────────────────────────────────────────────┐
   │  2. SSH in with your private key                         │
   └──────────────────────────────────────────────────────────┘
                              │
                              ▼
   ┌──────────────────────────────────────────────────────────┐
   │  3. Install Docker (and git)                             │
   └──────────────────────────────────────────────────────────┘
                              │
                              ▼
   ┌──────────────────────────────────────────────────────────┐
   │  4. Clone your repo from GitHub                          │
   └──────────────────────────────────────────────────────────┘
                              │
                              ▼
   ┌──────────────────────────────────────────────────────────┐
   │  5. Create production .env files                         │
   └──────────────────────────────────────────────────────────┘
                              │
                              ▼
   ┌──────────────────────────────────────────────────────────┐
   │  6. docker compose up -d --build                         │
   └──────────────────────────────────────────────────────────┘
                              │
                              ▼
   ┌──────────────────────────────────────────────────────────┐
   │  7. Open the right ports in the Security Group           │
   │     so users can reach the app                           │
   └──────────────────────────────────────────────────────────┘
                              │
                              ▼
                    Visit http://EC2_IP:5173
```
### Step 1 — Launch the instance

In the AWS Console (sign-in at console.aws.amazon.com):

1. **EC2 → Instances → Launch instance.**
2. **Name:** `mern-prod-server`
3. **AMI:** Amazon Linux 2023 (or Ubuntu 22.04 — both fine; commands below assume Amazon Linux).
4. **Instance type:** `t2.micro` (free tier) or `t3.small` for real work.
5. **Key pair:** Create new → name it `mern-key` → download the `.pem` file. **You can't re-download this — store it carefully.** Anyone with this file can SSH in as root.
6. **Network settings:**
    - Auto-assign public IP: Enable.
    - Security Group: create new, named `mern-sg`. Initially allow only SSH (port 22) from "My IP".
7. **Storage:** 8 GB default is enough for the demo.
8. Click **Launch instance.**

Within ~30 seconds the instance is _running_ with a public IPv4 address (e.g., `3.91.123.45`).
### Step 2 — SSH in

Open your terminal (WSL on Windows, native on macOS/Linux):

```bash
# AWS REQUIRES strict permissions on the key file (you learned this in 1.1.7)
chmod 400 mern-key.pem

# Connect
ssh -i mern-key.pem ec2-user@3.91.123.45
#       │            │       │
#       │            │       └── public IPv4 from the EC2 console
#       │            └────────── default user for Amazon Linux
#       │                        (Ubuntu uses "ubuntu", RHEL uses "ec2-user")
#       └────────────────────── private key file

# First time: "The authenticity of host can't be established. Continue?"
# Type: yes
```

You're now inside the EC2 instance. Verify with `whoami` (→ `ec2-user`) and `pwd` (→ `/home/ec2-user`).
### Step 3 — Install Docker and Git

```bash
# Update package list and installed packages
sudo yum update -y                  # Amazon Linux uses yum/dnf (not apt)

# Install Docker
sudo yum install -y docker

# Start the Docker daemon
sudo systemctl start docker
sudo systemctl enable docker        # auto-start on reboot

# Let ec2-user run docker without sudo (you set this up in 1.1.4 too)
sudo usermod -aG docker ec2-user

# IMPORTANT: log out and back in for the group change to take effect
exit
ssh -i mern-key.pem ec2-user@3.91.123.45

# Verify
docker --version
docker run hello-world

# Install Docker Compose (if not bundled in your AMI)
sudo curl -L "https://github.com/docker/compose/releases/latest/download/docker-compose-$(uname -s)-$(uname -m)" \
  -o /usr/local/bin/docker-compose
sudo chmod +x /usr/local/bin/docker-compose
docker-compose --version
# Modern installations include "docker compose" (v2) automatically — try that first.

# Install Git
sudo yum install -y git
git --version
```
### Step 4 — Clone the repo

```bash
# Public repo — straightforward
git clone https://github.com/yourusername/mern-app.git
cd mern-app

# Private repo — needs a deploy key or HTTPS token
# (For demo, use a public repo. We'll handle private repos in CD later.)
```
### Step 5 — Create production `.env` files

Recall from 1.4: `.env` is gitignored, so it's NOT in the cloned repo. You create it directly on the server:

```bash
# Backend env
cat > server/.env <<EOF
PORT=5000
MONGO_URI=mongodb+srv://prod-user:STRONG_PASSWORD@cluster.mongodb.net/prod_db
NODE_ENV=production
JWT_SECRET=$(openssl rand -hex 32)
EOF

# Frontend env (Vite — for build)
cat > client/.env.production <<EOF
VITE_API_URL=http://3.91.123.45:5000/api
EOF
```
### Step 6 — Build and run with Compose

```bash
# Bring up the whole stack (build + start, in background)
docker compose up -d --build

# Watch logs to confirm it's healthy
docker compose logs -f

# Exit log view: Ctrl+C
# Confirm containers are running
docker compose ps
```
### Step 7 — Open ports in the Security Group

The Security Group is AWS's firewall _in front of_ the instance. By default, only SSH is open. Your app's ports (5000 backend, 5173 frontend) are still blocked from the internet — that's why visiting `http://3.91.123.45:5173` would hang.

In the AWS Console:

1. **EC2 → Security Groups → `mern-sg` → Edit inbound rules → Add rule.**
2. Two rules:

|Type|Protocol|Port range|Source|Description|
|---|---|---|---|---|
|Custom TCP|TCP|5000|0.0.0.0/0|Backend API|
|Custom TCP|TCP|5173|0.0.0.0/0|Frontend|

3. **Save rules.**
### What `0.0.0.0/0` means (CIDR notation)

`0.0.0.0/0` = "every possible IPv4 address." It opens the port to the whole internet, which is what a public web app needs.

CIDR notation in general:

- `192.168.1.10/32` — exactly one IP (the `/32` says "match all 32 bits").
- `10.0.0.0/24` — 256 IPs (`/24` says "match the first 24 bits", so the last 8 bits vary: `10.0.0.0` through `10.0.0.255`).
- `10.0.0.0/16` — 65,536 IPs.
- `0.0.0.0/0` — all IPs.

For SSH, restrict to your office IP:

|Type|Protocol|Port|Source|Description|
|---|---|---|---|---|
|SSH|TCP|22|73.45.123.45/32|My IP|

For HTTP / HTTPS / app ports that need to be public, use `0.0.0.0/0`.

> ⚠️ **The most common AWS-deployment failure:** "I deployed everything but the URL doesn't load!" Cause: SG rule missing or scoped wrong. Always check SG inbound rules first when an EC2 service is unreachable. Second-most-common cause: the _app_ didn't start (check `docker compose logs`).
### Visit the app

```
   http://3.91.123.45:5173    →  React frontend
   http://3.91.123.45:5000    →  Backend API
```
### Three deployment scenarios — what's missing here

What we built above is the simplest scenario. As your app grows, you upgrade through these stages:
#### a) EC2 alone (what we just did)

```
   Browser  ──→  EC2 instance (port 5173 frontend, port 5000 backend)
```

Pros: simple. Cons: ports 5000/5173 in the URL is ugly; HTTP not HTTPS; one server = single point of failure.
#### b) EC2 + Nginx (reverse proxy on the same machine)

```
   Browser  ──→  Nginx :80 ──→ frontend :5173 (for /)
                              ──→ backend  :5000 (for /api)
```

We cover Nginx in detail in Section 3.2. The win: clean URLs (no port numbers visible), single port (80) exposed to the internet, can add HTTPS, can serve static files faster.
#### c) EC2 + Load Balancer

```
   Browser  ──→ AWS ALB :443 ──→ EC2 #1 (running Nginx + app)
                              ──→ EC2 #2 (running Nginx + app)
                              ──→ EC2 #3 (running Nginx + app)
```

The Application Load Balancer (ALB) is an AWS-managed proxy that distributes traffic across multiple EC2s. Survives the loss of any single EC2; provides HTTPS via AWS Certificate Manager (free certs); enables blue/green deploys and rolling updates.
#### d) Kubernetes + Load Balancer

```
   Browser ──→ Cloud LB ──→ Ingress Controller (in K8s) ──→ Service ──→ Pod replicas
```

Highest scale and complexity. Auto-scaling, self-healing, declarative ops. Covered in Section 3.3 onward.

We climb this ladder as the app grows.

> 🔁 **Recap (EC2):** EC2 = a rented Linux VM with a public IP. SSH in with the `.pem` key. Install Docker + Git. Open SG ports for the services you publish. The one trap nobody warns you about: SG rules. Always check them first when "it doesn't load."

---
## 1.10 — Continuous Deployment (CD)
### What problem are we solving?

We can deploy manually (Section 1.9). But "manually" means SSHing into the server, running `git pull`, running `docker compose up -d --build`, watching logs — every time anyone merges a PR. At the rate of "multiple times a day," this becomes the team's full-time job. Worse, it's error-prone — human forgets a step → outage.

We want: when code is merged to `main`, the production server **automatically** updates itself. No human in the loop.

That's **CD — Continuous Deployment**.
### CI vs CD recap

```
   git push  ─────►  CI (lint + test + build)  ─────►  CD (deploy)
                     ▲                                  ▲
                     │                                  │
              "is the code correct?"        "make production look like main"
                  GitHub Actions                  GitHub Actions (different job)
```

The CI workflow guarded `main` from broken code. The CD workflow takes whatever's now in `main` and ships it to the server.
### The CD flow we'll build

```
   PR merged to main
         │
         ▼
   GitHub Actions runs the "deploy" workflow
         │
         ▼
   Workflow uses an SSH private key (stored in Secrets)
   to log into the EC2 instance
         │
         ▼
   On the EC2:
     cd /home/ec2-user/mern-app
     git pull origin main
     docker compose up -d --build
         │
         ▼
   New version is live in ~1 minute
```
### The deploy workflow YAML

`.github/workflows/deploy.yml`:

```yaml
name: Deploy to EC2

on:
  push:
    branches: [ main ]            # ONLY when code lands on main (after PR merge)

jobs:
  deploy:
    runs-on: ubuntu-latest

    steps:
      - name: Checkout
        uses: actions/checkout@v4

      - name: Deploy to EC2 over SSH
        uses: appleboy/ssh-action@v1.0.3
        with:
          host: ${{ secrets.EC2_HOST }}        # public IP, e.g. 3.91.123.45
          username: ${{ secrets.EC2_USER }}    # "ec2-user"
          key: ${{ secrets.EC2_SSH_KEY }}      # the private .pem contents
          port: 22
          script: |
            # The runner has SSH'd into the EC2; this script runs on the EC2
            cd /home/ec2-user/mern-app
            git pull origin main
            docker compose up -d --build
            docker image prune -f          # tidy up old image layers to save disk
```

Six lines of bash inside an SSH session, wrapped in a workflow. That's all CD is.
### Setting up the secrets

In GitHub: Settings → Secrets and variables → Actions → New repository secret. Add three:

|Secret name|Value|
|---|---|
|`EC2_HOST`|`3.91.123.45` (your EC2 public IP)|
|`EC2_USER`|`ec2-user` (or `ubuntu` for an Ubuntu AMI)|
|`EC2_SSH_KEY`|The **entire contents** of your `.pem` file, including the header/footer|

For `EC2_SSH_KEY`, paste literally everything:

```
-----BEGIN RSA PRIVATE KEY-----
MIIEowIBAAKCAQEA1234abcd...
...many lines...
-----END RSA PRIVATE KEY-----
```

> ⚠️ **Common pitfall:** people paste only the middle of the key, missing the BEGIN/END lines. SSH refuses; the action fails with `ssh: handshake failed`. Always paste the _entire_ file content.
### What the SSH key on EC2 needs

The key you give to GitHub must already be authorized to log into the EC2 instance. There are two ways:

1. **Reuse the AWS-generated key pair** (`mern-key.pem`). It's already authorized for `ec2-user` via `~/.ssh/authorized_keys` on the instance. Paste its contents into `EC2_SSH_KEY`.
2. **Create a deploy-only key.** Generate a new keypair on your laptop (`ssh-keygen -t ed25519 -C "github-actions-deploy"`), `ssh-copy-id` the public key to the EC2 instance (so it's appended to `authorized_keys`), and paste the _private_ key into the GitHub Secret.

Option 2 is safer (you can revoke the deploy key without rotating the master key), but Option 1 is fine for learning.
### Why this works (security walkthrough)

When the workflow runs:

1. The runner (a fresh Ubuntu VM in Microsoft Azure) is allocated.
2. GitHub injects `secrets.EC2_SSH_KEY` into the runner's environment as a temporary file.
3. The `appleboy/ssh-action` SSHs into your EC2 using that key.
4. The script runs on the EC2 — `git pull`, `docker compose up`.
5. SSH session closes.
6. The runner is destroyed (and the key file with it).
7. Logs are returned to GitHub.

The private key never appears in any `.yml` file in your repo, never appears in logs, and never persists.
### Branching strategy for CI/CD together

A complete setup has both files:

```
.github/workflows/
   ci.yml         ← runs on PRs and pushes to main: lint + test
   deploy.yml     ← runs on push to main only: deploy to EC2
```

Lifecycle:

1. Dev opens PR → `ci.yml` runs → green ✓ required by branch protection.
2. PR merges to `main` → both `ci.yml` AND `deploy.yml` run on the merge commit.
3. `deploy.yml` only proceeds if `ci.yml` already passed (you can chain workflows with `workflow_run`, or — more simply — keep the lint/test inside the deploy workflow as initial steps).

A more defensive deploy:

```yaml
jobs:
  test:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-node@v4
        with: { node-version: 18, cache: 'npm' }
      - run: npm ci
        working-directory: ./server
      - run: npm test
        working-directory: ./server

  deploy:
    needs: test                   # only run if `test` job succeeded
    runs-on: ubuntu-latest
    steps:
      - uses: appleboy/ssh-action@v1.0.3
        with:
          host: ${{ secrets.EC2_HOST }}
          username: ${{ secrets.EC2_USER }}
          key: ${{ secrets.EC2_SSH_KEY }}
          script: |
            cd /home/ec2-user/mern-app
            git pull origin main
            docker compose up -d --build
            docker image prune -f
```

The `needs: test` line is the safety harness — `deploy` is skipped if `test` fails, so you never auto-deploy a broken commit even if it somehow landed on main.
### Where this CD design breaks (and the path forward)

The `git pull + docker compose up -d --build` strategy is great for a single server. It breaks at scale:

|Scenario|Problem|
|---|---|
|5 servers behind a load balancer|Each one needs to be pulled+rebuilt. Race conditions. Inconsistent state.|
|Build fails on the server|Server has half-old, half-new code, possibly down|
|Rollback needed|No artifact. You'd have to `git revert` and re-pull. Slow.|
|Build is heavy (10 min)|Every deploy locks up the production server's CPU|
|Multiple devs deploy simultaneously|Two `docker compose build` invocations stomping on each other|

Industry-grade CD addresses these by **building images in CI** (in the runner, not on the server), pushing them to a registry, and **pulling pre-built images** on the production servers. The production server's job becomes "stop old container, pull new image, start new container" — fast, atomic, rollback-able.

We refine toward that pattern in Section 2 (registries) and Section 3 (Kubernetes / ArgoCD).

> 🔁 **Recap (CD):** CD = automated deploy on every push to main. The simplest implementation is GitHub Actions + SSH + `git pull + docker compose up`. Secrets keep credentials out of YAML. `needs:` chains jobs so deploy can't happen if tests failed. The pattern scales up by moving builds out of the server and into the registry-driven flow.

---
## 1.11 — Kubernetes (introduction)

This is a _first introduction_. We come back deep in Section 3.3 once we've laid the Docker-networking and reverse-proxy foundations.
### What problem are we solving?

Docker Compose is fantastic for one server. But what happens when:

- **Traffic 10x's.** You need 5 servers, not 1. Now Compose isn't enough — Compose only knows about the one machine it's running on.
- **A container crashes at 3 AM.** With Compose's `restart: unless-stopped`, it'll restart. But if the _whole machine_ dies, nothing brings it back.
- **You deploy a new version.** Compose's `up -d --build` causes a brief downtime as containers stop and restart. For a 24/7 service, that's unacceptable.
- **CPU/memory needs vary.** Maybe the app needs 3 backend replicas during business hours and 1 at night. Compose can't auto-scale.

You need an **orchestrator** — a system that manages a _fleet_ of machines and a _fleet_ of containers across them, automatically. That's Kubernetes (often abbreviated **K8s** — k, 8 letters, s).
### What Kubernetes is, in one sentence

> Kubernetes is a system for running containerized applications across a cluster of machines, with automatic scheduling, self-healing, scaling, and rolling updates.
### Kubernetes vs Docker Compose

|Concern|Docker Compose|Kubernetes|
|---|---|---|
|Number of machines|1|Many (a "cluster")|
|If a container crashes|Restart on the same machine|Restart anywhere in the cluster|
|If a machine dies|Stack is down|Containers reschedule onto other machines|
|Scaling|Manual (`docker compose up --scale`)|Automatic (HPA — Horizontal Pod Autoscaler)|
|Rolling updates|Brief downtime|Zero-downtime by default|
|Configuration|YAML (Compose v2)|YAML (much more of it)|
|Complexity|Tiny|Significant; weeks to learn properly|
|Right tool for...|Small apps, single server, dev/staging|Production at scale, multi-server, enterprise|

> 💡 You don't graduate _from_ Compose _to_ Kubernetes — they coexist. Compose for local dev; Kubernetes for production at scale. Many teams start on Compose for years before needing K8s.
### High-level architecture

A Kubernetes cluster has two kinds of machines:

```
   ┌─────────────────────────────────────────────────────────┐
   │                      CONTROL PLANE                      │
   │                  (the "brain" — 1 machine)              │
   │                                                         │
   │   • API Server     — every interaction goes through it  │
   │   • Scheduler      — decides which node runs which Pod  │
   │   • Controller Mgr — watches state, reconciles          │
   │   • etcd           — the cluster's database (key-value) │
   └─────────────────────────────────────────────────────────┘
                              │
                              │ assigns work
                              ▼
   ┌──────────────────┐  ┌──────────────────┐  ┌──────────────────┐
   │  Worker Node 1   │  │  Worker Node 2   │  │  Worker Node 3   │
   │                  │  │                  │  │                  │
   │  kubelet         │  │  kubelet         │  │  kubelet         │
   │  kube-proxy      │  │  kube-proxy      │  │  kube-proxy      │
   │  container       │  │  container       │  │  container       │
   │   runtime        │  │   runtime        │  │   runtime        │
   │                  │  │                  │  │                  │
   │  [Pod] [Pod]     │  │  [Pod] [Pod]     │  │  [Pod] [Pod]     │
   └──────────────────┘  └──────────────────┘  └──────────────────┘
```

**Control Plane components:**

- **API Server** — the front door. Every `kubectl` command, every internal component, talks to the API Server. Single source of truth.
- **Scheduler** — when a new Pod needs to run, the scheduler picks which worker node has room.
- **Controller Manager** — runs the "controllers" that watch the cluster's state vs the desired state and continuously reconcile (if a Pod dies, a controller notices and creates a replacement).
- **etcd** — a distributed key-value store. The state of every Pod, Service, ConfigMap, Secret lives here.

**Worker Node components:**

- **kubelet** — the agent on each worker; talks to the API Server, runs Pods locally as instructed.
- **kube-proxy** — handles networking on the node; implements Service IPs.
- **Container runtime** — the actual thing that runs containers (Docker, containerd, CRI-O).
### Kubernetes core primitives (preview)

You'll meet each of these in detail in Section 3:

|Object|One-line meaning|
|---|---|
|**Pod**|The smallest unit. Wraps 1+ containers that share a network and storage.|
|**ReplicaSet**|Maintains _N_ identical Pods at all times.|
|**Deployment**|Higher-level: rolling updates, rollbacks, history. Manages a ReplicaSet.|
|**Service**|A stable network address (and load balancer) for a set of Pods.|
|**Ingress**|Routes external HTTP/HTTPS traffic into Services based on hostname/path.|
|**ConfigMap**|Non-sensitive config injected as env vars or files.|
|**Secret**|Sensitive config (base64-encoded; can be encrypted at rest).|
|**PersistentVolumeClaim (PVC)**|A request for durable storage that survives Pod restarts.|
|**Namespace**|A logical partition of the cluster (dev / staging / prod can coexist).|
### Why Pods, not just containers?

A Pod = 1+ containers + shared network + shared storage volumes.

Why wrap a container in a Pod at all? Because some containers genuinely need to run together — for example, a logging "sidecar" container running alongside your app, both writing to the same volume. The Pod is the unit Kubernetes schedules and replicates.

In practice, **most Pods contain exactly one container**. You think of "a Pod" and "a container" as nearly the same thing 95% of the time.
### Declarative ≠ Imperative — the philosophy that makes K8s work

This is the single most important conceptual shift when moving from Docker to Kubernetes.

- **Imperative:** "Do this. Now do that. Now do that." (`docker run`, `docker stop`...)
- **Declarative:** "Here's what the world should look like. Make it so, and keep it that way."

You write a YAML manifest:

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: backend
spec:
  replicas: 3
  selector: { matchLabels: { app: backend } }
  template:
    metadata: { labels: { app: backend } }
    spec:
      containers:
        - name: backend
          image: ankit/mern-backend:1.0
          ports: [{ containerPort: 5000 }]
```

You apply it: `kubectl apply -f deployment.yaml`.

K8s now has a **goal**: _"3 backend Pods should always exist."_ If a Pod dies, it creates a new one. If you change the YAML to `replicas: 5` and re-apply, K8s spawns 2 more. If a worker node crashes, all the Pods it was running are rescheduled onto surviving nodes. Forever, on its own.

That continuous loop — _compare desired state to actual state, take action to close the gap_ — is called the **reconciliation loop**, and it's the heart of every controller in K8s.
### Why this matters in practice

With Compose, you ran a _command_ and the state changed _once_. If something drifted later, nothing put it back.

With K8s, you describe a _goal_, and the system continuously enforces it. Self-healing, drift-correction, and scaling all fall out of the same mechanism.
### Where K8s runs

You don't usually run a Kubernetes cluster yourself — it's complex. Instead:

|Provider|Service|
|---|---|
|AWS|EKS (Elastic Kubernetes Service)|
|Google Cloud|GKE (Google Kubernetes Engine) — the original|
|Azure|AKS (Azure Kubernetes Service)|
|DigitalOcean|DOKS|
|Self-hosted|kubeadm, Rancher|
|Local dev|Minikube, kind, Docker Desktop K8s, k3d|

Cloud providers run the control plane for you — you only manage the worker nodes (or even those are managed in Fargate / GKE Autopilot).

![[Pasted image 20260501065712.png | 1000]]

> 🔁 **Recap (K8s intro):** Kubernetes orchestrates containers across many machines. Two-tier architecture (control plane + worker nodes). Declarative model — you describe goals, K8s reconciles toward them. Pods wrap containers. Deployments wrap ReplicaSets which wrap Pods. We'll go deep in Section 3.3.

---
---

This concludes **Section 1 — DevOps Fundamentals & Core Tools**. So far we've covered:

- ✅ Linux (commands, users, SSH, processes, permissions, disk)
- ✅ Shell scripting
- ✅ Git & GitHub
- ✅ Environment management
- ✅ Docker (architecture, Dockerfile, layer caching, CLI)
- ✅ Docker Compose (full MERN, depends_on, env_file vs args)
- ✅ Continuous Integration (philosophy + branch protection)
- ✅ GitHub Actions, ESLint, Prettier (working YAML, secrets)
- ✅ AWS EC2 deployment (the manual flow + 4 deployment scenarios)
- ✅ Continuous Deployment (auto-deploy via SSH from Actions)
- ✅ Kubernetes introduction (architecture + declarative model)

Next: **Section 2 — Deep Dive into Docker & Jenkins**.

---
# SECTION 2 — DEEP DIVE INTO DOCKER & JENKINS

In Section 1 we used Docker as a black box to package apps. Now we open the box. Three topics need real depth before they ever feel comfortable: **volumes** (so data survives container restarts), **networking** (so containers can talk to each other and to the outside world), and **registries** (so built images can move from CI to production).

After that, we install Jenkins — and the volume + network + socket concepts you learn here will come back immediately when we mount `/var/run/docker.sock` into the Jenkins container.

---
## 2.1 — Docker Volumes (persistence deep dive)
### What problem are we solving?

A container's filesystem is **ephemeral**. When the container is removed (or even just recreated by `docker compose up --build`), everything written inside it is gone forever.

Imagine the consequences:

- You run a Mongo container. Users sign up, data accumulates, the database has 10,000 records.
- You bump the Mongo image version and recreate the container.
- Every record vanishes. Production is empty.

That's catastrophic. Containers are designed to be disposable, but **data isn't**. We need a way to keep data **outside** the container's filesystem so the container can be replaced without losing the data.

That's what **volumes** are for.

> 💡 **Mental model:** the container is a _light bulb_ — easy to swap. The volume is the _socket in the wall_ — fixed, persistent, the bulb plugs into it. Recreating the bulb (container) doesn't disturb the socket (volume).
### Three storage options Docker gives you

```
   Container's filesystem            Persists?         Use for
   ───────────────────────           ─────────         ─────────────────
   Container's writable layer        ❌ NO             Temporary scratch
   Bind mount (host path)            ✅ YES            Dev (live code reload)
   Named volume                      ✅ YES            Prod (databases, logs)
   tmpfs mount                       ❌ NO (RAM)       Secrets, scratch in RAM
```

We'll cover each. Volumes (named) and bind mounts are the workhorses; tmpfs is a niche tool.
### Bind Mount — "the host path is the source of truth"

A **bind mount** maps a directory on the host machine into a path inside the container. The container reads/writes to it directly.

```bash
# Run Mongo with a bind mount
docker run -d \
  --name mongo \
  -v /home/ankit/mongo-data:/data/db \
  mongo:6.0
#     │                    │
#     │                    └── path inside the container (Mongo's data dir)
#     └─────────────────────── path on the host machine
```

Now whatever Mongo writes to `/data/db` actually lands on the host at `/home/ankit/mongo-data`. Delete the container? The data is still on the host. Start a fresh Mongo container with the same bind mount? It picks up where it left off.

**When you'd use a bind mount:**

1. **Local development with live code reload.** Mount your source code into the container; edit on the host; the container sees changes immediately. No rebuilds.
    
    ```bash
    docker run -d -p 5000:5000 -v $(pwd)/server:/app my-backend:dev
    ```
    
2. **Mounting a config file.** A single Caddy/nginx config file on the host, made available read-only to the container.
    
    ```bash
    -v ./Caddyfile:/etc/caddy/Caddyfile:ro
    ```
    
3. **You explicitly want to manage data with normal Linux tools.** Backup with `cp`, inspect with `ls`, etc.

**Where bind mounts can hurt you:**

- **Path absolutism.** `-v /home/ankit/data:/data/db` only works on a machine that has `/home/ankit/data`. Move to a different host → broken.
- **Permission mismatches.** The user inside the container may have a different UID than the host owner; you get `Permission denied` errors.
- **Cross-platform pain.** A path like `/home/ankit` on Linux becomes `C:\Users\ankit` on Windows; Docker for Windows tries to abstract it but it's leaky.
- **Overwriting image content.** `-v $(pwd):/app` _replaces_ whatever was in `/app` from the image. If the image had `/app/node_modules` baked in, that disappears the moment you bind-mount source code over `/app`. Standard workaround: add an "anonymous volume" on top of the bind mount: `-v $(pwd):/app -v /app/node_modules`.
### Named Volume — "Docker manages the storage"

A **named volume** is a volume that Docker creates and manages somewhere on the host (typically under `/var/lib/docker/volumes/<name>/_data`), referenced by _name_, not by path.

```bash
# Create a named volume explicitly (optional — `-v mongo-data:...` auto-creates)
docker volume create mongo-data

# Use it
docker run -d \
  --name mongo \
  -v mongo-data:/data/db \
  mongo:6.0
#     │
#     └── just a name, NOT a path. Docker decides where the data lives.
```

In Compose:

```yaml
services:
  mongo:
    image: mongo:6.0
    volumes:
      - mongo-data:/data/db    # left side = volume name; right side = container path

volumes:
  mongo-data:                  # declare the named volume at the top level
```

**Why named volumes win in production:**

1. **Portable.** No host-path assumption. Works on every machine the same way.
2. **Permission-friendly.** Docker handles permissions appropriately for the image's user.
3. **Manageable via Docker.** `docker volume ls`, `docker volume inspect mongo-data`, `docker volume rm mongo-data`, `docker volume prune`.
4. **Backed up via Docker tooling.** You can `docker run --rm -v mongo-data:/data -v $(pwd):/backup alpine tar czf /backup/mongo.tgz /data` to back up.
5. **Survives `docker compose down`** (but NOT `docker compose down -v`).

> 💡 **Rule of thumb:** for **databases and any production state**, use **named volumes**. For **dev-time source code mounting and individual config files**, use **bind mounts**.
### tmpfs Mount — "RAM-only, never on disk"

A **tmpfs mount** stores data in the host's memory instead of on disk. Faster than disk, gone on container stop. Useful when:

- You want a scratch directory that's fast (build caches in CI).
- You want a place for sensitive temporary files that should never touch disk.

```bash
docker run -d \
  --name app \
  --tmpfs /tmp \
  --tmpfs /run:rw,size=64m \
  my-image
```

You'll rarely use this directly; it shows up implicitly when Docker mounts `/tmp` for some images.
### Inspecting volumes

```bash
docker volume ls
# DRIVER    VOLUME NAME
# local     mongo-data
# local     statusboard_caddy_data

docker volume inspect mongo-data
# [
#   {
#     "Name": "mongo-data",
#     "Driver": "local",
#     "Mountpoint": "/var/lib/docker/volumes/mongo-data/_data",   ← actual host path
#     "CreatedAt": "2026-04-29T...",
#     "Labels": null,
#     "Scope": "local"
#   }
# ]

# Show containers using a volume
docker ps -a --filter volume=mongo-data
```

You can `cd` into the `Mountpoint` (you may need root) and see the raw files. Don't edit them while the container is running — corruption risk for databases.
### Removing volumes (the "I lost my data" mistake)

```bash
docker volume rm mongo-data           # explicitly remove

docker compose down                   # keeps volumes — safe
docker compose down -v                # ALSO removes volumes — destructive

docker volume prune                   # remove every volume not in use by any container
docker system prune --volumes         # nukes everything unused
```

> ⚠️ Three commands every team has had a junior accidentally fire on production at 2 AM:
> 
> 1. `docker compose down -v`
> 2. `docker volume prune`
> 3. `docker system prune --volumes`
> 
> Treat them like `rm -rf /` — read twice, run once.
### A practical comparison

```bash
# BIND MOUNT — host path is the source of truth, dev-style
docker run -d \
  -v $(pwd)/server:/app \
  -v /app/node_modules \
  -p 5000:5000 \
  my-backend:dev

# NAMED VOLUME — Docker manages it, prod-style
docker run -d \
  -v mongo-data:/data/db \
  mongo:6.0

# READ-ONLY MOUNT — config you want the container to read but never modify
docker run -d \
  -v $(pwd)/Caddyfile:/etc/caddy/Caddyfile:ro \
  caddy:latest
#                                          │
#                                          └── ":ro" suffix = read-only
```
### Where this fits in the rest of this guide

|Where|What it persists|
|---|---|
|Mongo container (Compose)|Named volume `mongo-data` → MongoDB's `/data/db`|
|Caddy container (Section 4.5)|Named volume `caddy_data` → SSL certificates from Let's Encrypt|
|Prometheus container (Section 4.7)|Named volume `prometheus_data` → 30 days of metrics|
|Grafana container (Section 4.7)|Named volume `grafana_data` → custom dashboards & users|
|Uptime Kuma container (Section 4.6)|Named volume `uptime-kuma-data` → monitor configs & history|
|Jenkins container (Section 2.7)|Named volume `jenkins_home` → all jobs, plugins, configs|

Lose the right volume and the wrong things die. Recall from Section 4.5 that losing `caddy_data` makes Caddy re-request SSL certs from Let's Encrypt — and Let's Encrypt rate-limits you to 5 certs per domain per week, so frequent volume loss can lock you out of HTTPS for days.

![[Pasted image 20260501062206.png | 500]]

> 🔁 **Recap (Volumes):** Container layer = ephemeral. Bind mount = host path mounted in (dev). Named volume = Docker-managed, portable (prod). Always use named volumes for databases. `down -v` is the silent killer.

---
## 2.2 — Docker Networking

This is where every Docker beginner gets stuck for a day. We'll fix that with first principles.
### What problem are we solving?

Inside a single MERN stack:

- Backend container needs to talk to Mongo container.
- Frontend container needs to talk to Backend container.
- Browser (running on the host) needs to talk to Frontend container.

Each of these is a different _kind_ of conversation. The biggest beginner mistake — using `localhost` everywhere — is wrong for two of them. Let's fix the model.
### Mental model: each container is its own tiny computer

When Docker starts a container, the container gets:

- Its own filesystem (image layers + writable layer).
- Its own process tree (PID 1 inside is _not_ the host's PID 1).
- **Its own network stack** — its own loopback device, its own IP address, its own routing table.

This last point is what trips beginners. To a container, `localhost` (or `127.0.0.1`) means **itself**, not the host. So when your backend container sees `mongodb://localhost:27017`, it's looking for Mongo _inside its own network namespace_ — where there is no Mongo. It fails.

This is **the localhost trap**.
### The 4 network drivers

Docker has 4 built-in network types ("drivers"):

|Driver|When you'd use it|
|---|---|
|**bridge**|Default. Every container gets a private IP on a virtual switch (`docker0`)|
|**host**|Container shares the _host's_ network stack — no isolation|
|**none**|No networking at all (sandboxes, batch jobs)|
|**overlay**|Multi-host networking for Docker Swarm / cross-machine container talk|

We use **bridge** 95% of the time. Specifically, _user-defined bridges_, which add automatic DNS.
### The default bridge — and why we don't use it

When you `docker run` without specifying a network, the container joins the **default bridge** (named `bridge`):

```bash
docker run -d --name web nginx
docker network inspect bridge
# Shows containers on default bridge with their IPs (172.17.0.x).
```

What's wrong with it? **No DNS.** A container on the default bridge can reach another only by IP — which changes every restart. Hardcoding IPs is unworkable.
### User-defined bridges — and the magic of name-based DNS

When you create your own bridge network:

```bash
docker network create mern-net
docker run -d --name mongo --network mern-net mongo:6.0
docker run -d --name backend --network mern-net my-backend
```

Docker spins up an **embedded DNS server** for every user-defined bridge. Inside any container on `mern-net`:

- `mongo` resolves to Mongo's current IP.
- `backend` resolves to Backend's current IP.

The container _names_ become hostnames. Restart Mongo — its IP changes — but the name still resolves to the new IP. **Magic, courtesy of Docker DNS.**

Compose creates a user-defined bridge for each project automatically (named `<project>_default` unless you override). So in Compose:

```yaml
services:
  mongo:
    image: mongo:6.0          # service name "mongo" → DNS name on the network
  backend:
    build: ./server           # can use mongodb://mongo:27017 — works!
```

You don't need to declare the network — it's there.
### The 3 communication patterns (memorize this table)

```
   ┌─────────────────────────────────────────────────────────────────┐
   │                                                                 │
   │   Pattern A:  Container → Container                             │
   │                                                                 │
   │       backend  ──→  mongo                                       │
   │                                                                 │
   │       URL inside backend code:  mongodb://mongo:27017           │
   │       (use the SERVICE NAME, container's INTERNAL port)         │
   │                                                                 │
   ├─────────────────────────────────────────────────────────────────┤
   │                                                                 │
   │   Pattern B:  Browser/Host → Container                          │
   │                                                                 │
   │       browser  ──→  frontend (port 5173 published to host)      │
   │                                                                 │
   │       URL in the browser:  http://localhost:5173                │
   │       (use LOCALHOST, the PUBLISHED port, NOT the service name) │
   │                                                                 │
   ├─────────────────────────────────────────────────────────────────┤
   │                                                                 │
   │   Pattern C:  Container → Host's localhost                      │
   │                                                                 │
   │       container  ──→  service running on the host (rare)        │
   │                                                                 │
   │       URL inside container:  http://host.docker.internal:5432   │
   │       (special DNS name Docker provides; on Linux you may need  │
   │        --add-host=host.docker.internal:host-gateway)            │
   │                                                                 │
   └─────────────────────────────────────────────────────────────────┘
```
### Walking through a real bug

Beginner writes a backend `.env`:

```
MONGO_URI=mongodb://localhost:27017/dev_db
```

They `docker compose up`. Mongo container starts, backend container crashes. Logs say `connection refused`.

Why? Inside the backend container, `localhost` is the _backend container itself_. It has no Mongo. It can't reach Mongo on its own loopback.

**Fix:** change to:

```
MONGO_URI=mongodb://mongo:27017/dev_db
```

`mongo` is the service name in `compose.yaml`. Docker DNS resolves it. Connection succeeds.

This single bug accounts for ~40% of Docker support questions on the internet. Internalize the rule: **container talking to container → service name, never localhost.**
### `EXPOSE` vs `ports:` (also commonly confused)

|Directive|Effect|
|---|---|
|`EXPOSE 5000` in Dockerfile|Pure metadata — documents that the app listens on 5000. Opens nothing.|
|`-p 5000:5000` (or `ports: ["5000:5000"]`)|Actually publishes the port from the container to the host.|

Two containers on the same network can talk to each other on internal ports **without** publishing. The published port is only needed when something _outside_ Docker (your browser, an external curl) needs in.

A common pattern: backend listens on 5000 internally, never published, only the reverse proxy published on 80/443. Backend is unreachable from the internet directly — exactly what you want for security.
### Inspecting and debugging networks

```bash
docker network ls

docker network inspect mern-net
# Shows every connected container with its IP.

# From inside a container, test DNS:
docker exec -it backend sh
/ # nslookup mongo
/ # ping -c 2 mongo
/ # wget -qO- http://mongo:27017       # Mongo replies "It looks like you are trying to access MongoDB over HTTP" — proves connection works

# Disconnect a container from a network
docker network disconnect mern-net backend

# Connect to a different network (containers can be on multiple networks)
docker network connect monitoring backend
```
### The `host` driver (for completeness)

```bash
docker run -d --network host nginx
```

Now nginx isn't isolated — it shares the host's network namespace. `localhost` inside the container = the host. No port publishing needed (it's already on the host). Used for performance-critical services or for containers that need to bind to many host ports dynamically.

Cost: you lose isolation. Two containers using `--network host` and binding to the same port collide. Most apps shouldn't use it.
### Where this comes back

|Where|Networking concept used|
|---|---|
|Compose default behavior|Auto-creates a user-defined bridge per project|
|Backend → Mongo URL|`mongodb://mongo:27017` (service name)|
|Backend code calling another API|`http://other-service:port` if internal, `http://api.example.com` if external|
|Caddy reverse proxy (Section 4.5)|`reverse_proxy server:5000` — `server` is a service name on the same network|
|Prometheus → Node Exporter (Section 4.7)|`node-exporter:9100` — service name resolution|
|Grafana data source URL|`http://prometheus:9090` — same|
|Browser → app|`https://devopscourseankit.duckdns.org` (public IP / DNS — not a service name)|

![[Pasted image 20260501062523.png | 900]]

> 🔁 **Recap (Networking):** Each container has its own network namespace. `localhost` inside a container = that container. User-defined bridges give you DNS by container/service name. Container-to-container = service name. Browser-to-container = `localhost:<published-port>`. `EXPOSE` is metadata; `-p`/`ports:` is the real publishing.

---
## 2.3 — Docker Registry
### What problem are we solving?

You've built a Docker image on your laptop. How does it get to the production server?

Bad answers:

- **`docker save | scp | docker load`** — works once but is slow and manual.
- **Rebuild on the server.** Slow (uses production CPU), requires source code on the server, defeats the immutability promise.
- **Email the image as a tarball.** Don't laugh — it has happened.

The right answer: a **registry**. A registry is to Docker images what GitHub is to source code — a centralized server you push to and pull from.
### What a registry is

A **Docker Registry** is a service that stores Docker images and lets clients push (upload) and pull (download) them, addressed by name and tag. Each image lives at:

```
   <registry>/<namespace>/<repository>:<tag>

   Examples:
     docker.io/library/nginx:1.27.3-alpine
     docker.io/ankitsangwan/mern-backend:1.0
     ghcr.io/yourorg/yourapp:v2.1
     123456789012.dkr.ecr.us-east-1.amazonaws.com/myapp:latest
```

When the registry is `docker.io` (Docker Hub), the registry portion is omitted in commands — `nginx` is sugar for `docker.io/library/nginx:latest`.
### The major registries

|Registry|Run by|Best for|
|---|---|---|
|**Docker Hub** (`docker.io`)|Docker, Inc.|Public images, the default. Free with rate limits.|
|**GitHub Container Registry** (GHCR, `ghcr.io`)|GitHub|Public/private, integrated with GitHub Actions. Generous free tier.|
|**AWS ECR**|Amazon|Private images for AWS workloads. EKS pulls free.|
|**Google GAR / GCR**|Google|GKE, Cloud Run|
|**Azure ACR**|Microsoft|AKS|
|**JFrog Artifactory / Nexus**|Self-hosted|Enterprise; one place for all artifact types|
|**Harbor**|CNCF, self-hosted|Open-source, with image signing and vuln scanning|
|**Self-hosted `registry:2`**|Docker, Inc.|Roll your own registry on a machine you control|
### Pull = read. Push = write.

```bash
# PULL — download image from registry
docker pull nginx:1.27.3-alpine

# RUN — pulls automatically if not local
docker run -d nginx                # auto-pulls nginx:latest if missing

# Tag a local image so it can be pushed
docker tag mern-backend:1.0 ankitsangwan/mern-backend:1.0
#         │                  │
#         │                  └── new name (must include your username/namespace)
#         └─────────────────────── existing local name

# Authenticate to Docker Hub (one-time)
docker login                       # prompts for username + access token

# PUSH — upload to Docker Hub
docker push ankitsangwan/mern-backend:1.0

# Logout
docker logout
```

> 💡 Use a **Personal Access Token** (PAT), not your real password. Docker Hub → Account Settings → Security → New Access Token. PATs are revocable per-device and never expose your account password.
### Tagging strategy — the difference between sloppy and clean

|Tag strategy|When|
|---|---|
|`:latest`|Mutable, **never use in production**|
|`:1.0`, `:1.1`, `:2.0`|Semantic versioning — human-meaningful|
|`:<git-sha>` (e.g. `:a3b9c1`)|Immutable, traceable to the exact commit|
|`:<branch>-<sha>`|"main-a3b9c1" — supports parallel branches|
|`:<env>-<timestamp>`|"prod-2026-04-30-1830"|

A common combination: push `:<git-sha>` (immutable) AND `:latest` (convenience for local devs). Production deployments always reference the SHA tag — so a redeploy gets _exactly_ the same image, never a moving target.

```bash
# A typical CI build & push, in the runner
TAG=${GITHUB_SHA::7}                                           # short SHA
docker build -t ankitsangwan/mern-backend:$TAG -t ankitsangwan/mern-backend:latest ./server
docker push ankitsangwan/mern-backend:$TAG
docker push ankitsangwan/mern-backend:latest
```
### Public vs Private images

- **Public** — anyone can `docker pull`. Used for OSS images, demo images.
- **Private** — only authenticated users can pull. Used for proprietary code.

Docker Hub: Free tier = 1 free private repo + unlimited public. Beyond that: pay or move to GHCR (much more generous private tier).

Pulling private images on a server requires `docker login` first. In Kubernetes you provide a **`imagePullSecret`** referencing your registry credentials.
### Pushing from GitHub Actions to GHCR (the modern path)

This is what real CI/CD looks like. Add a job to your workflow:

```yaml
build-and-push:
  runs-on: ubuntu-latest
  permissions:
    contents: read
    packages: write              # required for GHCR pushes
  steps:
    - uses: actions/checkout@v4

    - name: Login to GHCR
      uses: docker/login-action@v3
      with:
        registry: ghcr.io
        username: ${{ github.actor }}
        password: ${{ secrets.GITHUB_TOKEN }}     # auto-provided by GitHub

    - name: Build and push backend
      uses: docker/build-push-action@v5
      with:
        context: ./server
        push: true
        tags: |
          ghcr.io/${{ github.repository }}/backend:${{ github.sha }}
          ghcr.io/${{ github.repository }}/backend:latest
```

The image is now sitting in `ghcr.io/yourorg/yourrepo/backend`. Production servers (or Kubernetes) pull from there, never rebuilding from source.
### Why "build in CI, pull in prod" is the right pattern

In Section 1.10's CD workflow, we did `git pull + docker compose build` _on the production server_. That works for one server but has problems:

|Problem with on-server builds|How a registry fixes it|
|---|---|
|Production CPU spent compiling, not serving users|Build happens in CI runner; prod just pulls|
|5 servers each build the image 5 times|Build once in CI, all 5 pull the same image|
|If build fails on prod, the prod is half-broken|Build fails in CI → prod never touched|
|Can't roll back; no immutable artifact|Tagged image is the artifact — rollback = pull old tag|
|Source code must live on the prod server|Prod has only the runtime, not the source|

The mature CD pipeline therefore looks like:

```
   Code  →  CI builds + tests + pushes image  →  Registry  →  Prod pulls + runs
```

We refine the CD workflow toward this pattern in Section 3 once Kubernetes / Helm enter the picture.
### Self-hosted registry (for completeness)

Docker provides an open-source `registry:2` image. Run it once and you have your own private registry:

```bash
docker run -d -p 5000:5000 --restart=always --name registry \
  -v registry-data:/var/lib/registry \
  registry:2
```

Now you can `docker push localhost:5000/myapp:v1`. Used in air-gapped enterprises, or behind a corporate VPN. In practice most teams pay for managed registries (ECR, GHCR) instead.

> 🔁 **Recap (Registry):** A registry stores images so they can move between machines. Docker Hub is the default; GHCR / ECR are the production-grade choices. Always tag with an immutable identifier (Git SHA) for production. The mature CI/CD pattern is "build in CI, push to registry, pull in prod" — never rebuild on the server.

---
## 2.4 — Volumes + Networking + Registry — Combined MERN Example

We now stitch all three concepts together into a single working production stack. This is the kind of `compose.yaml` you'd actually deploy to a real server.
### The architecture

```
   ┌─────────────────────────────────────────────────────────────┐
   │  HOST: production server                                    │
   │                                                             │
   │  ┌──────────────────────────────────────────────────────┐   │
   │  │  Docker Network "mern-net" (user-defined bridge)     │   │
   │  │                                                      │   │
   │  │   ┌──────────┐   ┌──────────┐   ┌──────────┐         │   │
   │  │   │ frontend │ → │ backend  │ → │  mongo   │         │   │
   │  │   │ :5173    │   │ :5000    │   │ :27017   │         │   │
   │  │   └──────────┘   └──────────┘   └──────────┘         │   │
   │  │       ▲              ▲              ▲                │   │
   │  └───────┼──────────────┼──────────────┼────────────────┘   │
   │          │              │              │                    │
   │       :5173          :5000        (NOT published)           │
   │      published      published     internal-only             │
   │          │              │              │                    │
   │  ┌───────┼──────────────┼──────────────┼─────────────────┐  │
   │  │       │              │              ▼                 │  │
   │  │  Host port    Host port      Named volume             │  │
   │  │     :5173        :5000       "mongo-data"             │  │
   │  └───────────────────────────────────────────────────────┘  │
   │                                                             │
   └─────────────────────────────────────────────────────────────┘
              ↑                  ↑
          Browser           curl / Postman
       http://VPS_IP:5173
       http://VPS_IP:5000
```

Notice:

- **Mongo is NOT published.** Only the backend can reach it (via the network's DNS). The internet cannot. Security win.
- **Frontend & backend are published**, because the browser needs them.
- **Mongo's data lives in a named volume** so DB content survives container recreation.
- **Service names are DNS** — backend's `MONGO_URI` is `mongodb://mongo:27017/...`.
- **Images can come from Docker Hub** (`mongo:6.0`) or be **built locally** (the backend & frontend Dockerfiles in our repo).
### compose.yaml

```yaml
services:
  mongo:
    image: mongo:6.0                      # ← pulled from Docker Hub
    container_name: mongo
    restart: unless-stopped
    volumes:
      - mongo-data:/data/db               # named volume → DB persistence
    networks:
      - mern-net
    # NOTE: no `ports:` — Mongo is internal-only

  backend:
    image: ankitsangwan/mern-backend:1.0  # ← from our private/public registry
    # Or, during dev:
    # build:
    #   context: ./server
    container_name: backend
    restart: unless-stopped
    env_file:
      - ./server/.env                     # MONGO_URI=mongodb://mongo:27017/db ← service-name DNS
    ports:
      - "5000:5000"                       # ← published so browser/Postman can reach
    depends_on:
      - mongo
    networks:
      - mern-net

  frontend:
    image: ankitsangwan/mern-frontend:1.0
    # Or:
    # build:
    #   context: ./client
    #   args:
    #     VITE_API_URL: http://VPS_IP:5000/api  ← bake in at BUILD time (1.4 gotcha)
    container_name: frontend
    restart: unless-stopped
    ports:
      - "5173:5173"
    depends_on:
      - backend
    networks:
      - mern-net

volumes:
  mongo-data:                              # the named volume — Docker manages it

networks:
  mern-net:
    driver: bridge                         # user-defined bridge → DNS by service name
```
### server/.env (created on the production server, NOT in Git)

```
PORT=5000
MONGO_URI=mongodb://mongo:27017/prod_db    # ← `mongo` is the service name; DNS resolves it
NODE_ENV=production
JWT_SECRET=<long-random-string>
```
### server/Dockerfile (cache-friendly, from 1.5)

```dockerfile
FROM node:18-alpine
WORKDIR /app
COPY package*.json ./
RUN npm ci --only=production
COPY . .
EXPOSE 5000
USER node                # run as non-root for security
CMD ["node", "index.js"]
```
### client/Dockerfile (multi-stage so we can bake in VITE_API_URL)

```dockerfile
# ----- Stage 1: build -----
FROM node:18-alpine AS builder
WORKDIR /app
ARG VITE_API_URL                              # build-arg from compose
ENV VITE_API_URL=$VITE_API_URL                # promote so `npm run build` sees it
COPY package*.json ./
RUN npm ci
COPY . .
RUN npm run build                             # bakes VITE_API_URL into dist/

# ----- Stage 2: serve -----
FROM nginx:alpine
COPY --from=builder /app/dist /usr/share/nginx/html
EXPOSE 80
CMD ["nginx", "-g", "daemon off;"]
```
### The full deploy sequence

On a freshly provisioned server:

```bash
# 1. Auth to your registry (one-time per server)
docker login                              # if pulling from Docker Hub
# OR
echo $GHCR_PAT | docker login ghcr.io -u ankitsangwan --password-stdin

# 2. Clone the repo (it has compose.yaml, but NOT the .env or images)
git clone https://github.com/ankitsangwan/mern-app.git
cd mern-app

# 3. Create production .env (generated on the server, never committed)
cat > server/.env <<'EOF'
PORT=5000
MONGO_URI=mongodb://mongo:27017/prod_db
NODE_ENV=production
JWT_SECRET=__GENERATED_LONG_RANDOM__
EOF

# 4. Pull the images that CI already pushed
docker compose pull                       # pulls all `image:`-referenced services

# 5. Bring everything up
docker compose up -d

# 6. Verify
docker compose ps
curl -s http://localhost:5000/api/health
docker compose logs -f mongo
```
### What this single file demonstrates

|Concept|Where it shows up|
|---|---|
|Named volume|`mongo-data` keeps DB across `docker compose up --build`|
|User-defined bridge|`mern-net` — gives DNS for service names|
|Service-name DNS|Backend's MONGO_URI uses `mongo` — not `localhost`, not an IP|
|Internal-only service|`mongo:` has no `ports:` — unreachable from internet|
|Public service|`frontend:` has `ports: 5173:5173` — reachable from browser|
|Pre-built images|`image: ankitsangwan/mern-backend:1.0` — pulled from registry, not built|
|Build args for Vite|`args: VITE_API_URL: ...` — baked at build time|
|`.env` separation|Loaded via `env_file:`, file is gitignored|
|`restart: unless-stopped`|Docker keeps the stack alive across crashes & host reboots|
|`depends_on:` start order|Mongo starts before backend; backend before frontend|
### Operating it day-to-day

```bash
# Update only the backend to a new image version
docker compose pull backend && docker compose up -d --no-deps backend
# `--no-deps` = don't restart Mongo just because backend was updated

# Roll back to the previous image
docker compose down
sed -i 's/mern-backend:1.1/mern-backend:1.0/' compose.yaml
docker compose up -d

# Check resource usage live
docker stats

# Backup the Mongo volume to a tarball
docker run --rm \
  -v mongo-data:/data \
  -v $(pwd):/backup \
  alpine tar czf /backup/mongo-$(date +%F).tgz /data

# Restore from backup
docker run --rm \
  -v mongo-data:/data \
  -v $(pwd):/backup \
  alpine sh -c "cd /data && tar xzf /backup/mongo-2026-04-29.tgz --strip-components=1"

# Tear everything down — KEEP volumes
docker compose down

# Tear everything down — DELETE volumes (you'll lose data!)
docker compose down -v
```
### The three traps you've now learned to avoid

1. **Using `localhost` in a container's DB URL.** ✗ → Use the service name.
2. **Forgetting the named volume on Mongo.** Every recreate wipes the database.
3. **Publishing Mongo's port to the internet.** Bots scan port 27017; default Mongo with no auth is the most-attacked DB on the internet.

> 🔁 **Recap (combined example):** Volumes for state, networks for talk, registries for distribution. The `compose.yaml` above is a real production-grade single-server stack, and every line of it is now explainable from first principles.

---

That closes the Docker deep-dive. Next: Jenkins — and the mental model you've just built (containers, volumes, sockets, networks, registries) is exactly what you need to understand why Jenkins-in-a-container with `/var/run/docker.sock` mounted is so clever.

---
## 2.5 — Jenkins Intro

### What problem are we solving?

We already have GitHub Actions doing CI/CD (Sections 1.8 and 1.10). It's free, it's right inside GitHub, it's YAML-driven. So why does Jenkins exist, and why is it still everywhere in industry?

Because GitHub Actions is _coupled to GitHub_. If your code lives on:

- **A self-hosted GitLab instance** behind your company's firewall,
- **A self-hosted Gitea / Bitbucket Server** because the company doesn't want code in any cloud,
- **An air-gapped network** with zero internet access,

then GitHub Actions is unavailable. You need a CI server _you control_, sitting _on your infrastructure_.

That's Jenkins. Open-source, self-hosted, runs on a server you own, integrates with anything via plugins. It predates GitHub Actions by more than a decade and is still the dominant CI engine in large enterprises (banks, telecoms, governments, automakers) for one reason: **they need to own the build pipeline end to end, on hardware they control, with auditability, and no third-party dependencies.**
### What Jenkins is

Jenkins is a **self-hosted automation server** that runs jobs (builds, tests, deploys) on schedules or in response to events (a Git push, a new tag, a manual button click). It's:

- **Open-source** (MIT license)
- **Java-based** (runs on any JVM-supported OS)
- **Plugin-driven** — there are 1,800+ plugins for nearly every tool you can name (Git, GitHub, GitLab, Docker, Kubernetes, Slack, Jira, JFrog, SonarQube, AWS, Terraform, Ansible…)
- **Capable of distributed builds** (one master coordinating many worker agents)

```
   Jenkins is to GitHub Actions
     what
   self-hosted Postgres is to managed AWS RDS.
```

Same concept; one is hosted-by-you, the other is hosted-by-someone-else.
### When to choose Jenkins vs GitHub Actions

|You want…|Use|
|---|---|
|Quick CI for a small team, code on GitHub|GitHub Actions|
|Code on private GitLab or Bitbucket Server, behind corp VPN|Jenkins (or GitLab CI)|
|Air-gapped or regulated environment (banking, healthcare)|Jenkins|
|Heavy custom build logic with shared libraries across many repos|Jenkins|
|Cheapest option ever|Either, both have free tiers|
|Don't want to maintain a CI server yourself|GitHub Actions / GitLab CI / CircleCI|
|Hundreds of microservices with shared pipeline templates|Jenkins (Shared Libraries) or GitLab CI templates|

> 💡 In practice, many modern teams **start** on GitHub Actions and **never need** Jenkins. Many enterprise teams have **always** been on Jenkins and have no reason to migrate. Both are valid.
### What you can do with Jenkins (the menu)

Once Jenkins is running, you can configure jobs that:

- Watch a Git repo and run on every push.
- Build Docker images and push to a registry.
- Run unit tests, lint, security scans.
- Deploy to staging, run integration tests, wait for human approval, deploy to prod.
- Schedule nightly database backups.
- Trigger another job in another team's Jenkins.
- Notify Slack/Teams/Email on success or failure.
- Generate signed binaries, PDF reports, code-coverage badges.

Anything you'd put in a shell script becomes a Jenkins job — with a UI, history, logs, retries, and notifications wrapped around it.
### Jenkins terminology you must know

|Term|Meaning|
|---|---|
|**Controller** (or **Master**)|The Jenkins server itself. Runs the UI, schedules jobs, stores config.|
|**Agent** (or **Node**)|A worker that actually executes the build. Can be the same machine as the controller (default) or a separate one.|
|**Job** / **Project**|A unit of work — "build the backend", "deploy to staging".|
|**Build**|One execution of a job. Has a number (#42), logs, status.|
|**Workspace**|The directory on the agent where the job's files live during a build.|
|**Pipeline**|A job defined as code (a `Jenkinsfile`) — multi-stage, expressive.|
|**Freestyle**|A job defined through the UI with point-and-click forms — limited.|
|**Plugin**|An extension that adds capabilities (Docker plugin, Git plugin, etc.).|
|**Credential**|An encrypted secret (SSH key, password, token) stored in Jenkins.|
|**Trigger**|What starts a job (poll SCM, webhook, schedule, manual).|
|**Stage** / **Step**|Sub-units of a Pipeline — `stage('Build') { ... steps { sh '...' } }`.|

> 🔁 **Recap (Jenkins intro):** Jenkins is the OG self-hosted CI server. You run it; you own the data; you customize via plugins. Strongest in enterprise, regulated, and air-gapped environments. Same job category as GitHub Actions, different deployment model.

---
## 2.6 — Jenkins Flow Architecture
### The big picture in one diagram

```
   ┌──────────────────────────────────────────────────────────────────┐
   │                  Developer pushes code to GitHub                 │
   └──────────────────────────────────────────────────────────────────┘
                                   │
                                   │  webhook (or poll)
                                   ▼
   ┌──────────────────────────────────────────────────────────────────┐
   │                       JENKINS CONTROLLER                         │
   │                                                                  │
   │   • Web UI (port 8080)                                           │
   │   • Job queue                                                    │
   │   • Job definitions (Jenkinsfiles)                               │
   │   • Credentials store                                            │
   │   • Plugins                                                      │
   │   • Build history & logs                                         │
   │                                                                  │
   │   When a job is triggered, the controller decides where it       │
   │   runs (on itself, or on an agent).                              │
   └──────────────────────────────────────────────────────────────────┘
                                   │
                                   │  dispatch
                                   ▼
   ┌─────────────────────┐  ┌─────────────────────┐  ┌──────────────────┐
   │   Built-in Agent    │  │   Linux Agent       │  │   Windows Agent  │
   │   (the controller   │  │   (build Linux apps,│  │   (build .NET    │
   │    itself)          │  │    Docker images)   │  │    apps, MSI)    │
   │                     │  │                     │  │                  │
   │   Workspace:        │  │   Workspace:        │  │   Workspace:     │
   │   /var/jenkins_home │  │   /home/jenkins/wks │  │   C:\jenkins\wks │
   └─────────────────────┘  └─────────────────────┘  └──────────────────┘
```
### Controller vs Agent

The **controller** is the brain:

- Hosts the web UI.
- Stores all the job configs (in `JENKINS_HOME`, default `/var/jenkins_home`).
- Coordinates the queue.
- Talks to plugins.

The **agent** is the muscle:

- Actually executes job steps (compiles code, runs Docker, runs tests).
- Has its own workspace directory.
- Can be the same machine as the controller (default for small setups) or a remote machine.

Why split them?

- **Scale.** One controller can dispatch to dozens of agents in parallel.
- **OS variety.** Build Windows apps on Windows agents, Mac apps on Mac agents, Linux on Linux.
- **Isolation.** Jobs from different teams use different agents to avoid collisions.
- **Capacity bursts.** Spin up agents on demand (e.g., as containers in Kubernetes via the Jenkins Kubernetes plugin).

For learning and small projects, the **built-in agent** (the controller running jobs on itself) is fine. We'll use it.
### What happens during a build, step by step

```
   1. Trigger fires (webhook from GitHub, or scheduled cron, or manual click)
              │
              ▼
   2. Controller selects an agent (matching labels/availability)
              │
              ▼
   3. Agent prepares the WORKSPACE
              ├─ creates a fresh directory if needed
              ├─ git clone (or git pull) the repo into workspace
              │
              ▼
   4. For each STAGE in the Pipeline:
              ├─ run each STEP
              ├─ stream stdout/stderr back to the controller
              ├─ if any step fails → fail the build (unless caught)
              │
              ▼
   5. Post-build actions
              ├─ archive artifacts (binaries, test reports)
              ├─ publish test results
              ├─ notify Slack / email / GitHub commit status
              │
              ▼
   6. Update job history; agent goes idle (or is destroyed if ephemeral)
```
### Where Jenkins stores everything: `JENKINS_HOME`

Every Jenkins controller has a directory called `JENKINS_HOME`. By default, on a Linux install, it's `/var/jenkins_home`. Everything Jenkins is — configs, jobs, plugins, build history, credentials — lives there.

```
/var/jenkins_home/
├── config.xml                  # global config
├── credentials.xml             # encrypted credentials
├── jobs/                       # one folder per job
│   └── mern-pipeline/
│       ├── config.xml          # job's config
│       └── builds/             # past build logs and metadata
│           ├── 1/
│           ├── 2/
│           └── 3/
├── plugins/                    # installed plugin .jpi files
├── secrets/                    # encryption keys (very sensitive)
├── users/                      # user accounts
└── workspace/                  # active build workspaces
    └── mern-pipeline/
```

This is **the** thing to back up. Lose it → lose every job, every credential, every build history. We'll mount it as a Docker volume in the next subsection precisely so it survives container recreation.
### Triggers — how jobs get started

Five triggers you'll meet:

1. **Manual** — click "Build Now" in the UI. Useful for one-offs.
2. **Poll SCM** — Jenkins runs `git ls-remote` on a schedule (e.g. every 5 min) to check for new commits. Wasteful but simple.
3. **Webhook** — GitHub sends Jenkins an HTTP POST when something happens. Instant. The proper choice.
4. **Scheduled / Cron** — `H 2 * * *` runs every night at ~2 AM. Used for nightly backups and reports.
5. **Triggered by another job** — Job A finishes → automatically starts Job B. Used for pipelines split across multiple jobs.

> 💡 Webhook setup, in two steps:
> 
> 1. In the GitHub repo: Settings → Webhooks → Add: `http://<jenkins-public-url>/github-webhook/`. Content type JSON.
> 2. In the Jenkins job: enable trigger "GitHub hook trigger for GITScm polling". Now every push to GitHub triggers the Jenkins job within seconds.

![[Pasted image 20260501063543.png | 1000]]

> 🔁 **Recap (Jenkins architecture):** Controller = brain, Agent = muscle. `JENKINS_HOME` holds all state — protect it. Builds are dispatched to agents and run in workspaces. Webhooks are the right trigger.

---
## 2.7 — Jenkins Setup with Docker and Docker Compose

There are several ways to install Jenkins:

|Method|When|
|---|---|
|Direct package install (apt/yum)|Traditional. You manage the Java, the service, etc.|
|Docker container|Cleanest for learning + small teams.|
|Docker Compose|Even cleaner — declarative setup.|
|Kubernetes Helm chart|Production-grade, autoscaling agents.|
|Cloud-managed (CloudBees Jenkins)|Pay someone to operate it.|

We'll use Docker Compose, which is the most teaching-friendly path and matches the rest of this guide.
### The Docker socket trick (this is critical)

Jenkins itself runs in a container. Inside, we want to run `docker build` to build _our_ images. But if Jenkins is in a container, where does it get a Docker daemon to talk to?

**Two wrong answers:**

1. **Run Docker inside Docker (DinD).** Spin up a separate Docker daemon inside the Jenkins container. Works, but heavy: nested daemons, complex networking, surprising failure modes.
2. **Install Docker CLI in Jenkins and somehow expect it to find a daemon.** It can't — there's no daemon to find inside the container.

**The right answer (the trick):**

Mount the **host's Docker socket** (`/var/run/docker.sock`) into the Jenkins container. Now Jenkins's `docker` CLI talks to the _host's_ daemon — which has access to all the host's images, networks, and containers.

```yaml
volumes:
  - /var/run/docker.sock:/var/run/docker.sock
```

> 🔁 **Recap from 2.1:** the daemon's socket file is the kernel of how `docker` CLI works. CLI sends instructions over the socket; daemon executes. Mounting the socket makes the container's CLI talk to the host's daemon.

Effect: `docker build` inside the Jenkins container builds an image that ends up on the _host_. `docker ps` inside Jenkins shows the host's containers — including itself.

> ⚠️ **This is also a security trade-off.** Anyone with access to the Jenkins container can issue arbitrary docker commands on the host — including starting privileged containers, mounting `/`, and effectively having root on the host. In production, lock down who can edit Jenkins jobs, or use a Docker-in-Docker setup with strict isolation, or use a dedicated builder API (BuildKit). For learning and small teams, the socket-mount pattern is standard and acceptable.
### compose.yaml for Jenkins

Create a folder `jenkins/` and put this `compose.yaml` inside:

```yaml
services:
  jenkins:
    image: jenkins/jenkins:lts                    # ← LTS = Long-Term Support, stable
    container_name: jenkins
    restart: unless-stopped
    user: root                                    # ← needed so it can read /var/run/docker.sock
    ports:
      - "8080:8080"                               # ← Jenkins web UI
      - "50000:50000"                             # ← agent connection port (for remote agents)
    volumes:
      - jenkins_home:/var/jenkins_home            # ← named volume → ALL Jenkins state persists
      - /var/run/docker.sock:/var/run/docker.sock # ← THE SOCKET TRICK
    environment:
      - JAVA_OPTS=-Djenkins.install.runSetupWizard=false  # ← (optional) skip wizard if seeding admin via JCasC

volumes:
  jenkins_home:                                   # named volume — Docker manages it
```
### Bring it up

```bash
cd jenkins
docker compose up -d

# Watch the logs to find the initial admin password
docker compose logs jenkins | grep -A 2 "Please use the following password"
# Output looks like:
# *************************************************************
# Jenkins initial setup is required. An admin user has been created
# and a password generated.
# Please use the following password to proceed to installation:
# 4f2c1e9b7a8d4d3a9e6c1b2f3d4e5f6a
# *************************************************************
```

The same password also appears in `/var/jenkins_home/secrets/initialAdminPassword` inside the container:

```bash
docker exec jenkins cat /var/jenkins_home/secrets/initialAdminPassword
```

Copy that password.
### Reaching the web UI

Open your browser:

```
http://localhost:8080            # if Jenkins runs on your laptop
http://<server-ip>:8080          # if Jenkins runs on an EC2 / VPS
```
### A sanity check

After Jenkins is up, prove the socket trick works:

```bash
docker exec -it jenkins bash
# inside the Jenkins container:
apt-get update && apt-get install -y docker.io     # install Docker CLI inside Jenkins
docker ps                                          # ← should list the host's containers, including jenkins itself
exit
```

You'll see Jenkins listing itself. That's the host's daemon answering.

> 💡 **In a real Jenkinsfile**, we don't apt-install docker each time — we either build a custom Jenkins image with docker baked in, or use the `docker` agent block in the Pipeline (which runs steps inside a Docker container directly).

> 🔁 **Recap (Jenkins setup):** Run Jenkins as a container with the host Docker socket mounted; persist `JENKINS_HOME` in a named volume. The Jenkins container can now build images using the host's daemon — exactly what CI needs. Get the initial admin password from logs or the file `/var/jenkins_home/secrets/initialAdminPassword`.

---
## 2.8 — Jenkins Admin Setup and Plugin Installation
### The "Unlock Jenkins" first-run wizard
![[Pasted image 20260418055602.png | 800]]

Open `http://localhost:8080`. You see _Unlock Jenkins_.

```
   ┌──────────────────────────────────────────────────────┐
   │  Unlock Jenkins                                      │
   │                                                      │
   │  An administrator password has been created and      │
   │  written to the log:                                 │
   │      /var/jenkins_home/secrets/initialAdminPassword  │
   │                                                      │
   │  Administrator password:                             │
   │  [____________________________________]              │
   │                                                      │
   │  [Continue]                                          │
   └──────────────────────────────────────────────────────┘
```

Paste the initial admin password from the previous section → Continue.
### Customize Jenkins → "Install Suggested Plugins"

Next page asks how you want to install plugins. Choose **Install Suggested Plugins**. This installs the standard set:

- Git plugin
- GitHub plugin
- Pipeline (with `Jenkinsfile` support)
- Credentials Binding
- Workspace Cleanup
- Build Timeout
- Email Extension
- Folders, Mailer, SSH Build Agents
- A handful of other essentials

Plugins install in parallel (1–3 minutes). Watch the green progress bars finish.
### Create the first admin user

Replace the temporary `admin/<long-password>` with a proper account:

```
Username:  ankit
Password:  ********
Full name: Ankit Sangwan
Email:     ankit@example.com
```

→ Save and Continue.
### Confirm Jenkins URL

Default is `http://localhost:8080`. Adjust if Jenkins runs on a server with a public IP/domain. → Save and Finish → Start using Jenkins.

You're in. Welcome to the dashboard.
![[Pasted image 20260418091225.png | 900]]
### Plugins to install on top of the suggested set

The suggested set is enough for "build a Java project, run a shell script". For Docker/MERN/modern workflows, install these via **Manage Jenkins → Plugins → Available**:

|Plugin|Why|
|---|---|
|**Docker Pipeline**|Adds `docker.image('node:18').inside { ... }` syntax to pipelines|
|**Docker plugin**|Lets Jenkins use Docker as a dynamic agent provider|
|**NodeJS Plugin**|Manages Node.js installations as a tool|
|**Blue Ocean**|Modern, prettier UI for pipelines (visual stage view)|
|**GitHub Branch Source**|Auto-create jobs for every branch + PR|
|**Credentials Binding**|Inject credentials into pipeline steps (already in suggested)|
|**HTML Publisher**|Publish build reports (e.g., test coverage HTML)|
|**Slack Notification**|Send messages on build success/fail|
|**Pipeline Stage View**|Visual pipeline progression (alternative to Blue Ocean)|

Click **Install without restart** and let them install. Some plugins prompt to restart Jenkins — do so.
### Configuring credentials

Go to: **Manage Jenkins → Credentials → System → Global credentials → Add Credentials**.

Common kinds:

|Kind|What it stores|Used for|
|---|---|---|
|Username with password|user/password|Docker Hub login, npm registry|
|SSH Username with private key|username + SSH private key|Git clone over SSH, deploy via SSH|
|Secret text|a single string|API tokens|
|Secret file|upload a file|Kubeconfig, GCP service account JSON|
|Certificate|a PKCS#12 cert bundle|TLS client auth|

Each credential has an **ID** (e.g., `dockerhub-creds`, `github-deploy-key`) which the pipeline references:

```groovy
withCredentials([usernamePassword(credentialsId: 'dockerhub-creds',
                                  usernameVariable: 'DH_USER',
                                  passwordVariable: 'DH_PASS')]) {
  sh 'echo $DH_PASS | docker login -u $DH_USER --password-stdin'
}
```

The actual values are encrypted in `JENKINS_HOME/credentials.xml` using the master key in `JENKINS_HOME/secrets/`.
### Global tool configuration

Go to: **Manage Jenkins → Tools**. Configure:

- **NodeJS** — add an installation: pick "Install automatically" → Node 18.x → name it `node18`. Pipelines can then `tool 'node18'` to use it.
- **Git** — usually auto-detected.
- **JDK / Maven / Gradle** — for Java projects.

This decouples job configuration from the underlying agent — every job that says "I need Node 18" gets it consistently.
### Securing Jenkins

A couple of must-dos before exposing Jenkins to the internet:

1. **Use a real domain + HTTPS.** Put Caddy or Nginx in front of port 8080 with a valid SSL cert.
2. **Restrict signup.** Manage Jenkins → Security → "Sign up disabled" (or only via invite).
3. **Apply Matrix-based authorization.** Different users get different permissions on different jobs.
4. **Audit Log plugin.** Track who changed what.
5. **CSRF protection enabled** (default since 2.0).
6. **Configuration as Code (JCasC) plugin** — describes the entire Jenkins config in YAML, version-controlled, reproducible.

> 🔁 **Recap (admin setup):** initial password from log/file → install suggested plugins + Docker plugins → create admin → configure tools (NodeJS) and credentials (DockerHub, GitHub). Now Jenkins is ready for jobs.

---
## 2.9 — Jenkins Web UI Walkthrough

Quick orientation around the main pages — once you can navigate confidently, you're not afraid of Jenkins anymore.
### The Dashboard (homepage)

```
┌─────────────────────────────────────────────────────────────────────┐
│  Jenkins                                                            │
│  Dashboard                                                          │
├─────────────────────────────────────────────────────────────────────┤
│   [+ New Item]   [People]   [Build History]   [Manage Jenkins]      │
│                                                                     │
│   ┌──────────────────────────────────────────────────────────────┐  │
│   │  S    W    Name              Last Success  Last Failure   #  │  │
│   │  ☀    ☀    mern-pipeline     5 min ago      —               │  │
│   │  ☀    ⛅   backend-build     2 hr ago       1 day ago       │  │
│   │  ⛈    ⛈   broken-job        —              5 min ago       │  │
│   └──────────────────────────────────────────────────────────────┘  │
│                                                                     │
│   Build Queue                Build Executor Status                  │
│   ─────────────              ───────────────────────                │
│   No builds in queue         #1 Idle                                │
│                              #2 Idle                                │
└─────────────────────────────────────────────────────────────────────┘
```

Things to notice:

- **S column = Status:** ☀ (sun) = recent builds passing; ⛅ = mixed; ⛈ (storm) = recent failures. Cute and intuitive.
- **W column = Weather:** trend over multiple builds.
- **Build Queue:** jobs waiting to start (because all executors are busy).
- **Build Executor Status:** how many parallel jobs can run on the controller (default: 2).
### Top navigation

|Item|What's there|
|---|---|
|**New Item**|Create a new job (Freestyle / Pipeline / Multibranch / Folder)|
|**People**|All users who've contributed (mainly authors of git commits)|
|**Build History**|Every build across every job, chronologically|
|**Manage Jenkins**|The settings universe (plugins, credentials, security, tools, system info)|
|**My Views**|Customizable dashboards if you want to filter to specific jobs|
### Inside a job's page

Click a job (e.g., `mern-pipeline`):

```
┌────────────────────────────────────────────────────────────┐
│  mern-pipeline                                             │
├────────────────────────────────────────────────────────────┤
│  [Build Now]   [Configure]   [Delete]   [Pipeline Syntax]  │
│                                                            │
│  Recent Builds                                             │
│  ──────────────                                            │
│  #42  ✅ 5 min ago    ✓ 2m13s                              │
│  #41  ❌ 10 min ago   ✗ 35s                                │
│  #40  ✅ 30 min ago   ✓ 2m05s                              │
│                                                            │
│  Stage View                                                │
│  ──────────                                                │
│   Checkout → Install → Lint → Test → Build → Push          │
│      ✓        ✓        ✓      ✓      ✓       ✓             │
└────────────────────────────────────────────────────────────┘
```

Buttons:

- **Build Now** — manually trigger a build.
- **Configure** — edit the job (the form for Freestyle jobs; nearly empty for Pipeline jobs since the config is in the `Jenkinsfile`).
- **Pipeline Syntax** — a snippet generator that helps you write `Jenkinsfile` lines correctly (especially `withCredentials` blocks).
### Inside a single build (e.g., `#42`)

```
┌─────────────────────────────────────────────────────────────┐
│  Build #42                                                  │
├─────────────────────────────────────────────────────────────┤
│  Status: SUCCESS                                            │
│  Started: 2026-04-30 10:14:21                               │
│  Duration: 2 min 13 sec                                     │
│  Triggered by: GitHub push by ankit                         │
│                                                             │
│  ┌─ Build Steps ─────────────────────────────────────────┐  │
│  │  ▸  Console Output    [view full log]                 │  │
│  │  ▸  Changes           (3 commits since last build)    │  │
│  │  ▸  Pipeline Steps    (visual, click stages)          │  │
│  │  ▸  Workspace         (browse files used in this run) │  │
│  │  ▸  Restart from Stage  (rerun from a failed point)   │  │
│  └───────────────────────────────────────────────────────┘  │
└─────────────────────────────────────────────────────────────┘
```

The most-used: **Console Output** (raw logs, your friend during debugging) and **Changes** (which commits this build covers — handy when something broke and you want to bisect).
### Manage Jenkins — the admin universe
![[Pasted image 20260418091309.png  | 1000]]
```
┌─────────────────────────────────────────┐
│  Manage Jenkins                         │
├─────────────────────────────────────────┤
│   System         (URL, email, env vars) │
│   Security       (auth, authorization)  │
│   Plugins        (install/update/remove)│
│   Tools          (Node, JDK, Maven)     │
│   Credentials    (secrets store)        │
│   Nodes          (agents)               │
│   System Information                    │
│   Logs           (Jenkins's own logs)   │
│   Statistics                            │
│   Reload Configuration from Disk        │
│   Script Console (run Groovy as admin)  │
└─────────────────────────────────────────┘
```

- **Script Console** — runs arbitrary Groovy code with admin privileges. Powerful but dangerous; restrict it.
- **Reload Configuration from Disk** — re-reads `JENKINS_HOME` after manual edits.

> 🔁 **Recap (UI):** Dashboard → job page → build page → console output. "Manage Jenkins" is the admin hub. Get familiar with these four pages and you'll never feel lost.

---
## 2.10 — Jenkins Freestyle Job

A **Freestyle** job is the original Jenkins job type — configured entirely through the web UI, with form fields for SCM, build steps, post-build actions. It's the simplest possible way to wire something up.
### Why use Freestyle (and why often not)

**Pros:**

- Visual, beginner-friendly. No code.
- Fastest path to "watch a repo and run a script."

**Cons:**

- Configuration lives only in `JENKINS_HOME` — not in Git.
- Hard to share between environments (dev/staging/prod Jenkins instances drift).
- Hard to express conditional logic, parallel stages, manual approvals.
- Can't be code-reviewed.
- Industry has moved to **Pipeline as Code** (`Jenkinsfile` in your repo). Pipeline is what you'll use 95% of the time.

We learn Freestyle because it teaches the underlying concepts (SCM, triggers, build steps, post-build actions) before the Pipeline DSL hides them.
### Building "Hello, Jenkins" as a Freestyle job
#### Step 1 — Create the job

Dashboard → **New Item** → enter a name (`hello-jenkins`) → choose **Freestyle project** → OK.
#### Step 2 — Configure SCM (Source Code Management)

Scroll to **Source Code Management** → select **Git** → set:

- **Repository URL:** `https://github.com/yourusername/hello-jenkins.git`
- **Credentials:** None (for public repos) or pick a credential ID (for private repos).
- **Branch Specifier:** `*/main`

Now every build will start by `git clone`ing this repo into the workspace.
#### Step 3 — Configure Build Triggers

Scroll to **Build Triggers**. Pick one:

- ☐ **Poll SCM** — schedule like `H/5 * * * *` (every 5 min). Polls the repo for changes.
- ☑ **GitHub hook trigger for GITScm polling** — proper webhook (recommended).
- ☐ **Build periodically** — runs on a schedule regardless of changes (e.g., nightly).
#### Step 4 — Configure Build Steps

Click **Add build step** → **Execute shell**:

```bash
echo "===== Build started ====="
echo "Workspace: $WORKSPACE"
echo "Build number: $BUILD_NUMBER"

ls -la

echo "Running tests..."
npm ci
npm test

echo "===== Build complete ====="
```

You can chain multiple build steps — first step compiles, second step runs tests, third runs `docker build`, etc. They run sequentially; if one fails, the rest are skipped.
#### Step 5 — Post-build Actions

These run after the build steps, regardless of pass/fail (mostly):

- **Archive the artifacts** — store build outputs (e.g., `dist/**/*.zip`) so you can download them later.
- **Publish JUnit test results** — make `*-results.xml` files visible as a test report.
- **Email Notification** — send mail to the committer on failure.
- **Build other projects** — kick off another job after this one succeeds.

Click **Save**.
#### Step 6 — Run a build

On the job page, click **Build Now**. A `#1` build appears at the bottom-left. Click it → **Console Output** to watch live logs:

```
Started by user ankit
Building in workspace /var/jenkins_home/workspace/hello-jenkins
Cloning repository https://github.com/yourusername/hello-jenkins.git
 > git init /var/jenkins_home/workspace/hello-jenkins
 > git fetch ...
Checking out Revision 8a3b9c1...
[hello-jenkins] $ /bin/sh -xe /tmp/jenkins1234.sh
+ echo '===== Build started ====='
===== Build started =====
+ echo 'Workspace: /var/jenkins_home/workspace/hello-jenkins'
...
+ npm test

> hello-jenkins@1.0.0 test
> jest

PASS  __tests__/hello.test.js
✓ should print hello (3ms)

Test Suites: 1 passed, 1 total
Tests:       1 passed, 1 total
+ echo '===== Build complete ====='
===== Build complete =====
Finished: SUCCESS
```

Green `SUCCESS` = the entire job worked.

If a step fails (e.g., a test fails, a shell command exits non-zero), Jenkins marks the build `FAILURE` and stops at that step. The console output shows exactly where it broke.
### What a Freestyle job teaches you

Even if you never use Freestyle in production, configuring this once internalizes:

- The **SCM section** (where the code comes from).
- The **Trigger section** (when builds start).
- The **Build Steps** (what runs).
- The **Post-build Actions** (what happens after).
- The **workspace lifecycle** (every build inherits the workspace; clean it with **Wipe out current workspace** if needed).

Pipeline jobs do the same things — just expressed in a `Jenkinsfile`.

> 🔁 **Recap (Freestyle):** form-driven, web-UI configured, fine for one-offs but not version-controlled. Teaches the underlying concepts; in real work you upgrade to Pipeline.

---
## 2.11 — Jenkins Pipeline & MERN Implementation
### The shift to Pipeline as Code

A `Jenkinsfile` is a Groovy-based DSL file checked into your repo's root that defines the entire pipeline as code:

- Stages (Checkout, Install, Lint, Test, Build, Push, Deploy)
- Steps within each stage
- Parallelism, retries, timeouts
- Credentials usage
- Notifications

Benefits over Freestyle:

- **Version-controlled** alongside the code. Reverting the code reverts the pipeline.
- **Reviewable.** Pipeline changes go through PR review.
- **Shareable.** Same `Jenkinsfile` works on multiple Jenkins controllers.
- **Branchable.** Each branch can have its own pipeline (try a new step on a feature branch without breaking main's pipeline).
### Two pipeline syntaxes

|Syntax|Look|Use when|
|---|---|---|
|**Declarative**|Structured blocks: `pipeline { agent { ... } stages { stage('X') { steps { ... } } } }`|95% of cases — easier, recommended|
|**Scripted**|Pure Groovy with `node { ... }` — full programming language|When you need complex control flow|

We'll use **Declarative**.
### Anatomy of a declarative Jenkinsfile

```groovy
pipeline {                                  // top-level block; must wrap everything
    agent any                               // run on any available agent

    environment {                           // env vars available in every stage
        IMAGE_NAME = 'ankitsangwan/mern-backend'
    }

    options {                               // pipeline-wide options
        timeout(time: 30, unit: 'MINUTES')  // fail if it takes >30 min
        timestamps()                        // prefix every log line with a timestamp
        buildDiscarder(logRotator(numToKeepStr: '10'))  // keep last 10 builds
    }

    stages {                                // ordered list of stages
        stage('Checkout') {
            steps {
                checkout scm                // clone the repo (`scm` is auto-injected)
            }
        }

        stage('Install') {
            steps {
                dir('server') {
                    sh 'npm ci'             // run shell — "sh" is the most common step
                }
            }
        }

        stage('Lint & Test') {
            parallel {                      // run sub-stages in parallel
                stage('Lint') {
                    steps { dir('server') { sh 'npm run lint' } }
                }
                stage('Test') {
                    steps { dir('server') { sh 'npm test' } }
                }
            }
        }

        stage('Build Image') {
            steps {
                sh "docker build -t ${IMAGE_NAME}:${env.BUILD_NUMBER} ./server"
                sh "docker tag ${IMAGE_NAME}:${env.BUILD_NUMBER} ${IMAGE_NAME}:latest"
            }
        }

        stage('Push to Registry') {
            steps {
                withCredentials([usernamePassword(
                    credentialsId: 'dockerhub-creds',
                    usernameVariable: 'DH_USER',
                    passwordVariable: 'DH_PASS')]) {
                    sh 'echo $DH_PASS | docker login -u $DH_USER --password-stdin'
                    sh "docker push ${IMAGE_NAME}:${env.BUILD_NUMBER}"
                    sh "docker push ${IMAGE_NAME}:latest"
                    sh 'docker logout'
                }
            }
        }

        stage('Deploy to Staging') {
            when { branch 'main' }          // only deploy from main branch
            steps {
                sshagent(['staging-ssh-creds']) {
                    sh '''
                        ssh -o StrictHostKeyChecking=no ec2-user@staging.example.com '
                            cd /opt/mern-app &&
                            docker compose pull &&
                            docker compose up -d
                        '
                    '''
                }
            }
        }
    }

    post {                                  // runs at the end, regardless of stage outcomes
        always {
            sh 'docker image prune -f'      // clean up regardless
        }
        success {
            echo '✅ Pipeline succeeded.'
        }
        failure {
            mail to: 'team@example.com',
                 subject: "Build failed: ${env.JOB_NAME} #${env.BUILD_NUMBER}",
                 body: "Check ${env.BUILD_URL} for details."
        }
    }
}
```
### What each block does

|Block|Purpose|
|---|---|
|`pipeline { }`|The outer wrapper. Required.|
|`agent`|Where the pipeline runs. `any` = any available; `none` = each stage chooses; `docker` = inside a container.|
|`environment`|Set env vars for the whole pipeline (or one stage).|
|`options`|Knobs: timeouts, retention, ANSI colors, timestamps.|
|`stages { stage('X') { steps { ... } } }`|The actual work, in order.|
|`parallel { }`|Run multiple stages simultaneously (lint and test together).|
|`when { }`|Conditional execution (branch matching, env values, expressions).|
|`post { always / success / failure / changed }`|What to do at the end.|
|`withCredentials { }`|Inject a credential as env vars for steps inside.|
### Common steps you'll use

|Step|Effect|
|---|---|
|`sh '<command>'`|Run a shell command (Linux/Mac).|
|`bat '<command>'`|Same, on Windows.|
|`checkout scm`|Clone the repo.|
|`dir('subdir') { ... }`|Run nested steps inside a subdirectory.|
|`withCredentials([...]) { ... }`|Inject secrets safely.|
|`withEnv(['KEY=VAL']) { ... }`|Add env vars for inner steps.|
|`archiveArtifacts artifacts: 'dist/**'`|Save build outputs.|
|`junit 'reports/**/*.xml'`|Publish test reports.|
|`input 'Deploy to prod?'`|Pause and wait for human approval.|
|`parallel { 'A': { ... }, 'B': { ... } }`|Inline parallel branches.|
|`retry(3) { ... }`|Retry the inner block up to 3 times on failure.|
|`timeout(time: 5, unit: 'MINUTES')`|Fail if the inner block takes too long.|
### Multibranch Pipeline — the production setup

A **Multibranch Pipeline** job watches a Git repo and **automatically creates a sub-job for every branch** that contains a `Jenkinsfile`. Open a feature branch with a `Jenkinsfile`? Jenkins discovers it within a minute and starts running the pipeline against it.

To set up:

Dashboard → New Item → Multibranch Pipeline. Configure:

- **Branch Sources:** GitHub → repo URL + credentials.
- **Build Configuration:** "by Jenkinsfile" → Script Path: `Jenkinsfile`.
- **Scan Triggers:** Periodically (e.g., every 5 minutes) or via webhook.

Now your job dashboard shows all branches with independent build histories. Open PRs even appear automatically. This is what real teams use.
### A complete Jenkinsfile for a MERN stack

Putting it all together — backend + frontend + Docker images + push + deploy:

```groovy
pipeline {
    agent any

    environment {
        BACKEND_IMG  = 'ankitsangwan/mern-backend'
        FRONTEND_IMG = 'ankitsangwan/mern-frontend'
        TAG          = "${env.BUILD_NUMBER}"
    }

    options {
        timeout(time: 20, unit: 'MINUTES')
        timestamps()
        buildDiscarder(logRotator(numToKeepStr: '20'))
    }

    stages {

        stage('Checkout') {
            steps { checkout scm }
        }

        stage('Backend - Install & Test') {
            steps {
                dir('server') {
                    sh 'npm ci'
                    sh 'npm run lint'
                    sh 'npm test'
                }
            }
        }

        stage('Frontend - Install & Test') {
            steps {
                dir('client') {
                    sh 'npm ci'
                    sh 'npm run lint'
                    sh 'npm run build'   // proves the Vite build works
                }
            }
        }

        stage('Build Docker Images') {
            parallel {
                stage('Backend Image') {
                    steps {
                        sh "docker build -t ${BACKEND_IMG}:${TAG} -t ${BACKEND_IMG}:latest ./server"
                    }
                }
                stage('Frontend Image') {
                    steps {
                        sh "docker build -t ${FRONTEND_IMG}:${TAG} -t ${FRONTEND_IMG}:latest \
                            --build-arg VITE_API_URL=https://api.your-prod.com ./client"
                    }
                }
            }
        }

        stage('Push to DockerHub') {
            when { branch 'main' }
            steps {
                withCredentials([usernamePassword(
                    credentialsId: 'dockerhub-creds',
                    usernameVariable: 'DH_USER',
                    passwordVariable: 'DH_PASS')]) {
                    sh 'echo $DH_PASS | docker login -u $DH_USER --password-stdin'
                    sh "docker push ${BACKEND_IMG}:${TAG}"
                    sh "docker push ${BACKEND_IMG}:latest"
                    sh "docker push ${FRONTEND_IMG}:${TAG}"
                    sh "docker push ${FRONTEND_IMG}:latest"
                    sh 'docker logout'
                }
            }
        }

        stage('Deploy to Staging') {
            when { branch 'main' }
            steps {
                sshagent(['staging-ssh-creds']) {
                    sh '''
                        ssh -o StrictHostKeyChecking=no ec2-user@staging.example.com "
                            cd /opt/mern-app &&
                            docker compose pull &&
                            docker compose up -d &&
                            docker image prune -f
                        "
                    '''
                }
            }
        }

        stage('Approve Production') {
            when { branch 'main' }
            steps {
                input message: 'Promote to PRODUCTION?', ok: 'Deploy'
            }
        }

        stage('Deploy to Production') {
            when { branch 'main' }
            steps {
                sshagent(['prod-ssh-creds']) {
                    sh '''
                        ssh -o StrictHostKeyChecking=no ec2-user@prod.example.com "
                            cd /opt/mern-app &&
                            docker compose pull &&
                            docker compose up -d &&
                            docker image prune -f
                        "
                    '''
                }
            }
        }
    }

    post {
        always {
            sh 'docker image prune -f || true'   // tidy up regardless
        }
        success {
            slackSend channel: '#deploys',
                      color: 'good',
                      message: "✅ ${env.JOB_NAME} #${env.BUILD_NUMBER} deployed."
        }
        failure {
            slackSend channel: '#deploys',
                      color: 'danger',
                      message: "❌ ${env.JOB_NAME} #${env.BUILD_NUMBER} FAILED — ${env.BUILD_URL}"
        }
    }
}
```

What this single file gives you, end-to-end:

1. Every push triggers a build.
2. Backend and frontend are linted, tested.
3. Both images are built in parallel (saves ~50% wall time).
4. On `main` branch only, images are pushed to Docker Hub with two tags (build number + `latest`).
5. Staging is auto-deployed.
6. A human approval gate guards production.
7. After approval, prod is deployed.
8. Slack is notified on success or failure.
### Common Jenkins pipeline mistakes

1. **Editing `Jenkinsfile` only in Jenkins's web editor instead of the repo.** Your changes don't survive; the next git push overwrites them. Always edit in your editor + commit to Git.
2. **Hardcoding credentials.** Use `withCredentials` always. The Jenkins log auto-masks variables bound this way.
3. **Forgetting `agent`.** Without it, the pipeline parser errors out cryptically.
4. **Using `bash` step on a Linux container that's Alpine** — Alpine has `sh`, not `bash`. Use `sh` for portability.
5. **Building images sequentially when they could be parallel.** Wraps every minute matters at scale.
6. **Not cleaning up Docker images.** Disk fills on the agent. Add `docker image prune -f` to the `post.always` block.
7. **No timeouts.** A bad test that hangs forever locks an executor. Always set `timeout`.
### How a Jenkins MERN pipeline differs from the GitHub Actions one

You've now seen both:

|Aspect|GitHub Actions (Section 1.10)|Jenkins (this section)|
|---|---|---|
|Where it runs|GitHub-hosted Linux VMs|Your own Jenkins server / agents|
|Where the workflow lives|`.github/workflows/*.yml`|`Jenkinsfile` in repo root|
|Trigger mechanism|GitHub events (push, PR)|Webhooks / polling|
|Secrets|GitHub Secrets (encrypted)|Jenkins Credentials store|
|Cost|Free tier + per-minute|Self-hosted = your hardware bill|
|Extensibility|Marketplace actions|1,800+ plugins|
|Audit|GitHub-managed|Fully under your control|

Both end up doing the same thing — automated lint, test, build, push, deploy — just via different engines.

> 🔁 **Recap (Pipelines):** Pipeline as Code lives in `Jenkinsfile`. Declarative syntax has `pipeline { agent { ... } stages { stage { steps { ... } } } post { ... } }`. Use `withCredentials` for secrets, `parallel` to speed things up, `when { branch 'main' }` to gate deploy stages. Multibranch jobs auto-discover branches. The MERN example above is a real production-grade pipeline.

---
---

This concludes **Section 2 — Deep Dive into Docker & Jenkins**. New content covered:

- ✅ Docker Volumes — bind mounts, named volumes, tmpfs, the `down -v` killer, volumes for every prod service later
- ✅ Docker Networking — the 4 drivers, user-defined bridges with name-based DNS, the **localhost trap**, the 3 communication patterns table, `EXPOSE` vs `ports:`
- ✅ Docker Registry — public vs private, tag strategies (immutable git SHA), `docker login`, push/pull, the "build in CI, pull in prod" pattern
- ✅ Combined MERN example — production-grade `compose.yaml` integrating all three concepts
- ✅ Jenkins introduction — what it is, when to choose it over GitHub Actions, terminology
- ✅ Jenkins architecture — controller/agent split, `JENKINS_HOME`, triggers, build lifecycle
- ✅ Jenkins setup with Docker + Compose — including the **`/var/run/docker.sock` trick** explained from first principles
- ✅ Admin & plugin setup — initial password, suggested plugins, must-add plugins, credentials, tools, security
- ✅ Web UI walkthrough — dashboard, job pages, build pages, Manage Jenkins
- ✅ Freestyle Job — full setup with all 4 sections (SCM, triggers, build steps, post-build)
- ✅ Pipeline & MERN — declarative syntax, all blocks, parallel stages, complete production-grade Jenkinsfile with build/test/push/deploy/approval/Slack

Next: **Section 3 — Advanced Orchestration & Networking** (Container-to-container, Nginx, Kubernetes Declarative, ConfigMaps + Secrets, Ingress, Storage).

---
# SECTION 3 — ADVANCED ORCHESTRATION & NETWORKING

Section 2 ended with a working production-grade `compose.yaml` for the MERN stack. But that stack still has problems we'll now fix:

1. **Backend and frontend are exposed on different ports** (`:5000`, `:5173`). Users see ugly URLs. There's no HTTPS. There's no caching layer.
2. **Docker Compose is a single-machine tool.** What happens when traffic 10x's and one server isn't enough?

Section 3 closes both gaps. First we'll deepen our understanding of how containers talk to each other (because mistakes here cause 80% of "it doesn't work" moments). Then we'll put **Nginx** as a reverse proxy in front of the stack. Then we'll graduate to **Kubernetes**, where the same ideas (containers, networks, configs, storage) reappear at cluster scale.

---
## 3.1 — Docker — Container-to-Container Communication

You learned in Section 2.2 that user-defined bridges give DNS by service name. This subsection drills further: _why_ it works, _when_ it doesn't, and the patterns for designing multi-service stacks.
### The mental model in one picture

```
   ┌──────────────────────────────────────────────────────────────┐
   │  HOST (e.g., your EC2 instance)                              │
   │                                                              │
   │  ┌────────────────────────────────────────────────────────┐  │
   │  │  Docker network: "mern-net" (a virtual switch)         │  │
   │  │                                                        │  │
   │  │   ┌──────────┐    ┌──────────┐    ┌──────────┐         │  │
   │  │   │ frontend │    │ backend  │    │  mongo   │         │  │
   │  │   │ 172.18.  │    │ 172.18.  │    │ 172.18.  │         │  │
   │  │   │  0.4     │    │  0.3     │    │  0.2     │         │  │
   │  │   └────┬─────┘    └────┬─────┘    └────┬─────┘         │  │
   │  │        │               │               │               │  │
   │  └────────┼───────────────┼───────────────┼───────────────┘  │
   │           │               │               │                  │
   │       :5173 published  :5000 published  (not published)      │
   │           │               │                                  │
   │           ▼               ▼                                  │
   │      Host port        Host port                              │
   │       :5173            :5000                                 │
   └───────────┼───────────────┼──────────────────────────────────┘
               │               │
        Browser/curl     Browser/curl
        from outside     from outside
```

The takeaway: every container has its **own IP** on the bridge (e.g., `172.18.0.3`). Docker's embedded DNS resolves the **service name** to that IP. When the IP changes (because a container restarted), the name still resolves correctly.
### How Docker DNS actually works

When you create a user-defined bridge network and attach containers to it, Docker:

1. Assigns each container an IP from the bridge's subnet (default `172.17.0.0/16` for the default bridge; user-defined bridges get fresh subnets like `172.18.0.0/16`).
2. Runs an **embedded DNS server** at `127.0.0.11` inside every container on that network.
3. Updates that DNS server's records whenever containers join, leave, or get a new IP.

To prove this, exec into a container:

```bash
docker exec -it backend sh
/ # cat /etc/resolv.conf
nameserver 127.0.0.11
options ndots:0
/ # nslookup mongo
Server:    127.0.0.11
Address:   127.0.0.11:53
Name:   mongo
Address: 172.18.0.2          ← the current IP of the mongo container
```

Restart Mongo and check again — the IP may change, but the name resolution always works.
### The 3 patterns, with concrete code
#### Pattern A — Container talks to container (use the service name)

The classic case from your `.env`:

```
# server/.env
MONGO_URI=mongodb://mongo:27017/dev_db
#                  ↑
#                  The service name from compose.yaml. NOT localhost. NOT an IP.
```

In code:

```javascript
// server/index.js
import mongoose from 'mongoose';
mongoose.connect(process.env.MONGO_URI);
```

When the backend container starts, it asks Docker DNS: "what's the IP of `mongo`?", gets `172.18.0.2`, opens a TCP connection to `172.18.0.2:27017`. Done.
#### Pattern B — Browser/host talks to container (use localhost + published port)

The browser doesn't speak Docker DNS. So the React frontend, running in your laptop's browser, can't say `http://backend:5000` — `backend` won't resolve in your browser.

Instead, the backend's port must be **published** to the host (`ports: ["5000:5000"]`), and the browser hits `http://localhost:5000` or, in production, `https://api.example.com`.

```yaml
# compose.yaml (snippet)
backend:
  ports:
    - "5000:5000"   # publishes — browser CAN reach it
mongo:
  # no ports: → NOT published — browser CANNOT reach it
```

```javascript
// client/src/api.js
const API = import.meta.env.VITE_API_URL;     // http://localhost:5000/api in dev
fetch(`${API}/tasks`).then(...);
```
#### Pattern C — Container talks to host's localhost (rare, but real)

You're running Mongo locally on your laptop (not in Docker), but the backend is in Docker. The backend container's `localhost` is itself, not your laptop. To reach the host, use the special DNS name `host.docker.internal`:

```bash
docker run -e MONGO_URI=mongodb://host.docker.internal:27017/db my-backend
```

On Linux, this name doesn't resolve out-of-the-box. Add:

```yaml
backend:
  extra_hosts:
    - "host.docker.internal:host-gateway"
```

This is mostly a development trick — production setups put both Mongo and backend in containers.
### Multi-network containers

A container can join **multiple** networks. This is how monitoring stacks isolate themselves from app stacks while still being able to scrape them:

```yaml
services:
  backend:
    networks:
      - mern-net           # talk to mongo, frontend
      - monitoring         # also reachable by Prometheus

networks:
  mern-net:
  monitoring:
    external: true         # this network was created elsewhere
```

The container shows up in two address spaces. Prometheus on `monitoring` can `scrape backend:9090`; the frontend on `mern-net` can hit `backend:5000`. They're literally the same container, dual-homed.
### Why "containers on different Compose projects can't see each other" (and the fix)

By default, each `compose.yaml` creates its own network named `<projectname>_default`. So if you have `app/compose.yaml` and `monitoring/compose.yaml` running in different folders, their containers are on different networks — service-name DNS doesn't work across them.

Fix: declare a shared **external network**:

```bash
# Create the shared network once
docker network create shared-net
```

```yaml
# app/compose.yaml
services:
  backend:
    networks: [shared-net, app-net]
networks:
  app-net:
  shared-net:
    external: true        # use the existing network, don't create new

# monitoring/compose.yaml
services:
  prometheus:
    networks: [shared-net, monitoring]
networks:
  monitoring:
  shared-net:
    external: true
```

Now `prometheus` can resolve `backend` despite being launched from a different Compose project.
### Common debugging recipe — "my containers can't talk!"

When two containers should be able to talk but can't:

```bash
# 1. Are they on the same network?
docker network ls
docker network inspect mern-net | grep -A 2 Containers

# 2. From container A, can it resolve container B?
docker exec -it backend sh
/ # nslookup mongo

# 3. From container A, can it reach container B's port?
/ # nc -zv mongo 27017            # netcat: 0 = open, 1 = closed
/ # wget -qO- http://other:5000/health

# 4. Is the target container actually listening on that port?
docker exec other netstat -tlnp     # or `ss -tlnp` on Alpine
```

99% of "they can't talk" problems are one of:

1. They're on different networks.
2. Wrong service name (typo).
3. Trying to use `localhost` from inside a container.
4. The target container hasn't actually started its listener yet (depends_on ≠ readiness — see Section 1.6).

![[Pasted image 20260501074814.png | 600]]

> 🔁 **Recap (3.1):** Containers on a user-defined bridge get a virtual IP and can resolve each other by service name through Docker's embedded DNS at `127.0.0.11`. Cross-project communication needs a shared external network. The 3-pattern table from Section 2.2 plus the debugging recipe above is everything you need.

---
## 3.2 — NGINX
### What problem are we solving?

Our MERN app currently exposes:

```
   http://3.91.123.45:5173   →  the React frontend
   http://3.91.123.45:5000   →  the Express backend
```

Five problems:

1. **Two different ports.** Users have to remember which is which.
2. **No HTTPS.** Everything is plain HTTP — passwords sent over the wire in clear.
3. **Port numbers in URLs are ugly.** `:5173` looks unprofessional.
4. **No way to add caching, rate limiting, header rewriting** without changing app code.
5. **CORS headaches.** Frontend on `:5173` calling backend on `:5000` is technically a different origin, requiring CORS configuration.

What we want:

```
   https://example.com/        →  React frontend
   https://example.com/api/*   →  Express backend
```

A single domain, single port (443), HTTPS, both services accessible through clean paths.

A **reverse proxy** sits in front of multiple services, accepts incoming requests on a public port, and routes each request to the right backend service based on URL, hostname, or other rules. Nginx is the most popular reverse proxy in the world.

> 💡 **Forward proxy vs reverse proxy:**
> 
> - **Forward proxy** sits in front of _clients_ and protects/anonymizes them (corporate proxy, VPN exit). The client knows about it.
> - **Reverse proxy** sits in front of _servers_ and protects/routes for them. The client doesn't know it's there — it thinks it's talking to one server.
> 
> Nginx, Caddy, HAProxy, and Traefik are all reverse proxies.
![[Pasted image 20260418171442.png | 500]]
### What Nginx is

Nginx (pronounced "engine-x") is:

- A high-performance HTTP server (originally a web server).
- A reverse proxy.
- A load balancer.
- A static file server.
- An SSL/TLS terminator.

All in one binary, written in C, asynchronous, capable of tens of thousands of concurrent connections per process. It's the dominant reverse proxy on the public internet.
### Nginx as a reverse proxy — the diagram

```
   ┌─────────────────────────────────────────────────────────────┐
   │  HOST (production server)                                   │
   │                                                             │
   │   Public port :80 / :443                                    │
   │            │                                                │
   │            ▼                                                │
   │   ┌────────────────────┐                                    │
   │   │   Nginx container  │   listens on :80                   │
   │   │                    │                                    │
   │   │   nginx.conf:      │                                    │
   │   │   • /     → web    │                                    │
   │   │   • /api/* → server│                                    │
   │   └─────┬────────┬─────┘                                    │
   │         │        │                                          │
   │         ▼        ▼                                          │
   │     ┌──────┐ ┌────────┐         ┌──────────┐                │
   │     │ web  │ │server  │ ──→     │  mongo   │                │
   │     │:5173 │ │:5000   │         │ :27017   │                │
   │     └──────┘ └────────┘         └──────────┘                │
   │     (not published)  (not published)  (not published)       │
   │                                                             │
   │   Only Nginx's :80 is published to the host.                │
   └─────────────────────────────────────────────────────────────┘
```

Notice: with Nginx in front, **none of your application containers need their ports published anymore**. They're internal-only. Only the proxy (Nginx) is exposed to the internet — a much smaller attack surface.
### A minimal nginx.conf for our MERN app

```nginx
# nginx.conf

events {}                                # required block, can be empty

http {
    # Gzip for faster transfer
    gzip on;
    gzip_types text/plain application/json application/javascript text/css;

    # Define upstream services (logical names → container references)
    upstream backend_servers {
        server backend:5000;             # service name from compose.yaml
    }

    upstream frontend_servers {
        server frontend:5173;
    }

    server {
        listen 80;
        server_name _;                   # match any hostname (for now)

        # Rule 1: anything starting with /api/ goes to the backend
        location /api/ {
            proxy_pass http://backend_servers;          # forward to backend service
            proxy_http_version 1.1;

            # Pass real client info to the backend (otherwise it sees only nginx's IP)
            proxy_set_header Host $host;
            proxy_set_header X-Real-IP $remote_addr;
            proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
            proxy_set_header X-Forwarded-Proto $scheme;
        }

        # Rule 2: everything else goes to the frontend
        location / {
            proxy_pass http://frontend_servers;
            proxy_http_version 1.1;
            proxy_set_header Host $host;
            proxy_set_header X-Real-IP $remote_addr;
            proxy_set_header Upgrade $http_upgrade;     # for Vite's hot-reload websockets
            proxy_set_header Connection "upgrade";
        }
    }
}
```

What happens at request time:

1. Browser requests `http://example.com/api/tasks`.
2. Nginx receives it on `:80`, matches `/api/` location, forwards to `http://backend:5000/api/tasks` via Docker DNS.
3. Express handles it, returns JSON.
4. Nginx forwards the response back to the browser.

For `http://example.com/` (no `/api`), it lands on the frontend container instead.
### Why those `proxy_set_header` lines matter

Without them, Express sees only `nginx-container-ip` as the client IP. That breaks:

- **Rate limiting.** You'd rate-limit nginx, not real users.
- **IP-based logging / fraud detection.** All traffic looks the same.
- **Knowing if the original request was HTTPS.** Without `X-Forwarded-Proto`, the backend doesn't know the user came in over HTTPS.

Express has middleware for this:

```javascript
app.set('trust proxy', 1);       // trust the first proxy in front (nginx)
// Now req.ip is the real client IP, not nginx's.
```
### Adding HTTPS to Nginx (Let's Encrypt + Certbot)

For HTTPS you need a TLS certificate. The most common path is **Let's Encrypt** (free) plus the **Certbot** ACME client. Roughly:

```bash
# On the host
sudo apt install -y certbot python3-certbot-nginx
sudo certbot --nginx -d example.com -d www.example.com
# Certbot fetches a cert, edits nginx.conf to add HTTPS, reloads nginx.

# Cron auto-renews every 60 days
sudo systemctl enable certbot.timer
```

After that you have an HTTPS-enabled `nginx.conf` like:

```nginx
server {
    listen 80;
    server_name example.com;
    return 301 https://$host$request_uri;       # force HTTPS
}

server {
    listen 443 ssl;
    server_name example.com;

    ssl_certificate     /etc/letsencrypt/live/example.com/fullchain.pem;
    ssl_certificate_key /etc/letsencrypt/live/example.com/privkey.pem;

    # ... same /api/ and / locations as before
}
```

> 💡 **This is exactly the manual labor that Caddy automates.** Caddy (Section 4.5) is "Nginx + automatic Let's Encrypt with zero config." If you understand Nginx + Certbot, you understand what Caddy is doing for you under the hood.
### Nginx in Docker Compose

```yaml
services:
  nginx:
    image: nginx:1.27-alpine
    container_name: nginx
    restart: unless-stopped
    ports:
      - "80:80"
      - "443:443"
    volumes:
      - ./nginx.conf:/etc/nginx/nginx.conf:ro     # mount our config (read-only)
      - ./certs:/etc/letsencrypt:ro               # mount certs (if using LE)
    depends_on:
      - frontend
      - backend
    networks:
      - mern-net

  backend:
    build: ./server
    # NOTE: removed `ports:` — backend is now internal-only
    env_file: ./server/.env
    networks:
      - mern-net

  frontend:
    build: ./client
    # NOTE: removed `ports:` — frontend is now internal-only
    networks:
      - mern-net

  mongo:
    image: mongo:6.0
    volumes:
      - mongo-data:/data/db
    networks:
      - mern-net
```

Notice that backend and frontend no longer have `ports:` blocks — they're only reachable through the proxy. The attack surface is now just port 80 and 443.
### The Vite ARG vs ENV deep dive (the bug that haunts every beginner)

We mentioned this in Sections 1.4 and 1.6. Now let's resolve it definitively.

**The setup that breaks:**

```yaml
frontend:
  build: ./client
  environment:
    - VITE_API_URL=https://example.com/api      # ← This DOES NOTHING for the build
```

You deploy. You open your site. The browser console says:

```
GET http://localhost:5000/api/tasks  net::ERR_CONNECTION_REFUSED
```

But you set `VITE_API_URL` to the production URL! Why is it calling localhost?

**Why this happens:**

Vite is a **build-time** tool. When you run `npm run build`, Vite reads `VITE_*` env vars _at that moment_ and _embeds the literal string_ into the compiled JavaScript bundle. After the build, `dist/assets/index-abc123.js` literally contains:

```js
fetch("http://localhost:5000/api/tasks")    // baked in at build time
```

Setting `VITE_API_URL` at _runtime_ (`environment:` in compose) is too late — the build already happened (during `docker build`), with the variable missing, so Vite fell back to the default in `vite.config.js` or `.env.development`.

**The fix — use `args:` so the variable is available at BUILD time:**

```yaml
frontend:
  build:
    context: ./client
    dockerfile: Dockerfile
    args:
      VITE_API_URL: https://example.com/api    # ← passed to the Dockerfile during build
```

And the Dockerfile must declare `ARG` and promote to `ENV`:

```dockerfile
FROM node:18-alpine AS builder
WORKDIR /app

ARG VITE_API_URL                       # accept the build arg
ENV VITE_API_URL=$VITE_API_URL          # promote so `npm run build` sees it

COPY package*.json ./
RUN npm ci
COPY . .
RUN npm run build                       # bakes VITE_API_URL into dist/

FROM nginx:alpine
COPY --from=builder /app/dist /usr/share/nginx/html
EXPOSE 80
```

> 💡 **The mental rule:**
> 
> - **`ENV`** = "this variable exists when the container is RUNNING". Backend env vars use ENV.
> - **`ARG`** = "this variable exists during BUILD". Frontend bundlers like Vite need ARG.
> 
> Backend reads env vars at runtime (`process.env.MONGO_URI`) — so `environment:` works. Frontend bakes env vars into static files at build time — so you must use `args:` + ARG.

After fixing this, you must rebuild:

```bash
docker compose build --no-cache frontend
docker compose up -d
```

(`--no-cache` is needed because the previous build already cached the wrong value.)
### Why this serves as the bridge to Nginx

A common pattern: serve the Vite-built static files directly with Nginx instead of running `vite preview`:

```dockerfile
# Stage 1: build with Node
FROM node:18-alpine AS builder
WORKDIR /app
ARG VITE_API_URL
ENV VITE_API_URL=$VITE_API_URL
COPY package*.json ./
RUN npm ci
COPY . .
RUN npm run build

# Stage 2: serve static files with Nginx (tiny, fast)
FROM nginx:alpine
COPY --from=builder /app/dist /usr/share/nginx/html
COPY nginx.conf /etc/nginx/conf.d/default.conf
EXPOSE 80
```

Now your "frontend container" is just Nginx serving static HTML/CSS/JS. ~30 MB. Boots in milliseconds. Caches well. This is the production frontend pattern.
### Common Nginx mistakes

1. **Forgetting to mount the config.** You edit `nginx.conf` on the host, but the container has the default. Always check that `volumes: - ./nginx.conf:/etc/nginx/nginx.conf:ro` is present.
2. **`proxy_pass http://backend:5000` without trailing slash gotcha.** Nginx's `proxy_pass` URL behavior changes based on whether your `location` ends with `/` and whether `proxy_pass` ends with `/`. If your routes look mangled, this is usually why.
3. **Forgetting to reload after editing config.** `docker compose exec nginx nginx -s reload` (or restart the container).
4. **`502 Bad Gateway` errors** — Nginx can't reach the upstream. Causes: typo in service name; backend container crashed; backend not yet ready; backend listening on a different port than declared.
5. **Not setting `proxy_set_header X-Forwarded-For`** — backend logs all show nginx's IP.
6. **Forgetting websocket headers** — Vite hot-reload, Socket.IO, all break without `Upgrade`/`Connection: upgrade`.

> 🔁 **Recap (Nginx):** A reverse proxy is the front door for all incoming traffic. Nginx routes by URL prefix to internal services. Use `proxy_set_header` to preserve client info. Frontend containers serving Vite output should use multi-stage builds (Node to build, Nginx to serve). Vite uses `ARG` (build time), backends use `ENV` (run time). Most production setups put backend & frontend behind a proxy and don't expose them directly.

---
## 3.3 — Kubernetes Declarative

We met Kubernetes in Section 1.11 — high level. Now we go deep on the **declarative model** and the core primitives. Read 1.11's intro again if it's faded; this section assumes you remember control plane vs worker nodes, Pods, and the reconciliation loop idea.
### The shift from imperative to declarative — once more, carefully

In Docker:

```bash
docker run -d --name backend -p 5000:5000 my-backend:1.0
```

You told Docker exactly what to do, _once_. If the container later crashes, Docker (with `--restart unless-stopped`) restarts it on the same machine. But if the _machine_ dies, nothing brings it back.

In Kubernetes, you write a YAML file:

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: backend
spec:
  replicas: 3                        # I want 3 backend pods, ALWAYS
  selector: { matchLabels: { app: backend } }
  template:
    metadata: { labels: { app: backend } }
    spec:
      containers:
        - name: backend
          image: my-backend:1.0
          ports: [{ containerPort: 5000 }]
```

You apply it: `kubectl apply -f backend.yaml`.

Now Kubernetes has a **goal**: _"3 backend pods should always exist."_ Forever after, controllers in the control plane keep checking: "Are there 3? No, only 2 — one died." Within seconds, K8s spawns a replacement on a healthy node. If a whole node fails, every Pod that was on it gets rescheduled elsewhere automatically.

That **continuous reconciliation** is the core of K8s. You describe _what_ you want; the system figures out _how_ to keep it that way.
### The kubectl tool

`kubectl` (often pronounced "cube-cuttle" or "cube-ctl") is the CLI that talks to the Kubernetes API server. Daily commands:

|Command|What it does|
|---|---|
|`kubectl version`|Show client + server versions|
|`kubectl get nodes`|List worker nodes in the cluster|
|`kubectl get pods`|List pods in current namespace|
|`kubectl get pods -A`|All namespaces|
|`kubectl get pods -o wide`|More columns (node, IP)|
|`kubectl get pods -w`|Watch live updates|
|`kubectl describe pod <name>`|Detailed pod info — events, resources, status|
|`kubectl logs <pod>`|Pod's stdout/stderr|
|`kubectl logs -f <pod>`|Follow logs live|
|`kubectl logs <pod> -c <container>`|Specific container in a multi-container pod|
|`kubectl exec -it <pod> -- sh`|Open a shell inside a pod|
|`kubectl apply -f file.yaml`|Create or update from YAML (idempotent)|
|`kubectl apply -f .`|Apply every YAML in the directory|
|`kubectl delete -f file.yaml`|Remove what was created|
|`kubectl delete pod <name>`|Manually delete a pod (its Deployment will recreate it)|
|`kubectl get all`|Pods, ReplicaSets, Deployments, Services in one view|
|`kubectl rollout status deployment/x`|Watch a rolling update finish|
|`kubectl rollout undo deployment/x`|Roll back to the previous version|
|`kubectl scale deployment/x --replicas=5`|Imperatively change replicas (also editable in YAML)|
|`kubectl port-forward pod/x 5000:5000`|Temporary local access to a pod's port|
|`kubectl edit deployment/x`|Open a live YAML editor (saves on close)|
|`kubectl explain pod.spec`|Built-in docs for any resource field|
|`kubectl config get-contexts`|Show clusters you have access to|
|`kubectl config use-context <name>`|Switch clusters|

> 💡 **`apply` is the workhorse.** Memorize this: you build up a folder of YAML manifests, and `kubectl apply -f .` brings them all into existence. Re-running it after an edit _updates_ whatever changed; nothing is destroyed. This is the declarative workflow.
### Pods — the smallest unit

A **Pod** is a wrapper around 1+ containers that:

- Share a network namespace (same IP, same localhost — Pod-internal containers reach each other on `localhost:port`).
- Share storage volumes.
- Are scheduled onto the same worker node.

Almost always, a Pod has **exactly one container**. The "1+ containers in a Pod" idea is reserved for tightly coupled helpers ("sidecars") — for example, a logging container that ships logs alongside the main app, or a service-mesh proxy.

A bare Pod manifest:

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: backend-pod
  labels:
    app: backend
spec:
  containers:
    - name: backend
      image: ankitsangwan/mern-backend:1.0
      ports:
        - containerPort: 5000
```

Apply: `kubectl apply -f backend-pod.yaml`. A Pod appears.

**You almost never create raw Pods in production**, because:

- If the pod dies, _nothing brings it back_. There's no controller above it.
- You can't roll a new version without manual stop-and-start.
- You can't have replicas.

We use Pods to _understand_ what's happening, then use higher-level objects (Deployments) to manage them.
### ReplicaSets — keep N pods running

A **ReplicaSet** continuously ensures that exactly N pods matching its label selector exist. If a pod disappears, it makes a new one.

```yaml
apiVersion: apps/v1
kind: ReplicaSet
metadata:
  name: backend-rs
spec:
  replicas: 3
  selector:
    matchLabels:
      app: backend
  template:                    # the Pod template — the ReplicaSet creates pods from this
    metadata:
      labels:
        app: backend           # MUST match selector.matchLabels
    spec:
      containers:
        - name: backend
          image: ankitsangwan/mern-backend:1.0
          ports:
            - containerPort: 5000
```

You also rarely create ReplicaSets by hand — you use a Deployment, which creates a ReplicaSet for you.
### Deployments — the everyday building block

A **Deployment** is a ReplicaSet with **rolling updates and history** on top. This is the object you create 95% of the time when launching an app on K8s.

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: backend
  labels:
    app: backend
spec:
  replicas: 3
  selector:
    matchLabels:
      app: backend
  strategy:
    type: RollingUpdate
    rollingUpdate:
      maxSurge: 1                # extra pods allowed during update
      maxUnavailable: 0          # how many can be missing during update (0 = zero downtime)
  template:
    metadata:
      labels:
        app: backend
    spec:
      containers:
        - name: backend
          image: ankitsangwan/mern-backend:1.0
          ports:
            - containerPort: 5000
          resources:
            requests:
              cpu: "100m"        # 0.1 CPU minimum guaranteed
              memory: "128Mi"
            limits:
              cpu: "500m"        # 0.5 CPU max
              memory: "512Mi"
          readinessProbe:
            httpGet:
              path: /api/health
              port: 5000
            initialDelaySeconds: 5
            periodSeconds: 10
          livenessProbe:
            httpGet:
              path: /api/health
              port: 5000
            initialDelaySeconds: 30
            periodSeconds: 30
```

Apply: `kubectl apply -f backend-deployment.yaml`.

Now:

- 3 backend pods exist.
- If any dies, a new one is started.
- If a node dies, pods reschedule to other nodes.
- Updating the image (`image: ...:1.1`) triggers a **rolling update**: K8s spawns new pods, waits for them to be ready, then terminates old ones — zero downtime.
#### Rolling update walkthrough

You change `image: ...:1.0` → `image: ...:1.1` and `kubectl apply`:

```
Initial:  [v1.0] [v1.0] [v1.0]                     (3 pods, all v1.0)
Step 1:   [v1.0] [v1.0] [v1.0] [v1.1-starting]     (maxSurge=1 → 4 pods briefly)
Step 2:   [v1.0] [v1.0] [v1.0] [v1.1-ready ✓]      (new pod ready)
Step 3:   [v1.0] [v1.0] [v1.1]                     (one v1.0 terminated)
Step 4:   [v1.0] [v1.0] [v1.1] [v1.1-starting]
...continues...
Final:    [v1.1] [v1.1] [v1.1]                     (3 pods, all v1.1)
```

`maxUnavailable: 0` means at no point are there fewer than 3 ready pods — true zero downtime.
#### Rollback in one command

```bash
kubectl rollout undo deployment/backend
```

K8s reverts to the previous ReplicaSet (still in history, just scaled to 0). Zero-downtime rollback. Compare to the SSH-based deploy from Section 1.10 where rolling back meant `git revert`, push, redeploy, hope.
#### Resource requests and limits

```yaml
resources:
  requests:                    # Guaranteed minimum
    cpu: "100m"                # 100 millicores = 0.1 of a CPU core
    memory: "128Mi"            # 128 mebibytes
  limits:                      # Hard cap
    cpu: "500m"
    memory: "512Mi"
```

- **Requests** = the scheduler uses these to decide which node has room.
- **Limits** = if a container exceeds CPU limit, it's throttled. If it exceeds memory limit, it's killed (OOMKilled).

In production, _always_ set both. Without limits, one bad pod can crash a whole node.
#### Probes — readiness vs liveness

Two checks K8s runs against your pod:

|Probe|Question|Effect on failure|
|---|---|---|
|**Readiness**|"Is this pod ready to receive traffic?"|Removes pod from Service load balancer (no traffic)|
|**Liveness**|"Is this pod still alive and not deadlocked?"|Restarts the pod|

You almost always want both. Readiness keeps half-started pods out of rotation; liveness recovers crashed-but-not-died pods.

```yaml
readinessProbe:
  httpGet: { path: /health, port: 5000 }
  initialDelaySeconds: 5      # don't probe for 5s after startup
  periodSeconds: 10           # then every 10s
livenessProbe:
  httpGet: { path: /health, port: 5000 }
  initialDelaySeconds: 30
  periodSeconds: 30
```

> ⚠️ **Setting livenessProbe too aggressively** kills slow-starting apps. The classic trap: a Java/Spring Boot app takes 60s to warm up; a 10s livenessProbe restarts it before it even starts. Always give a generous `initialDelaySeconds`.
### Services — stable network for unstable Pods

Pods come and go. Each restart gets a new IP. So you can never reliably point at a Pod's IP.

A **Service** is a stable, abstract endpoint that forwards traffic to all Pods matching a label selector. It gets:

- A stable virtual IP (the **ClusterIP**) that stays constant for its lifetime.
- A DNS name (`<service-name>.<namespace>.svc.cluster.local`).
- Load balancing across the matched pods.

```yaml
apiVersion: v1
kind: Service
metadata:
  name: backend                # → DNS name `backend` inside the cluster
spec:
  type: ClusterIP              # internal-only (default)
  selector:
    app: backend               # forwards to all pods labeled app=backend
  ports:
    - port: 5000               # the service's port
      targetPort: 5000         # the pod's port
```

Now anywhere inside the cluster, hitting `http://backend:5000` reaches one of the live backend pods. Pods can come and go; the Service is always there.
#### Service types

|Type|What it does|
|---|---|
|**ClusterIP**|Default. Internal-only. Reachable from inside the cluster.|
|**NodePort**|Opens a port (30000–32767) on every worker node. External traffic can reach it via `<any-node-ip>:<nodeport>`.|
|**LoadBalancer**|In a cloud, K8s asks the cloud (AWS/GCP/Azure) to provision a real load balancer with a public IP. This is how you expose a service to the internet on managed K8s.|
|**ExternalName**|Just a DNS alias to an external hostname (rare).|

In a managed cluster, putting a `Service` of type `LoadBalancer` in front of your backend gives you a real public IP within ~30 seconds. AWS gives an ALB, GCP gives a Google LB, etc.

> 💡 The `Service` is what makes the _Pod_ abstraction usable. Without Services, "the IPs change all the time" would make pod-to-pod talk impossible.
### Walkthrough: the kubernetes equivalent of our MERN compose

Recall the Compose file from 2.4. Here's the K8s version (without ConfigMap/Ingress yet — we add those in 3.4 and 3.5):

```yaml
# mongo-deployment.yaml
apiVersion: apps/v1
kind: Deployment
metadata: { name: mongo }
spec:
  replicas: 1
  selector: { matchLabels: { app: mongo } }
  template:
    metadata: { labels: { app: mongo } }
    spec:
      containers:
        - name: mongo
          image: mongo:6.0
          ports: [{ containerPort: 27017 }]
---
# mongo-service.yaml
apiVersion: v1
kind: Service
metadata: { name: mongo }
spec:
  selector: { app: mongo }
  ports: [{ port: 27017, targetPort: 27017 }]
---
# backend-deployment.yaml
apiVersion: apps/v1
kind: Deployment
metadata: { name: backend }
spec:
  replicas: 3
  selector: { matchLabels: { app: backend } }
  template:
    metadata: { labels: { app: backend } }
    spec:
      containers:
        - name: backend
          image: ankitsangwan/mern-backend:1.0
          ports: [{ containerPort: 5000 }]
          env:
            - name: MONGO_URI
              value: "mongodb://mongo:27017/prod_db"   # ← `mongo` is the Service DNS name!
---
# backend-service.yaml
apiVersion: v1
kind: Service
metadata: { name: backend }
spec:
  selector: { app: backend }
  ports: [{ port: 5000, targetPort: 5000 }]
---
# frontend-deployment.yaml
apiVersion: apps/v1
kind: Deployment
metadata: { name: frontend }
spec:
  replicas: 2
  selector: { matchLabels: { app: frontend } }
  template:
    metadata: { labels: { app: frontend } }
    spec:
      containers:
        - name: frontend
          image: ankitsangwan/mern-frontend:1.0
          ports: [{ containerPort: 80 }]
---
# frontend-service.yaml
apiVersion: v1
kind: Service
metadata: { name: frontend }
spec:
  selector: { app: frontend }
  ports: [{ port: 80, targetPort: 80 }]
```

Save all in a `k8s/` folder, then:

```bash
kubectl apply -f k8s/
```

Within a minute:

- `kubectl get pods` → 6 pods (1 mongo + 3 backend + 2 frontend).
- `kubectl get services` → 3 services with ClusterIPs.
- Backend can connect to Mongo via `mongo:27017`.
- Frontend can call backend via... well, that's where Ingress comes in (Section 3.5).
### Order of operations — apply in the right sequence

If you split your manifests across files and `kubectl apply -f .` them, generally apply in this order:

```
1. Namespace               (so everything else has a place to land)
2. ConfigMap & Secret      (so deployments can reference them)
3. PersistentVolumeClaim   (so deployments can mount them)
4. Deployments / StatefulSets
5. Services                (so traffic can route)
6. Ingress                 (so external traffic can enter)
```

In practice `kubectl apply -f .` applies them all together and K8s reconciles eventually — pods may CrashLoopBackOff briefly until their dependencies appear. Order matters mostly for cleanliness and faster startups.

![[Pasted image 20260501070221.png | 700]]
> 🔁 **Recap (3.3 declarative):** YAML manifests describe desired state. `kubectl apply` registers that state with the API server, which triggers controllers that reconcile reality toward the desired state. Pods are wrapped by ReplicaSets which are wrapped by Deployments. Services give Pods stable networking. The whole MERN stack expresses as a few dozen lines of YAML.

---
## 3.4 — Configuration and Secrets in Kubernetes
### What problem are we solving?

Look at the backend Deployment we just wrote:

```yaml
env:
  - name: MONGO_URI
    value: "mongodb://mongo:27017/prod_db"
```

Putting config inline like this is fine for one variable. But:

- A real backend has 10+ env vars.
- Some are secrets (`JWT_SECRET`, `DB_PASSWORD`).
- The same Deployment YAML should work for dev/staging/prod with different values.
- You want to update config without rebuilding the image OR editing the deployment YAML.

K8s gives you two purpose-built objects:

- **ConfigMap** — non-sensitive config (URLs, feature flags, log levels).
- **Secret** — sensitive config (passwords, tokens, certificates).

Both are decoupled from your Deployment YAML — you reference them.
### ConfigMap

A ConfigMap stores key-value pairs (or whole files) of plain configuration data.

```yaml
# backend-configmap.yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: backend-config
data:
  NODE_ENV: "production"
  LOG_LEVEL: "info"
  MONGO_URI: "mongodb://mongo:27017/prod_db"
  RATE_LIMIT_WINDOW: "60"
  RATE_LIMIT_MAX: "100"
```

```bash
kubectl apply -f backend-configmap.yaml
kubectl get configmap backend-config -o yaml
```
#### Two ways to use a ConfigMap from a Deployment

**Option 1 — inject as env vars (most common):**

```yaml
spec:
  template:
    spec:
      containers:
        - name: backend
          image: ankitsangwan/mern-backend:1.0
          envFrom:                              # bulk-inject all keys
            - configMapRef:
                name: backend-config
          # OR cherry-pick:
          # env:
          #   - name: NODE_ENV
          #     valueFrom:
          #       configMapKeyRef:
          #         name: backend-config
          #         key: NODE_ENV
```

After `kubectl apply`, the running container has env vars `NODE_ENV=production`, `LOG_LEVEL=info`, etc. — read by `process.env.NODE_ENV` in code, exactly like a `.env` file.

**Option 2 — mount as files in a directory:**

```yaml
volumes:
  - name: config-volume
    configMap:
      name: backend-config
volumeMounts:
  - name: config-volume
    mountPath: /etc/config
```

Now `/etc/config/` inside the container has files: `NODE_ENV`, `LOG_LEVEL`, etc., each containing its value. Useful for apps that read config from files instead of env vars (like a Caddyfile or nginx.conf injected from a ConfigMap).
#### Why ConfigMaps are powerful

- **Update config without rebuilding the image.** Edit the ConfigMap, restart pods (`kubectl rollout restart deployment/backend`), new config is live.
- **Different ConfigMaps per environment.** `backend-config-dev`, `backend-config-prod` — same Deployment YAML, different ConfigMap reference.
- **Multiple Deployments share one ConfigMap** if they need the same config.
### Secret

Identical structure to ConfigMap — but values are **base64-encoded** and treated as sensitive (limited access, optionally encrypted at rest, masked in `kubectl describe`).

```yaml
# backend-secret.yaml
apiVersion: v1
kind: Secret
metadata:
  name: backend-secret
type: Opaque
data:
  JWT_SECRET: bXktcmVhbC1zZWNyZXQta2V5LWdvLWhlcmU=     # base64 of the actual value
  DB_PASSWORD: cGFzc3dvcmQxMjM=
```

To encode a value:

```bash
echo -n 'my-real-secret-key-go-here' | base64
# → bXktcmVhbC1zZWNyZXQta2V5LWdvLWhlcmU=
```

Or — much easier — let `kubectl create` do it:

```bash
kubectl create secret generic backend-secret \
  --from-literal=JWT_SECRET='my-real-secret-key' \
  --from-literal=DB_PASSWORD='password123'
```

Or from a file:

```bash
kubectl create secret generic ssl-certs \
  --from-file=tls.crt=./fullchain.pem \
  --from-file=tls.key=./privkey.pem
```
#### Using Secrets in a Deployment

Same patterns as ConfigMap — `envFrom` or `secretKeyRef`:

```yaml
spec:
  template:
    spec:
      containers:
        - name: backend
          image: ankitsangwan/mern-backend:1.0
          envFrom:
            - configMapRef:
                name: backend-config
            - secretRef:
                name: backend-secret           # secrets injected as env vars
```

In code, `process.env.JWT_SECRET` returns the decoded value. K8s does the base64 decode automatically.
#### Important caveats about Secrets

1. **Base64 is not encryption.** It's just encoding. Anyone with `kubectl get secret -o yaml` access can decode them in seconds. The real protection is **RBAC** — restrict who can `get secret`.
2. **Encryption at rest is opt-in.** Enable encryption in etcd (the cluster's database) so Secrets aren't stored in plaintext on disk.
3. **For real secret management at scale**, use external secret stores: HashiCorp Vault, AWS Secrets Manager, Google Secret Manager. Sync via tools like External Secrets Operator.
4. **Never commit Secret YAMLs with real values to Git.** Either generate them imperatively (`kubectl create secret`), or use **Sealed Secrets** / **SOPS** to encrypt the YAML before committing.

> ⚙️ **Added Professional Context — Sealed Secrets:** Bitnami's Sealed Secrets is a controller you install in the cluster. You encrypt your `Secret` YAML with `kubeseal` using the cluster's public key, getting back a `SealedSecret` YAML. That encrypted YAML is safe to commit to Git. The controller decrypts it inside the cluster into a real Secret. This is the standard GitOps-friendly way to handle secrets.
### A complete dev/prod separation pattern

You build _one_ set of Deployment YAMLs. Your config differs across environments by swapping ConfigMaps and Secrets:

```
k8s/
├── base/
│   ├── backend-deployment.yaml      ← references `backend-config` and `backend-secret`
│   ├── backend-service.yaml
│   ├── frontend-deployment.yaml
│   └── ...
├── overlays/
│   ├── dev/
│   │   ├── backend-config.yaml      ← dev URLs
│   │   └── backend-secret.yaml      ← dev secrets (or external-secrets reference)
│   └── prod/
│       ├── backend-config.yaml      ← prod URLs
│       └── backend-secret.yaml      ← prod secrets
```

Deploy:

```bash
kubectl apply -f k8s/base -f k8s/overlays/dev    # for dev cluster
kubectl apply -f k8s/base -f k8s/overlays/prod   # for prod cluster
```

Tools like **Kustomize** (built into kubectl) and **Helm** (Section B.4) formalize this pattern.

> 🔁 **Recap (3.4):** ConfigMap = non-sensitive config. Secret = sensitive config (base64-encoded, RBAC-protected). Both are referenced from Deployments via `envFrom` or `valueFrom`. Updates don't need image rebuilds. For real secret management, use Sealed Secrets or external secret stores.

---
## 3.5 — Ingress
### What problem are we solving?

We have Services. To make a service reachable from the internet, we'd give it `type: LoadBalancer` — and the cloud provisions a load balancer with a public IP.

But what if you have **5 services** that should each be reachable from the internet? You'd need 5 LoadBalancers — 5 public IPs, 5 monthly bills, 5 separate certificates.

What if you want them all reachable through one domain by URL path?

```
https://example.com/         → frontend service
https://example.com/api/     → backend service
https://example.com/admin/   → admin-panel service
https://example.com/grafana/ → monitoring service
```

You need a **smart entry point** — a single LoadBalancer behind one domain, that inspects the URL and routes to the right Service. That's the role of **Ingress**.
### What Ingress is

An **Ingress** is a Kubernetes resource that defines HTTP(S) routing rules: "this hostname + path goes to that Service." But the Ingress object itself is just configuration — it does nothing until you install an **Ingress Controller**, which is the actual proxy (nginx, Traefik, Caddy, HAProxy, AWS ALB Controller, etc.) that reads the Ingress objects and applies them.

Two layers:

```
   ┌──────────────────────────────────────────────────────┐
   │  Ingress Controller (a Pod, e.g. nginx-ingress)      │
   │  Has a LoadBalancer Service in front of it →         │
   │  one public IP for the whole cluster.                │
   │                                                      │
   │  Reads all Ingress resources, builds nginx.conf,     │
   │  reloads. Receives all incoming HTTP/HTTPS traffic.  │
   └──────────────────────────────────────────────────────┘
                            ↑
                            │
              Ingress resources (YAML you write)
              describe routing rules:
                /         → frontend Service
                /api/*    → backend Service
                /grafana  → grafana Service
```

> 💡 **Mental model:** Ingress = Nginx (Section 3.2) but Kubernetes-native. You write YAML to describe the routing; the Ingress Controller (running as a Pod) compiles it into actual nginx config.
### Installing an Ingress Controller

The most popular is **ingress-nginx** (the official nginx-based one). You install once per cluster:

```bash
# Cloud-managed: usually one of:
helm install ingress-nginx ingress-nginx/ingress-nginx --namespace ingress-nginx --create-namespace

# Or apply the official manifest:
kubectl apply -f https://raw.githubusercontent.com/kubernetes/ingress-nginx/main/deploy/static/provider/cloud/deploy.yaml
```

This creates:

- A `Deployment` of nginx-ingress controller pods.
- A `Service` of type LoadBalancer in front of those pods → cloud provisions a public IP.

Get the public IP:

```bash
kubectl get svc -n ingress-nginx
# NAME                          TYPE           EXTERNAL-IP        PORT(S)
# ingress-nginx-controller      LoadBalancer   34.105.123.45      80, 443
```

Point your domain (`example.com`) at `34.105.123.45` in DNS. From now on, all traffic for that domain hits this controller.
### An Ingress resource

```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: mern-ingress
  annotations:
    nginx.ingress.kubernetes.io/rewrite-target: /            # optional
    cert-manager.io/cluster-issuer: letsencrypt-prod         # if using cert-manager for HTTPS
spec:
  ingressClassName: nginx
  tls:
    - hosts:
        - example.com
      secretName: example-com-tls                            # cert lives here
  rules:
    - host: example.com
      http:
        paths:
          - path: /api
            pathType: Prefix
            backend:
              service:
                name: backend
                port:
                  number: 5000
          - path: /
            pathType: Prefix
            backend:
              service:
                name: frontend
                port:
                  number: 80
```

Apply it. The Ingress Controller picks it up within seconds and updates its routing.

Now `https://example.com/api/tasks` → backend Service → backend Pods. `https://example.com/` → frontend Service → frontend Pods. One LoadBalancer, many services.
### `pathType` — exact, prefix, or implementationSpecific

```yaml
- path: /api
  pathType: Prefix              # /api, /api/x, /api/y/z all match
- path: /healthz
  pathType: Exact               # ONLY /healthz matches, not /healthz/sub
- path: /assets
  pathType: ImplementationSpecific   # depends on the controller's interpretation
```

`Prefix` covers most cases.
### Multiple hosts (virtual hosting)

```yaml
spec:
  rules:
    - host: app.example.com
      http:
        paths:
          - path: /
            pathType: Prefix
            backend:
              service: { name: frontend, port: { number: 80 } }
    - host: api.example.com
      http:
        paths:
          - path: /
            pathType: Prefix
            backend:
              service: { name: backend, port: { number: 5000 } }
    - host: status.example.com
      http:
        paths:
          - path: /
            pathType: Prefix
            backend:
              service: { name: uptime-kuma, port: { number: 3001 } }
```

One Ingress, three subdomains, three different Services. Same LoadBalancer.
### Automatic HTTPS — cert-manager

The other production must-have: free auto-renewing TLS certificates from Let's Encrypt, integrated with Ingress.

```bash
# Install cert-manager once per cluster
kubectl apply -f https://github.com/cert-manager/cert-manager/releases/download/v1.14.0/cert-manager.yaml
```

Create a `ClusterIssuer`:

```yaml
apiVersion: cert-manager.io/v1
kind: ClusterIssuer
metadata: { name: letsencrypt-prod }
spec:
  acme:
    server: https://acme-v02.api.letsencrypt.org/directory
    email: you@example.com
    privateKeySecretRef: { name: letsencrypt-prod }
    solvers:
      - http01:
          ingress: { class: nginx }
```

Now any Ingress with `cert-manager.io/cluster-issuer: letsencrypt-prod` annotation triggers cert-manager to:

1. Request a cert from Let's Encrypt.
2. Solve the ACME challenge through the Ingress controller.
3. Store the cert in the Secret named in `tls.secretName`.
4. Auto-renew it before expiry.

This is the K8s equivalent of what Caddy does automatically (Section 4.5), but plumbed through cluster primitives.
### Why we put Ingress + Ingress Controller in front of everything

Without Ingress:

- Each Service of type LoadBalancer = one cloud LB = one bill.
- No path-based routing.
- HTTPS configured per-service.

With Ingress + Ingress Controller:

- One LoadBalancer for the whole cluster.
- All routing rules expressed in YAML.
- HTTPS handled centrally with cert-manager.
- Easy to add new services (just write another Ingress).

> 🔁 **Recap (3.5):** Ingress = HTTP routing rules. Ingress Controller = the proxy that actually executes them. One controller serves many Ingress resources. Combined with cert-manager, you get one public IP, automatic HTTPS, and unlimited path/host-based routing for the whole cluster.

---
## 3.6 — Storage and Persistence in Kubernetes
### What problem are we solving?

When a Pod is deleted, its filesystem is gone — exactly like with raw Docker containers (Section 2.1). For databases (Mongo, Postgres) and any stateful service, this is unacceptable.

In Docker Compose we used named volumes. K8s has its own version, but with an additional layer: K8s runs across many machines, so storage has to _follow the Pod_ across nodes when it's rescheduled.
### The three pieces: PV, PVC, StorageClass

```
   ┌─────────────────────────────────────────────────────────────┐
   │  StorageClass                                               │
   │  "When somebody asks for storage, here's HOW to provision   │
   │   it" (e.g., aws-ebs, gcp-pd, longhorn)                     │
   └─────────────────────────────────────────────────────────────┘
                            │
                            │ provisions
                            ▼
   ┌─────────────────────────────────────────────────────────────┐
   │  PersistentVolume (PV)                                      │
   │  An actual chunk of storage that exists in the cluster.     │
   │  E.g., a 10Gi AWS EBS volume.                               │
   │  Cluster-scoped (not per-namespace).                        │
   └─────────────────────────────────────────────────────────────┘
                            ▲
                            │ bound to
                            │
   ┌─────────────────────────────────────────────────────────────┐
   │  PersistentVolumeClaim (PVC)                                │
   │  "I, this Pod, want a 10Gi volume."                         │
   │  Namespace-scoped. Pods reference PVCs.                     │
   └─────────────────────────────────────────────────────────────┘
                            ▲
                            │ mounted by
                            │
   ┌─────────────────────────────────────────────────────────────┐
   │  Pod                                                        │
   │  Mounts PVC at /data/db                                     │
   └─────────────────────────────────────────────────────────────┘
```

Three layers, each with a clear role:

- **StorageClass** — describes a _type_ of storage and how to provision it. `aws-ebs-gp3`, `gcp-pd-ssd`, `longhorn`. The cluster admin sets this up.
- **PersistentVolume (PV)** — an actual storage resource (a real disk somewhere). May be pre-created (static provisioning) or auto-created from a StorageClass when a PVC requests it (dynamic provisioning).
- **PersistentVolumeClaim (PVC)** — a request from a Pod ("I need 10Gi"). K8s binds it to a matching PV.

Pods don't reference PVs directly. They reference PVCs. The PVC is the contract; the PV is the implementation.
### StorageClass

```yaml
apiVersion: storage.k8s.io/v1
kind: StorageClass
metadata: { name: standard }
provisioner: kubernetes.io/aws-ebs    # the driver that provisions storage
parameters:
  type: gp3
  encrypted: "true"
reclaimPolicy: Retain                  # what to do with the PV when its PVC is deleted
volumeBindingMode: WaitForFirstConsumer
```

Most managed clusters ship a default StorageClass — you may never need to write one yourself. Check:

```bash
kubectl get storageclass
# NAME                 PROVISIONER             RECLAIMPOLICY   VOLUMEBINDINGMODE
# standard (default)   ebs.csi.aws.com         Delete          WaitForFirstConsumer
```
### PersistentVolumeClaim

```yaml
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: mongo-pvc
spec:
  accessModes:
    - ReadWriteOnce              # only one node can mount this volume r/w
  storageClassName: standard
  resources:
    requests:
      storage: 10Gi              # I want 10 GiB
```

Apply it:

```bash
kubectl apply -f mongo-pvc.yaml
kubectl get pvc
# NAME        STATUS    VOLUME                 CAPACITY   ACCESS MODES   STORAGECLASS
# mongo-pvc   Bound     pvc-a3b9c1d4-...       10Gi       RWO            standard
```

`Status: Bound` means K8s has provisioned a real PV (10Gi EBS volume on AWS, in this case) and bound it to your claim.

#### Access modes

|Mode|Meaning|
|---|---|
|`ReadWriteOnce` (RWO)|Volume can be mounted r/w by **one node** at a time. Most common (databases).|
|`ReadOnlyMany` (ROX)|Many nodes can mount it read-only. Useful for shared static assets.|
|`ReadWriteMany` (RWX)|Many nodes can mount it r/w. Requires special storage (NFS, EFS). Rare.|
|`ReadWriteOncePod` (RWOP)|Newer — only one Pod (not node) at a time.|

For a Mongo Pod, `RWO` is fine — only one Pod uses it.
### Mounting the PVC into a Pod

```yaml
apiVersion: apps/v1
kind: Deployment
metadata: { name: mongo }
spec:
  replicas: 1
  selector: { matchLabels: { app: mongo } }
  template:
    metadata: { labels: { app: mongo } }
    spec:
      containers:
        - name: mongo
          image: mongo:6.0
          ports: [{ containerPort: 27017 }]
          volumeMounts:
            - name: mongo-storage
              mountPath: /data/db          # Mongo's data dir
      volumes:
        - name: mongo-storage
          persistentVolumeClaim:
            claimName: mongo-pvc           # ← the PVC we created above
```

Now:

- Pod mounts the 10Gi volume at `/data/db`.
- Mongo writes data to it.
- Pod deleted? Volume survives.
- Pod recreated on another node? K8s detaches volume from old node, attaches to new node, mounts at the same path. Data intact.
### StatefulSet — when you want predictable identity

A `Deployment` is fine for stateless apps with interchangeable pods. For databases that need:

- A stable, predictable name (`mongo-0`, `mongo-1`, `mongo-2`).
- A unique PVC per pod (each pod's own data).
- Ordered startup (pod-0 starts first, then pod-1).

…use a **StatefulSet** instead:

```yaml
apiVersion: apps/v1
kind: StatefulSet
metadata: { name: mongo }
spec:
  serviceName: mongo
  replicas: 3
  selector: { matchLabels: { app: mongo } }
  template:
    metadata: { labels: { app: mongo } }
    spec:
      containers:
        - name: mongo
          image: mongo:6.0
          volumeMounts:
            - name: data
              mountPath: /data/db
  volumeClaimTemplates:                # K8s creates a PVC PER POD automatically
    - metadata: { name: data }
      spec:
        accessModes: [ReadWriteOnce]
        storageClassName: standard
        resources:
          requests:
            storage: 10Gi
```

This produces 3 pods (`mongo-0`, `mongo-1`, `mongo-2`), each with their own 10Gi PVC (`data-mongo-0`, `data-mongo-1`, `data-mongo-2`). When `mongo-1` reschedules to another node, _its_ volume follows it specifically.

Used for: distributed databases (Mongo replica set, Postgres, Cassandra), Kafka, Elasticsearch, Zookeeper.
### A practical recap example

```yaml
# Combined: PVC + Deployment for Mongo with persistent storage
---
apiVersion: v1
kind: PersistentVolumeClaim
metadata: { name: mongo-pvc }
spec:
  accessModes: [ReadWriteOnce]
  storageClassName: standard
  resources:
    requests:
      storage: 10Gi
---
apiVersion: apps/v1
kind: Deployment
metadata: { name: mongo }
spec:
  replicas: 1
  selector: { matchLabels: { app: mongo } }
  template:
    metadata: { labels: { app: mongo } }
    spec:
      containers:
        - name: mongo
          image: mongo:6.0
          ports: [{ containerPort: 27017 }]
          volumeMounts:
            - { name: mongo-data, mountPath: /data/db }
      volumes:
        - name: mongo-data
          persistentVolumeClaim:
            claimName: mongo-pvc
---
apiVersion: v1
kind: Service
metadata: { name: mongo }
spec:
  selector: { app: mongo }
  ports: [{ port: 27017, targetPort: 27017 }]
```

Apply, and you have a Mongo with stable storage that survives Pod restarts and node failures.
### What happens when a node dies

Scenario: the Pod was running on `node-A`. The PVC is bound to a 10Gi EBS volume in AWS. Node-A dies suddenly.

1. Kubelet on node-A stops reporting → control plane marks node NotReady.
2. The Deployment controller sees a missing Pod → creates a new one.
3. Scheduler picks node-B for the new Pod.
4. AWS EBS plugin detaches the volume from node-A, attaches to node-B.
5. Pod starts on node-B with the same data mounted.

Total downtime: typically 1–2 minutes (mostly waiting for the cluster to confirm node-A is dead, plus volume detach/attach time). This is what "stateful workloads on K8s" looks like in practice.
### Common storage gotchas

1. **`ReadWriteOnce` and multi-replica Deployments don't mix.** If you set `replicas: 3` and mount the same PVC in all of them, only one Pod can attach the volume — the others stay `Pending`. Use a StatefulSet with `volumeClaimTemplates`.
2. **Forgetting `storageClassName`** when no default exists. PVC stays `Pending` forever.
3. **`reclaimPolicy: Delete`** on the StorageClass = when the PVC is deleted, the underlying storage is _also_ deleted. Lose the PVC, lose the data. Use `Retain` for production databases.
4. **EBS volumes are zone-bound.** A PVC backed by an EBS volume in `us-east-1a` can never be mounted by a Pod scheduled in `us-east-1b`. The scheduler usually handles this via topology constraints, but if it doesn't, the Pod hangs `Pending`.
5. **No backups built in.** A PVC is just storage; if Mongo corrupts the database, the PVC happily preserves the corruption. Schedule logical backups (mongodump, pg_dump) separately.

> 🔁 **Recap (3.6):** Pods are ephemeral; data persists in **PersistentVolumes**. Pods request storage via **PersistentVolumeClaims**, which the cluster fulfills using **StorageClasses** to dynamically provision PVs. For multi-pod stateful apps with stable identity, use **StatefulSets** with `volumeClaimTemplates`. Volumes follow the Pod across node failures — that's the whole magic of stateful workloads on K8s.

---
---

This concludes **Section 3 — Advanced Orchestration & Networking**. Covered:

- ✅ Container-to-container communication (DNS at 127.0.0.11, multi-network containers, debugging recipe)
- ✅ Nginx (full reverse proxy walkthrough, the Vite ARG vs ENV deep dive resolved, multi-stage builds, common mistakes)
- ✅ Kubernetes Declarative (kubectl reference, Pods, ReplicaSets, Deployments, rolling updates, probes, resource limits, Services and types)
- ✅ ConfigMaps and Secrets (envFrom, file-mount patterns, Sealed Secrets context, dev/prod overlays)
- ✅ Ingress (controller architecture, ingress-nginx install, multi-host routing, automatic HTTPS via cert-manager)
- ✅ Storage and Persistence (StorageClass → PV → PVC chain, access modes, StatefulSets with volumeClaimTemplates, what happens when a node dies)

Next: **Section 4 — Infrastructure, Automation & Monitoring** — Tools Overview & Architecture, Sublyzer, Terraform, Ansible, Caddy, Uptime Kuma, Prometheus + Grafana, k6 load testing.

---
# SECTION 4 — INFRASTRUCTURE, AUTOMATION & MONITORING

We've built up through individual tools — Docker, Compose, Jenkins, Kubernetes. Now we put together the **complete production deployment pipeline**: from "no server exists" to "server provisioned, configured, app deployed, and monitored." Each tool in this section has a single, non-overlapping responsibility — together they form the full DevOps toolchain you'd see at a real company.

The architecture in this section is the one from your raw notes (Part 4) — Hostinger VPS + Terraform + Ansible + Docker Compose + Caddy + Uptime Kuma + Prometheus + Grafana + k6 + Sublyzer. Same shapes apply in AWS/GCP/Azure with substitutions (Hostinger VPS → EC2; Caddy → ALB+ACM; or even all of it on Kubernetes).

---
## 4.1 — Tools Overview and Project Architecture
### The complete production stack

Each tool fills one slot in the pipeline. None of them overlap. Memorize this table — it's the mental map for everything that follows.

|Tool|Role|Real-world analogy|
|---|---|---|
|**Hostinger VPS**|The actual cloud server hosting everything|The plot of land|
|**Terraform**|Provisions infrastructure (the VPS exists)|Building the plot of land|
|**Ansible**|Configures the server and deploys the app|Constructing the building|
|**Docker Compose**|Runs and manages containers|Furnishing the building|
|**Caddy**|Reverse proxy + automatic HTTPS|Front door + reception|
|**Uptime Kuma**|Monitors if endpoints are up or down|Smoke alarm|
|**Prometheus**|Collects and stores numeric metrics over time|Health monitor recording vitals|
|**Grafana**|Visualizes Prometheus data as dashboards|The screen displaying those vitals|
|**Grafana k6**|Load tests to simulate concurrent user traffic|The stress test|
|**Sublyzer**|Application-level observability + error tracking|The doctor analyzing symptoms|

> 💡 **Why so many tools?** Each one is _purpose-built_. Trying to make Docker do what Terraform does (or vice versa) leads to brittle, hand-rolled scripts. The split is a feature, not a bug.
### The full pipeline in order

```
   Terraform → Ansible → Docker Compose → Caddy → (Monitoring Stack)
   ────────   ───────   ──────────────   ─────    ──────────────────
   provision  configure  run app         expose   observe & alert
   the VPS    the VPS    containers      to web

                         Sublyzer wraps the application code (independent layer)
```

Visualized as the diagram from your notes:

```
   ┌──────────────────────────────────────────────────────────────────┐
   │  YOUR LAPTOP / CI RUNNER (control plane for ops)                 │
   │                                                                  │
   │   ┌────────────────────┐                                         │
   │   │  1. Terraform      │  reads .tf files → calls Hostinger API  │
   │   │     terraform.tf   │  → creates a VPS in Mumbai data center  │
   │   └────────────────────┘                                         │
   │              │                                                   │
   │              │ outputs vps_ip = "187.127.157.4"                  │
   │              ▼                                                   │
   │   ┌────────────────────┐                                         │
   │   │  2. Ansible        │  SSH to the new VPS                     │
   │   │     playbooks/*.yml│  → install Docker, Git                  │
   │   └────────────────────┘  → clone repo, copy .env, etc.          │
   │              │                                                   │
   │              ▼                                                   │
   └──────────────┼───────────────────────────────────────────────────┘
                  │ SSH
                  ▼
   ┌──────────────────────────────────────────────────────────────────┐
   │  HOSTINGER VPS (the production server, 187.127.157.4)            │
   │                                                                  │
   │   /opt/statusboard/  ← cloned by Ansible                         │
   │   ├── compose.yaml                                               │
   │   ├── Caddyfile                                                  │
   │   └── server/.env, client/.env.production                        │
   │                                                                  │
   │   ┌────────────────────────────────────────────────────────────┐ │
   │   │  3. Docker Compose                                         │ │
   │   │     docker compose up -d --build  ← started by Ansible     │ │
   │   │                                                            │ │
   │   │  ┌──────────┐  ┌──────────┐  ┌──────────┐  ┌────────────┐  │ │
   │   │  │  caddy   │  │   web    │  │  server  │  │   mongo    │  │ │
   │   │  │  :80/443 │→ │  :5173   │  │  :5000   │→ │  :27017    │  │ │
   │   │  └──────────┘  └──────────┘  └──────────┘  └────────────┘  │ │
   │   │       │            ▲                                       │ │
   │   │       │            │                                       │ │
   │   │       └────routing─┘                                       │ │
   │   └────────────────────────────────────────────────────────────┘ │
   │                                                                  │
   │   ┌────────────────────────────────────────────────────────────┐ │
   │   │  4. Observability stack (separate Compose project)         │ │
   │   │                                                            │ │
   │   │  ┌────────────┐ ┌──────────┐ ┌──────────┐ ┌─────────────┐  │ │
   │   │  │ uptime-kuma│ │prometheus│ │ grafana  │ │  exporters  │  │ │
   │   │  │   :3001    │ │  :9090   │ │  :3000   │ │ node, cAdv, │  │ │
   │   │  │            │ │          │ │          │ │  blackbox   │  │ │
   │   │  └────────────┘ └──────────┘ └──────────┘ └─────────────┘  │ │
   │   └────────────────────────────────────────────────────────────┘ │
   └──────────────────────────────────────────────────────────────────┘
                  ▲                            ▲
                  │ HTTPS                      │ HTTPS
                  │                            │
   ┌──────────────────────────┐  ┌──────────────────────────────────┐
   │ devopscourseankit        │  │ statusdevopscourseankit          │
   │ .duckdns.org             │  │ .duckdns.org                     │
   │ (the app)                │  │ (Uptime Kuma status page)        │
   └──────────────────────────┘  └──────────────────────────────────┘

         ┌──────────────────────────────────────────┐
         │  Sublyzer SaaS dashboard                 │
         │  receives error events from the SDK      │
         │  embedded in the React frontend          │
         └──────────────────────────────────────────┘
```
### How each layer feeds the next

The tools form a **producer-consumer chain**:

|Stage|Producer|What gets produced|Consumer|
|---|---|---|---|
|Provision|Terraform|VPS public IP|Ansible inventory|
|Configure|Ansible|A configured server with Docker installed and repo cloned|Docker Compose (started by Ansible)|
|Run|Docker Compose|Running containers with internal ports|Caddy proxies them externally|
|Expose|Caddy|HTTPS endpoints visible to the world|Browsers, Uptime Kuma, Prometheus blackbox-exporter|
|Observe|Prometheus + Exporters|Time-series metrics|Grafana, Alertmanager|
|Visualize|Grafana|Dashboards|Engineers|
|Validate|k6|Load test results|Capacity planning|
|Application Errors|Sublyzer SDK|Error events with stack traces|Sublyzer dashboard|
### Two distinct Compose projects on one server

A subtle but important detail: the **app stack** and the **observability stack** are two separate `compose.yaml` files (different folders, different networks):

```
/opt/statusboard/         ← app stack (caddy, web, server, mongo)
/opt/observability/       ← observability stack (prometheus, grafana, exporters)
```

Why? They have different lifecycles. You'll redeploy the app many times a day; Prometheus and Grafana stay up for months. Splitting them avoids needless restarts and lets you roll either one independently.

To allow Prometheus to scrape the app (e.g., the Blackbox Exporter probing the public URL), the observability stack reaches the app via the **public hostname** (`https://devopscourseankit.duckdns.org`) — coming in from the outside, just like a real user. No shared internal Docker network needed. This is "blackbox monitoring" by design.

> 🔁 **Recap (4.1):** Each tool is purpose-built. Pipeline order: Terraform → Ansible → Compose → Caddy → Monitoring → Validation. The app stack and the observability stack are independent Compose projects on the same VPS. You can keep this picture in your head from here on; everything below fills in the boxes.

---
## 4.2 — Sublyzer Integration
### What problem are we solving?

Prometheus knows when CPU is high. Uptime Kuma knows when the site is down. Both are _infrastructure-level_ signals — they tell you the _plumbing_ is wrong.

But what about _application-level_ problems?

- A specific React component is throwing an "undefined is not a function" exception for 5% of users.
- An API endpoint returns 200 OK but takes 8 seconds because of an N+1 query.
- A new build broke something subtle for users on Safari only.

These show up as **errors and slow operations from the user's perspective**, _not_ as infrastructure failures. The server is at 30% CPU, the endpoint returns 200, the uptime check is green — but real users are hitting issues. You need an observability tool that lives **inside the application** and reports what _the user actually experiences_.

That's the category Sublyzer (and competitors like Sentry, Bugsnag, Rollbar, LogRocket, Datadog RUM) sit in. Generally called **APM (Application Performance Monitoring)** or **error tracking** or **Real User Monitoring (RUM)**.
### What Sublyzer is

Sublyzer is a SaaS observability tool that gives you a real-time dashboard tracking your application's health, performance, and errors directly from the user's perspective. You drop a small SDK into your frontend, and from then on every JavaScript error, performance metric, and user session is captured and sent to Sublyzer's servers, where it shows up on your dashboard with full stack traces.
### Setup steps

1. **Create an account** on the Sublyzer platform and create a new project — this tells the system "I want to monitor this specific application."
2. **Get the integration key** — after setting up your project, Sublyzer gives you a unique integration code (like a secret password that lets your app talk to Sublyzer's servers).
3. **Connect the app** — save the Sublyzer SDK file into the project's `public/` folder, then paste the integration code into it. This step "activates" the monitoring.
4. **View the dashboard** — once the app is running, the Sublyzer dashboard immediately starts showing live data.

> 💡 **Why drop the SDK into `public/` instead of `npm install`-ing it?** Putting the SDK as a static file in `public/` means it loads via a `<script>` tag _outside_ your bundler. The benefit: errors that happen _before_ your React app finishes loading (e.g., during initial parse) still get captured. With an `npm install`-based SDK that initializes inside `main.jsx`, you'd miss those early-page-load errors.
### What the Overview dashboard shows

The Overview tab shows your key signals at a glance:

| Signal                     | What it tracks                                                                     |
| -------------------------- | ---------------------------------------------------------------------------------- |
| **Weekly Returning Users** | User retention over time                                                           |
| **Bugs**                   | Number of detected issues (linked to the Security tab)                             |
| **Performance**            | Average performance score (100% = fast, responsive)                                |
| **Active Users**           | Who is using the app right now (last 6 hours)                                      |
| **System Health**          | Composite score out of 100, calculated from errors, vulnerabilities, and load time |
| **7-day comparison**       | Trends for Errors, Vulnerabilities, Load time, Sessions                            |

![[Pasted image 20260423003928.png | 900]]

A score of **92/100** with "Worse than yesterday" means something degraded slightly — worth investigating.
### The Security tab — error tracking

The Security tab is where errors are tracked. It shows:

- **Summary** — total bug count and detected exploits broken down by severity (critical / high / medium / low)
- **AI Solutions** — if you connect your GitHub repository, Sublyzer can analyze errors against your actual source code and suggest fixes automatically
- **Error List** — searchable list of every error that occurred, with the full stack trace and severity rating

![[Pasted image 20260423003948.png | 900]]

> 💡 **The GitHub connection is the killer feature.** Without it, the AI can suggest generic JavaScript fixes ("did you check if X is defined?"). With it, Sublyzer reads your actual source code and can say "in `client/src/components/TaskList.jsx:42`, you're calling `.map` on `tasks` before the API response resolves — wrap it in `tasks?.map(...)`." Repository-aware AI suggestions dramatically reduce time-to-fix.

### Where Sublyzer fits in the observability picture

Three layers of observability, each with a different scope:

```
   ┌──────────────────────────────────────────────────────────┐
   │  SUBLYZER (Application layer)                            │
   │  • JavaScript errors in the browser                      │
   │  • Slow API calls from the user's perspective            │
   │  • Per-user, per-session tracking                        │
   └──────────────────────────────────────────────────────────┘
                              ▼ but not
   ┌──────────────────────────────────────────────────────────┐
   │  PROMETHEUS + EXPORTERS (Infrastructure layer)           │
   │  • CPU, RAM, disk, network                               │
   │  • Per-container resource usage                          │
   │  • Endpoint up/down (via Blackbox Exporter)              │
   └──────────────────────────────────────────────────────────┘
                              ▼ but not
   ┌──────────────────────────────────────────────────────────┐
   │  UPTIME KUMA (External availability)                     │
   │  • "Is the public URL responding?"                       │
   │  • From outside the system, like a user                  │
   └──────────────────────────────────────────────────────────┘
```

You need **all three** for a complete picture. They don't replace each other:

- Sublyzer tells you _which user_ hit an error and _what code_ failed.
- Prometheus tells you _which container_ is hot.
- Uptime Kuma tells you _if the site is reachable_.

> 🔁 **Recap (4.2):** Sublyzer = SaaS error & performance tracking inside the application code. Drop the SDK into `public/`, paste the integration key, and every JS error becomes a tracked, stack-traced incident in the dashboard. Connect GitHub for repository-aware AI fix suggestions. It complements (does not replace) Prometheus and Uptime Kuma.

---
## 4.3 — Terraform
### What problem are we solving?

Before deploying any app, _something_ has to create the server. The traditional answer was: log into the cloud console and click buttons.

```
   Click "Compute" → Click "Create VM" → Pick region → Pick size → Pick OS →
   Configure networking → Add firewall rule → ... → Launch.
```

This works once, but creates several problems at scale:

1. **No reproducibility.** Spin up a new dev environment? You have to remember every checkbox you clicked. Six months later you can't.
2. **No version control.** "What changed in our infrastructure last week?" → no record. No diff. No rollback.
3. **No code review.** Infrastructure changes happen by one engineer clicking buttons; nobody reviews them; mistakes go to production unchecked.
4. **No automation.** CI/CD can't click buttons in a console.
5. **Snowflake servers.** Manually configured servers slowly drift apart even when "identical" was the goal.

The fix: write the infrastructure as **code** — config files in Git that _describe_ what should exist. A tool reads them and makes it happen via cloud APIs.

That's **Infrastructure as Code (IaC)**, and Terraform is the dominant tool in the category.
### What Terraform is

Terraform is an IaC tool from HashiCorp. You write `.tf` configuration files describing what infrastructure should exist. Terraform reads those files, compares them with the current state of your cloud, computes a diff, and applies it via the cloud provider's API.

Key properties:

- **Declarative.** You describe the desired state, not the steps.
- **Multi-cloud.** Same workflow for AWS, GCP, Azure, Hostinger, DigitalOcean, Cloudflare, Datadog, GitHub itself — through provider plugins.
- **State-aware.** Terraform tracks what it created in a state file, so it knows what to update vs create vs destroy.
- **Plan before apply.** Always shows you the diff before making changes — you review, then approve.
### Why IaC matters (the benefits, concretely)

- **Reproducibility** — spin up identical environments in dev, staging, and production from the same code.
- **Version control** — infrastructure changes live in Git: reviewable, reversible, auditable.
- **Automation** — CI/CD pipelines can provision and tear down infrastructure automatically.
- **No snowflake servers** — without IaC, every manually configured server ends up slightly different. With Terraform, they are always identical.
### Provisioning vs Configuration Management — the most common interview question

This trips up everyone, so we'll be precise.

|Tool|Type|Question it answers|How it talks to the target|
|---|---|---|---|
|**Terraform**|Provisioning|"What infrastructure should EXIST?" (servers, networks, DNS records, load balancers)|Cloud provider APIs|
|**Ansible**|Configuration Management|"What should be INSTALLED and RUNNING on a server that already exists?" (Docker, Git, app code)|SSH|

> Terraform **builds the plot of land**. Ansible **constructs the building on it**.

Terraform's output (the new VPS's IP address) becomes Ansible's input (the inventory target). The tools chain together — neither replaces the other.

> 💡 In a job interview, the simplest crisp answer is: _"Terraform talks to cloud APIs to create resources. Ansible talks to servers over SSH to configure them."_
### The Terraform CLI workflow

The CLI is the bridge between your local `.tf` files and the cloud provider. It reads your configuration, interacts with provider APIs, and manages the full lifecycle.

```
   .tf config files (desired state)
            │
            ▼
   terraform init       → download provider plugins, prepare directory
   terraform fmt        → auto-format code to standard style
   terraform validate   → check syntax and logic (no API calls made)
   terraform plan       → calculate diff: what will be created / changed / destroyed
   terraform apply      → execute the plan, build real infrastructure
   terraform output     → print outputs (e.g., VPS IP) for downstream tools
```
#### `terraform init`

The mandatory first step. Downloads the required provider plugins and creates the `.terraform.lock.hcl` file. You must run this before any other command. Like `npm install` — it sets up the environment.

```bash
terraform init
# Initializing provider plugins...
# - Finding hostinger/hostinger versions matching "0.1.22"...
# Terraform has been successfully initialized!
```
#### `terraform fmt`

Automatically formats your `.tf` files to follow standard Terraform style — consistent indentation, spacing, and alignment. Run this before every commit.

```bash
terraform fmt
```
#### `terraform validate`

Checks that your configuration files are syntactically correct and logically valid before making any real changes. Does not call any cloud APIs — it only reads your local files.

```bash
terraform validate
# Success! The configuration is valid.
```
#### `terraform plan`

The most important safety step. Generates a "what-if" analysis showing exactly what Terraform would create, modify, or destroy — without actually doing anything. Always read this carefully before applying.

```bash
terraform plan
# Resource actions are indicated by symbols:
#   + = will be created
#   ~ = will be modified in-place
#   - = will be destroyed
#   -/+ = will be destroyed and recreated (forces replacement)
```
#### `terraform apply`

Executes the plan and builds or modifies real infrastructure. Asks for confirmation (`yes`) unless you pass `-auto-approve`.

```bash
terraform apply
# OR in CI/CD:
terraform apply -auto-approve
```
#### `terraform output`

After infrastructure is provisioned, prints the output values you defined (like the VPS IP address). These values are often passed to downstream tools like Ansible.

```bash
terraform output
# vps_ip = "187.127.157.4"
# vps_id = "1618885"
```
### Terraform block types

Infrastructure in Terraform is defined through five main block types:
#### Terraform Block — meta settings

Defines what provider plugins are needed and which versions to lock. This is the project's "rules" file.

```hcl
terraform {
  required_providers {
    hostinger = {
      source  = "hostinger/hostinger"
      version = "0.1.22"
    }
  }
}
```
#### Provider Block — auth and target cloud

Tells Terraform how to authenticate and communicate with the target cloud platform. Each cloud (AWS, Hostinger, GCP) has its own provider.

```hcl
provider "hostinger" {
  api_token = var.hostinger_api_token   # Never hardcode the actual token here
}
```
#### Resource Block — the actual infrastructure

The most important block. Defines the actual infrastructure objects you want Terraform to create and manage — VPS instances, DNS records, virtual networks. This is the core logic.

```hcl
resource "hostinger_vps" "prod" {
  plan           = var.plan
  data_center_id = var.data_center_id
  template_id    = var.template_id
  hostname       = var.hostname
  password       = var.root_password
}
```

The address `hostinger_vps.prod` reads as: _resource of type `hostinger_vps`, named `prod` in our code_. You reference this elsewhere as `hostinger_vps.prod.ipv4_address`.
#### Variable Block — inputs

Defines inputs that can be customized per environment. Keeps sensitive and environment-specific values out of the main logic files.

```hcl
variable "hostname" {
  type        = string
  description = "this is my hostname"
}
```
#### Output Block — return values

Prints important values after deployment and makes them available for downstream tools.

```hcl
output "vps_ip" {
  value = hostinger_vps.prod.ipv4_address
}
```
### Terraform file structure (the production layout)

```
ops/
└── terraform/
    └── prod/
        ├── main.tf                    ← core resource and provider definitions
        ├── variables.tf               ← variable declarations (the "shape")
        ├── outputs.tf                 ← output definitions (the "return values")
        ├── terraform.tfvars           ← actual variable values — NEVER commit to Git
        ├── terraform.tfvars.example   ← documentation template — safe to commit
        ├── .terraform.lock.hcl        ← provider version lock — commit this
        └── terraform.tfstate          ← state file — NEVER commit, store remotely
```
#### `main.tf` — the primary entry point

Contains the core resource definitions. Anything that describes what infrastructure to build lives here. If it grows beyond ~150 lines, split resources into logically named files (`network.tf`, `iam.tf`).

```hcl
# main.tf
terraform {
  required_providers {
    hostinger = {
      source  = "hostinger/hostinger"
      version = "0.1.22"
    }
  }
}

provider "hostinger" {
  api_token = var.hostinger_api_token
}

# Data sources: read-only lookups of existing cloud data
data "hostinger_vps_data_centers" "all" {}
data "hostinger_vps_templates"    "all" {}
data "hostinger_vps_plans"        "all" {}

# Resource: the actual VPS to provision
resource "hostinger_vps" "prod" {
  plan           = var.plan
  data_center_id = var.data_center_id
  template_id    = var.template_id
  hostname       = var.hostname
  password       = var.root_password
}
```

**Best practices:**

- Keep `main.tf` focused on resource definitions only.
- Move variables to `variables.tf` and outputs to `outputs.tf` — don't mix them in `main.tf`.
- Avoid "Terraliths" — don't manage completely unrelated systems in one file (mix one file per logical system: network.tf, vps.tf, dns.tf, etc.).
#### `variables.tf` — input declarations

This file _declares_ variables — it defines their name, type, and optionally a default value or description. It does NOT assign actual values (that happens in `terraform.tfvars`).

```hcl
# variables.tf

# Required variable — no default means Terraform FORCES you to provide it
variable "hostinger_api_token" {
  type        = string
  description = "API token from Hostinger control panel"
  sensitive   = true   # Prevents the value from being printed in CLI output
}

# Optional variable — has a default value as a fallback
variable "hostname" {
  type        = string
  description = "Hostname for the VPS"
  default     = "prod-server-1"
}

variable "plan" {
  type    = string
  default = "hostingerin-vps-kvm1-inr-1m"
}

variable "data_center_id" {
  type    = number
  default = 13                # 13 = Mumbai data center
}

variable "template_id" {
  type    = number
  default = 1200
}

variable "root_password" {
  type      = string
  sensitive = true
}
```

**Supported variable types:**

|Type|Example|
|---|---|
|`string`|`"us-east-1"`|
|`number`|`3`, `5.5`|
|`bool`|`true`, `false`|
|`list`|`["a", "b", "c"]`|
|`map`|`{ env = "prod", region = "us" }`|

**Required vs Optional:**

- Declare without `default` → required (Terraform errors if not provided).
- Add `default = "value"` → optional (falls back to the default if not provided).
- Use `sensitive = true` for passwords and tokens → value is hidden from all CLI output.

> 💡 **Two interview questions you'll get verbatim:**
> 
> - _"How do you write a required input in Terraform?"_ → By declaring a variable without a default value. Terraform forces the user to provide that input at runtime.
> - _"How do you make a variable optional?"_ → By adding a default value to its declaration. The default acts as a fallback if no value is provided.
#### `outputs.tf` — the return values

Outputs serve three purposes:

1. **CLI feedback** — display important information (like VPS IP) in your terminal after `terraform apply`.
2. **Downstream tools** — the VPS IP output becomes the target in Ansible's inventory.
3. **Inter-module communication** — pass data between Terraform modules in complex setups.

```hcl
# outputs.tf
output "vps_ip" {
  description = "The public IPv4 address of the production VPS"
  value       = hostinger_vps.prod.ipv4_address
}

output "vps_id" {
  description = "The Hostinger VPS resource ID"
  value       = hostinger_vps.prod.vps_id
}
```
#### `terraform.tfvars` — actual values (never commit)

Assigns real values to the variables declared in `variables.tf`. Terraform automatically loads this file without you specifying it on the command line.

```hcl
# terraform.tfvars  — ADD TO .gitignore
plan           = "hostingerin-vps-kvm1-inr-1m"
data_center_id = 13
template_id    = 1200
hostname       = "mern-prod-server"
root_password  = "your_secure_password_here"
```

Why not hardcode values directly in `main.tf`?

1. **Environment switching** — have `dev.tfvars`, `staging.tfvars`, `prod.tfvars` and switch with: `terraform apply -var-file="prod.tfvars"`.
2. **Security** — add `*.tfvars` to `.gitignore` so sensitive values never reach GitHub.
#### `terraform.tfvars.example` — documentation template (safe to commit)

A documentation-only file. Terraform does NOT read it — it is purely for humans reading the repository. It shows other developers exactly which values they need to provide.

```hcl
# terraform.tfvars.example
# Copy this file to terraform.tfvars and fill in your actual values.
# DO NOT commit terraform.tfvars — it contains sensitive data.

plan           = "your_plan_slug_here"
data_center_id = 0       # Run: terraform console > data.hostinger_vps_data_centers.all
template_id    = 0       # Run: terraform console > data.hostinger_vps_templates.all
hostname       = "your-server-hostname"
root_password  = "your_secure_root_password"
```
#### `.terraform.lock.hcl` — the dependency lock file

Ensures every team member uses the exact same version of provider plugins. Contains the version number and cryptographic checksums (hashes) to verify the provider code hasn't been tampered with and is identical across operating systems.

> 🔁 **Analogy:** `package-lock.json` in Node.js — it locks the exact dependency tree.

**Rule:** commit this file to Git. It prevents surprise breakage from provider version changes.
### `terraform.tfstate` — the brain (the most critical file)

The most critical file in a Terraform project. It is a JSON file that acts as the single source of truth, mapping your HCL resource names to real-world cloud resource IDs.

**What it stores:**

- **Mapping** — your code says `hostinger_vps.prod` but Hostinger's API sees `1618885`. The state file remembers this link.
- **Metadata** — resource dependencies, generated IPs, resource IDs, and attributes not written in your code but needed for updates.
- **Performance** — Terraform reads state before calling cloud APIs, so it only queries what actually needs updating.

**The state lifecycle:**

```
Before first apply    → state file is empty
After terraform apply → state file records all created resources and their IDs
On next terraform plan → Terraform compares: Code vs State File vs Reality
terraform destroy     → deletes resources, clears them from state
```

**Critical warnings:**

- **Never edit this file manually.** Use `terraform state` commands if you must fix state.
- The file contains sensitive data in plain text — resource IDs, sometimes passwords or generated keys.
- **Never commit it to a public Git repo.**
#### Why local state is bad for a team (interview question)

> _"Why is storing terraform.tfstate on a local laptop bad for a team?"_

Two reasons:

1. **Split brain** — if two engineers run `terraform apply` simultaneously with separate local state files, their states diverge. Each thinks it owns different things, leading to destructive conflicts.
2. **Security** — laptops are not encrypted to the standard required for a file containing cloud credentials and resource IDs.
#### Solution: remote backends

|Solution|What it provides|
|---|---|
|**AWS S3 + DynamoDB**|Store state in an S3 bucket; DynamoDB table provides state locking (prevents simultaneous applies).|
|**Terraform Cloud / HCP**|Managed remote state with built-in locking and access controls.|
|**Azure Blob Storage**|Same idea, on Azure.|
|**GCS (Google Cloud Storage)**|Same idea, on GCP.|

A typical S3 backend:

```hcl
terraform {
  backend "s3" {
    bucket         = "my-terraform-state"
    key            = "prod/terraform.tfstate"
    region         = "us-east-1"
    dynamodb_table = "terraform-state-locks"
    encrypt        = true
  }
}
```

When configured, every `terraform apply` reads/writes state in S3, locks the DynamoDB row while running, and unlocks on exit. Two engineers running `apply` simultaneously? The second one waits for the first to finish.
### Handling sensitive variables — environment variables

Instead of storing the API token in `terraform.tfvars`, set it as an OS environment variable. Terraform automatically reads variables prefixed with `TF_VAR_`.

```bash
# Windows PowerShell
$env:TF_VAR_hostinger_api_token = "your_token_here"

# macOS / Linux
export TF_VAR_hostinger_api_token="your_token_here"
```

Setting it as an environment variable means:

- You don't have to paste it into the terminal on every `terraform plan` run.
- It is never written to disk.
### Practical workflow — provisioning the Hostinger VPS

These are the exact commands run in the course:

```bash
# Current files: main.tf, variables.tf, outputs.tf

# Clean up stale state if switching provider versions
Remove-Item -Recurse -Force .terraform        -ErrorAction SilentlyContinue
Remove-Item -Force .terraform.lock.hcl        -ErrorAction SilentlyContinue

# Initialize — downloads hostinger/hostinger v0.1.22 provider
# Note: v0.1.3 had issues; v0.1.22 is the fix
terraform init -upgrade
# Output: Terraform has been successfully initialized!

terraform fmt
terraform validate
# Output: Success! The configuration is valid.

# Set API token via environment variable
$env:TF_VAR_hostinger_api_token = "your_token"

# Plan
terraform plan

# Apply
terraform apply

# Get outputs
terraform output
# vps_ip = "187.127.157.4"
```

**Exploring available data center IDs:**

```bash
terraform console
> data.hostinger_vps_data_centers.all.data_centers
# Returns:
# {
#   "city"      = "Mumbai"
#   "continent" = "Asia"
#   "id"        = 13
#   "location"  = "in"
#   "name"      = "mum"
# }
```
### Importing an existing VPS (instead of creating a new one)

`terraform import` tells Terraform: _"This resource already exists in the cloud — start managing it in state."_

```bash
# Format: terraform import <resource_address> <cloud_resource_id>
terraform import hostinger_vps.prod 1618885

# Output:
# hostinger_vps.prod: Importing from ID "1618885"...
# hostinger_vps.prod: Import prepared!
# hostinger_vps.prod: Refreshing state... [id=1618885]
# Import successful!
```

After importing, running `terraform plan` shows the diff between your `.tf` file and the real resource's current configuration. In the course, this showed:

```
-/+ resource "hostinger_vps" "prod" {
   ~ data_center_id = 23 -> 13       # forces replacement
   ~ template_id    = 1077 -> 1200   # forces replacement
   + password       = (sensitive value)        # forces replacement
   ~ plan           = "KVM 1" -> "hostingerin-vps-kvm1-inr-1m"
}

Plan: 1 to add, 0 to change, 1 to destroy.
```

The `-/+` symbol means the resource must be destroyed and recreated because the change to `data_center_id` cannot be done in-place.

`terraform.tfstate` is created automatically after the first `terraform apply` or `terraform import`. You will see the file appear in your project directory.
### A complete first run, recap

```bash
mkdir -p ops/terraform/prod && cd ops/terraform/prod

# Write main.tf, variables.tf, outputs.tf, terraform.tfvars (gitignored)

terraform init                    # download hostinger plugin
terraform fmt                     # tidy code
terraform validate                # check syntax
terraform plan                    # preview the diff
terraform apply                   # confirm + create the VPS
terraform output                  # see vps_ip = "187.127.157.4"
```

That single IP is now the input to the next stage — Ansible.

> 🔁 **Recap (4.3 — Terraform):** IaC = describe infrastructure as text files in Git. Terraform's lifecycle is `init → fmt → validate → plan → apply → output`. Five block types: terraform / provider / resource / variable / output. The `terraform.tfstate` file is the brain — never commit, store remotely (S3+DynamoDB) for teams. Sensitive variables go in `TF_VAR_*` env vars. Output values feed Ansible.

---
## 4.4 — Ansible
### What problem are we solving?

Terraform created a server. Now you SSH in. You see a fresh Ubuntu — nothing installed, no Docker, no Git, no app code, no `.env` files. To get the app running you'd run dozens of commands by hand. Then for the _next_ server, you'd run them again. And the next. And then six months later you'd realize the third server has Node 16 because you fat-fingered the install command, and now production is in a weird hybrid state.

A shell script would help — but shell scripts:

- Are hard to make idempotent (re-run safety).
- Are awkward to test.
- Don't have structured output ("this 47th step changed the apt cache").
- Don't have a clean way to template files.
- Can't easily parallelize across many servers.

What we need is a tool **dedicated** to "configure these servers and deploy this app on them" — _idempotent_, _parallel_, _structured_. That's Ansible.
### What Ansible is

Formal definition: **An automation engine for remote machines that is reachable, repeatable, and targeted. It provides consistency, confidence in deployment processes, and auditability.**

Practically: you write **YAML playbooks** describing what state your servers should be in (Docker installed, app cloned to `/opt/app`, container up). Ansible connects to each server over SSH, executes the necessary actions, and reports back what changed.
### Why Ansible instead of manual SSH

Before Ansible:

|Problem with bare SSH|Effect|
|---|---|
|**Not repeatable** — manual, undocumented|Knowledge lost when team members leave|
|**Not scalable** — running 50 setups manually|Impractical|
|**Not idempotent** — `apt install docker` twice|Errors or wastes time|
|**No audit trail** — no record of what was done|Unable to investigate after issues|

Ansible replaces ad-hoc SSH commands with **YAML playbooks** — structured, repeatable, version-controlled automation that works the same way every time.
### Core concepts
#### Agentless architecture

Unlike tools like Chef or Puppet, Ansible requires **no software installed on target machines**. It connects via standard SSH (Linux) or WinRM (Windows), runs its tasks, and then cleans up. This makes setup dramatically simpler.

**Comparison with peers:**

|Tool|Agent on target?|Style|Language|
|---|---|---|---|
|Ansible|No (SSH)|Push|YAML|
|Chef|Yes|Pull|Ruby|
|Puppet|Yes|Pull|Puppet DSL|
|SaltStack|Yes (or agentless)|Push or Pull|YAML/Python|

> ⚙️ **Added Professional Context:** Most modern teams pick Ansible because of the agentless model. No agent to install, no daemon to keep alive, no version mismatch to debug. You SSH-keypair into the targets and you're done.
#### Control Node vs. Managed Nodes

- **Control Node** — the machine where Ansible is installed and from which you run commands. Your laptop (WSL) or a CI/CD runner.
- **Managed Nodes** — the remote servers being automated. In this course, the Hostinger VPS provisioned by Terraform.
#### Inventory

A file listing all the managed nodes Ansible knows about, organized into groups. Instead of hardcoding IPs into every script, you define them centrally. Groups (like `prod`, `stage`, `web`, `db`) let you target specific environments precisely — run a deployment only against `prod` without touching `stage`.

```ini
# inventory/hosts
[prod]
187.127.157.4 ansible_user=root ansible_ssh_private_key_file=~/.ssh/id_ed25519

[stage]
192.168.1.100 ansible_user=ubuntu
```
#### Playbooks

Human-readable YAML files that define the desired state of a system. A sequence of tasks to execute on target nodes. They serve as both **automation** and **documentation**.

```yaml
---
- name: Bootstrap production server
  hosts: prod                    # Target "prod" group from inventory
  become: true                   # Run tasks as root
  tasks:
    - name: Update package cache
      apt:
        update_cache: yes

    - name: Install Docker
      apt:
        name: docker.io
        state: present           # "present" = install if not already there

    - name: Start Docker service
      service:
        name: docker
        state: started
        enabled: yes
```
#### Modules — Ansible's vocabulary

The tools in Ansible's toolbox — small programs that perform specific tasks on managed nodes. Each module is designed to be **idempotent**: it only makes a change if the system is not already in the desired state.

|Module|What it does|
|---|---|
|`apt` / `yum`|Install or remove packages|
|`copy`|Copy files from control node to managed node|
|`template`|Copy a Jinja2 template file with variable substitution|
|`service`|Start, stop, restart, or enable system services|
|`shell` / `command`|Run arbitrary shell commands|
|`get_url`|Download files from a URL|
|`file`|Create directories, set permissions, manage symlinks|
|`git`|Clone or update a Git repository|
|`lineinfile`|Add/modify a single line in a file (idempotently)|
|`cron`|Manage cron jobs|
|`user`|Create users and groups|
|`docker_compose_v2`|Run Docker Compose commands|
#### Idempotency — the killer feature

Running the same Ansible playbook 10 times produces the same result as running it once. If Docker is already installed, the `apt` module returns `ok` and skips installation. This is what makes Ansible safe to run repeatedly — in CI/CD pipelines, on every deploy, or whenever you want to ensure a server's state is correct.

> 💡 With shell scripts you fear the second run. With Ansible you _welcome_ it — running daily is a great way to detect drift.
#### `become: true` — privilege escalation

Many tasks (installing packages, modifying system files, creating users) require root permissions. `become: true` handles this automatically — it is the equivalent of prefixing every shell command with `sudo`.

```yaml
become: true                # Tells Ansible to escalate privileges
become_method: sudo         # How to escalate (sudo is the default)
become_user: root           # The user to become (root is the default)
```
#### Ansible output — task status

After every task, Ansible reports one of three statuses:

|Status|Color|Action Taken?|Meaning|
|---|---|---|---|
|`ok`|Green|No|Task succeeded; system was already in the desired state|
|`changed`|Yellow|Yes|Task succeeded and a modification was made|
|`failed`|Red|No / Partial|An error occurred; Ansible stops further tasks on this host|

> 💡 **Why `ok` is the goal:** Seeing all `ok` on a re-run means every server is already in the desired state — idempotency working perfectly.

**Practical tips:**

- `ignore_errors: true` — lets the playbook continue even if a specific task fails.
- Handlers only fire on `changed` — a "restart nginx" handler won't trigger if the config file didn't change.
### Project directory structure

```
ops/
└── ansible/
    ├── ansible.cfg                      # project-specific config — avoids passing flags every time
    ├── inventory/
    │   ├── hosts                        # list of servers and groups
    │   ├── group_vars/
    │   │   └── all.yml                  # variables applied to ALL groups
    │   └── host_vars/
    │       └── 187.127.157.4.yml        # variables for a specific host
    ├── playbooks/
    │   ├── bootstrap.yml                # makes raw server ready
    │   ├── deploy.yml                   # deploys the application
    │   └── observability.yml            # deploys the monitoring stack
    └── roles/                           # reusable, modular logic
        ├── common/
        │   ├── tasks/                   # main task files
        │   ├── handlers/                # service restarts (trigger on "changed")
        │   ├── templates/               # Jinja2 templates (.j2)
        │   ├── files/                   # static files to copy
        │   └── vars/                    # role-specific variables
        └── webserver/
```

**Why this structure?**

- `ansible.cfg` — stores the inventory path, SSH key, and remote user so you don't pass `--inventory` and `--user` flags on every command.
- Separating `group_vars/` and `host_vars/` from `playbooks/` — configuration data is decoupled from execution logic, keeping playbooks clean and DRY (Don't Repeat Yourself).
- `roles/` — reusable modules of automation that can be shared across different playbooks.
### `ansible.cfg`

```ini
[defaults]
inventory          = ./inventory/hosts
remote_user        = root
private_key_file   = ~/.ssh/id_ed25519
host_key_checking  = False
stdout_callback    = yaml
```

Once present, `ansible-playbook bootstrap.yml` works without any flags — it picks up the config from the file.
### Setting up SSH keys and running playbooks

```bash
# Install Ansible on WSL
sudo apt update
sudo apt install -y ansible

# Navigate to project root (Windows drives are accessible under /mnt/ in WSL)
cd /mnt/d/Coding/devops/"Final Part - 2026"

# Generate a dedicated SSH key for Ansible
# -t ed25519: modern elliptic curve algorithm (faster and more secure than RSA)
# -a 64:     64 rounds of key derivation (makes brute-force much harder)
# -C:        a label/comment to identify the key
ssh-keygen -t ed25519 -a 64 -f ~/.ssh/id_ed25519 -C "devops-course-ansible"

# Copy the public key to the VPS to authorize passwordless login
ssh-copy-id -i ~/.ssh/id_ed25519.pub root@187.127.157.4

# Verify SSH connection manually
ssh -i ~/.ssh/id_ed25519 root@187.127.157.4

# Point Ansible to your project's config file
export ANSIBLE_CONFIG="$PWD/ansible.cfg"

# Verify Ansible version and config
ansible --version | sed -n '1,6p'

# Test connectivity (ping all hosts in "prod" group)
ansible prod -m ping

# Run the bootstrap playbook (install Docker, Git, etc.)
ansible-playbook bootstrap.yml

# Run the deploy playbook
ansible-playbook deploy.yml

# Run the observability playbook (deploy Prometheus, Grafana, etc.)
ansible-playbook observability.yml
```

**Note on WSL paths:** In WSL, your Windows `D:\` drive is accessible as `/mnt/d/`. So `D:\Coding\devops\...` becomes `/mnt/d/Coding/devops/...`.
### The Ansible deployment flow

```
   Your Laptop (Control Node — runs Ansible)
            ↓ SSH
   Inventory → select "prod" group (187.127.157.4)
            ↓
   1. bootstrap.yml → install Docker, Docker Compose, Git
                    → create app directory (/opt/statusboard)
            ↓
   2. deploy.yml    → git clone / git pull latest code
                    → copy .env files
                    → docker compose up -d --build
            ↓
   3. verify        → curl http://VPS_IP/api/health
            ↓
   VPS (Managed Node) — app is running
```
### A bootstrap playbook (full example)

This playbook makes a raw VPS ready to run our app:

```yaml
# playbooks/bootstrap.yml
---
- name: Bootstrap server — install runtime dependencies
  hosts: prod
  become: true

  tasks:
    - name: Update apt cache
      apt:
        update_cache: yes
        cache_valid_time: 3600

    - name: Install base packages
      apt:
        name:
          - ca-certificates
          - curl
          - gnupg
          - git
          - ufw
        state: present

    - name: Add Docker official GPG key
      get_url:
        url: https://download.docker.com/linux/ubuntu/gpg
        dest: /etc/apt/keyrings/docker.asc
        mode: '0644'

    - name: Add Docker apt repo
      apt_repository:
        repo: "deb [arch=amd64 signed-by=/etc/apt/keyrings/docker.asc] https://download.docker.com/linux/ubuntu {{ ansible_distribution_release }} stable"
        state: present

    - name: Install Docker Engine + Compose plugin
      apt:
        name:
          - docker-ce
          - docker-ce-cli
          - containerd.io
          - docker-compose-plugin
        state: present
        update_cache: yes

    - name: Ensure docker is started and enabled
      service:
        name: docker
        state: started
        enabled: yes

    - name: Create application directory
      file:
        path: /opt/statusboard
        state: directory
        mode: '0755'

    - name: Configure firewall — allow SSH, HTTP, HTTPS
      ufw:
        rule: allow
        port: "{{ item }}"
        proto: tcp
      loop:
        - 22
        - 80
        - 443

    - name: Enable UFW
      ufw:
        state: enabled
        policy: deny             # default-deny everything else
```

Run once. It's fully idempotent — re-running on a configured server produces all `ok` and zero `changed`.
### A deploy playbook

```yaml
# playbooks/deploy.yml
---
- name: Deploy MERN application
  hosts: prod
  become: true

  vars:
    app_dir: /opt/statusboard
    repo_url: https://github.com/ankitsangwan/statusboard.git
    git_branch: main

  tasks:
    - name: Clone or update repo
      git:
        repo: "{{ repo_url }}"
        dest: "{{ app_dir }}"
        version: "{{ git_branch }}"
        force: yes               # discard local changes if any

    - name: Render server .env from template
      template:
        src: ../templates/server.env.j2
        dest: "{{ app_dir }}/server/.env"
        mode: '0600'             # only root can read (it has secrets)

    - name: Render Caddyfile from template
      template:
        src: ../templates/Caddyfile.j2
        dest: "{{ app_dir }}/Caddyfile"
        mode: '0644'

    - name: Ensure k6 directory exists on VPS
      file:
        path: "{{ app_dir }}/ops/tests/k6"
        state: directory

    - name: Sync k6 tests to VPS
      copy:
        src: ../../tests/k6/
        dest: "{{ app_dir }}/ops/tests/k6/"

    - name: Bring up the app stack
      shell: |
        cd {{ app_dir }} && docker compose up -d --build
      args:
        chdir: "{{ app_dir }}"

    - name: Verify health endpoint
      uri:
        url: "http://localhost/api/health"
        status_code: 200
      retries: 5
      delay: 6
```
### A Jinja2 template — `server.env.j2`

Templates let you generate config files with variables substituted at deploy time. This is how you keep secrets out of the repo while still having a "single source of truth" for the deploy process.
- Jinja2 is a **templating engine for Python**. It lets you write a file with **placeholders (variables)** instead of hardcoded values, and then at runtime those placeholders get replaced with actual values.

```
# rendered to {{ app_dir }}/server/.env
PORT=5000
MONGO_URI={{ mongo_uri }}
NODE_ENV=production
JWT_SECRET={{ jwt_secret }}
```

Variables come from `inventory/group_vars/prod.yml` (or `host_vars/`):

```yaml
# inventory/group_vars/prod.yml
mongo_uri: "mongodb://mongo:27017/prod_db"
jwt_secret: "{{ vault_jwt_secret }}"   # encrypted via ansible-vault
```

For real secrets, use `ansible-vault encrypt_string '...'` to store them encrypted in your repo. Decryption happens at playbook run time.
### Observability playbook (preview — Section 4.7 has the full Prometheus side)

```yaml
# playbooks/observability.yml
---
- name: Deploy observability stack
  hosts: prod
  become: true

  tasks:
    - name: Create observability directory on VPS
      file:
        path: /opt/observability
        state: directory
        mode: '0755'

    - name: Copy docker-compose file
      copy:
        src: ../observability/compose.yaml
        dest: /opt/observability/compose.yaml

    - name: Copy prometheus config
      copy:
        src: ../observability/prometheus.yml
        dest: /opt/observability/prometheus.yml

    - name: Copy blackbox config
      copy:
        src: ../observability/blackbox.yml
        dest: /opt/observability/blackbox.yml

    - name: Start observability stack
      shell: |
        cd /opt/observability
        docker compose up -d
```

Run with: `ansible-playbook observability.yml`. Then access:

```
http://187.127.157.4:9090/targets    # Prometheus
http://187.127.157.4:3000/           # Grafana
```
### Common Ansible pitfalls

1. **Forgetting `become: true`** — task fails with "Permission denied". Most system-level tasks need root.
2. **Using `shell` for everything** — kills idempotency. Always prefer a proper module (`apt`, `service`, `copy`) when one exists.
3. **Mixing tabs and spaces in YAML** — YAML is whitespace-sensitive. Use spaces only.
4. **Hardcoding IPs in playbooks** — defeats the inventory pattern. Always reference groups.
5. **Re-running and breaking** — always assume idempotency, but it requires writing tasks correctly. The `shell` module is non-idempotent by default; use `creates:` or `removes:` arguments to gate it.
6. **Hardcoded secrets in playbooks** — use `ansible-vault` to encrypt sensitive YAML files.
### A common one-liner

```bash
# Restart the app stack remotely without SSH-ing in manually
ansible prod -b -m shell -a "cd /opt/statusboard && docker compose -f compose.yaml down && docker compose -f compose.yaml up -d --build"
#         │  │   │           │
#         │  │   │           └── the shell command to run
#         │  │   └────────────── module: shell
#         │  └────────────────── -b = become root (sudo)
#         └───────────────────── target group (or host)
```

> 🔁 **Recap (4.4 — Ansible):** Agentless config-management tool. SSH + YAML playbooks + idempotent modules. Project layout: `ansible.cfg + inventory + playbooks + roles`. Three playbooks anchor the course: `bootstrap.yml` makes a raw server ready, `deploy.yml` ships the app, `observability.yml` ships the monitoring stack. Templates with Jinja2 generate config files. `become: true` for privilege escalation. The Terraform-output IP becomes Ansible's inventory target — that's the chain.

---
## 4.5 — Caddy
### What problem are we solving?

Our app runs on the VPS in containers — backend on `:5000`, frontend on `:5173`, Uptime Kuma on `:3001`. To put this in front of real users we need:

1. **One single entry point** (port 80 / 443) so URLs are clean.
2. **HTTPS** with valid SSL certificates so browsers don't show the scary "Not Secure" warning.
3. **Path/host routing** to send `/api` to the backend and `/` to the frontend.
4. **Auto-renewal** of certs (Let's Encrypt certs expire every 90 days).

Section 3.2 covered Nginx for routing — but Nginx requires Certbot + cron + manual config to handle HTTPS, and renewal failures are a classic 3 AM page. We want all of that automated, with zero config.
### What Caddy is

Caddy is a modern web server and reverse proxy that sits at the edge of your infrastructure — it's the first thing public internet traffic hits. Its defining feature is **automatic HTTPS**: you give it a domain name and it requests, installs, and renews SSL certificates from Let's Encrypt **entirely on its own**, with no manual steps.

Compared to the Nginx + Certbot approach we walked through in 3.2:

|Need|Nginx + Certbot|Caddy|
|---|---|---|
|Get a cert|Run `certbot --nginx -d ...`|List the domain in Caddyfile|
|Renew before expiry|Cron job (which sometimes fails)|Caddy renews silently in background|
|Add a new domain|Edit nginx.conf + re-run certbot|Add a domain block, save, reload|
|Config file size|~50 lines|~10 lines|
|Failure mode at renewal|"Cert expired, site offline at 3 AM"|Almost never happens|

Caddy's selling point is the friction it removes. For demos, side-projects, and small/medium production deployments, it's a clear win.

> 💡 **Why didn't we use Caddy from the start?** Nginx is the industry standard — knowing it is unavoidable. Caddy is what you'd often _prefer_ for new projects, but Nginx is what you'll meet in 80% of existing codebases. Both belong in your toolbelt.
### Caddy's role in our stack

```
   User Browser
        ↓
   DuckDNS (DNS) → resolves devopscourseankit.duckdns.org → VPS IP (187.127.157.4)
        ↓
   VPS port 80 / 443
        ↓
   Caddy Container (reverse proxy)
        │
        ├── /            → web container         (port 5173)
        ├── /api*        → server container      (port 5000)
        └── status sub   → uptime-kuma container (port 3001)
```

> 💡 **What DuckDNS is:** A free dynamic DNS service that gives you a subdomain like `yourname.duckdns.org` and lets you point it to any IP address. Used here to get a real domain for HTTPS without buying one.

Two domains were set up in the course:

- `devopscourseankit.duckdns.org` → main app
- `statusdevopscourseankit.duckdns.org` → Uptime Kuma status page

Both point to the same VPS IP via DuckDNS (set in the DuckDNS dashboard).
### How Caddy gets SSL certificates (the automatic ACME flow)

ACME is the protocol Let's Encrypt uses for issuing certificates. Caddy speaks ACME natively. Here's the flow, the first time you start Caddy:

```
   1. Caddy starts and reads the Caddyfile — sees devopscourseankit.duckdns.org
   2. Checks its cert store — no certificate exists yet
   3. Contacts Let's Encrypt: "I need a certificate for this domain"
   4. Let's Encrypt sends a challenge: "Prove you control that domain"
   5. Caddy hosts a temporary verification file on port 80
   6. Let's Encrypt fetches the file — domain ownership confirmed
   7. Let's Encrypt issues the certificate — Caddy installs it automatically
   8. All traffic served over HTTPS immediately
   9. ~30 days before expiry, Caddy silently renews in the background
```

This entire dance happens transparently. You never see it.

![[Pasted image 20260501072558.png | 900]]
### The Caddyfile

Caddy's config file (`Caddyfile`) is far simpler than nginx.conf:

```caddyfile
devopscourseankit.duckdns.org {
    handle /api* {
        reverse_proxy server:5000
    }
    handle {
        reverse_proxy web:5173
    }
}

statusdevopscourseankit.duckdns.org {
    reverse_proxy uptime-kuma:3001
}
```

That's it. Two domains, three routes, automatic HTTPS. Compare this with the equivalent ~80-line nginx.conf + Certbot setup.

**Reading the syntax:**

- Each top-level block starts with a **domain name** — that's the host this rule applies to.
- The domain alone tells Caddy: "obtain a Let's Encrypt cert for this and serve HTTPS." No flags, no annotations.
- `reverse_proxy <hostname>:<port>` forwards traffic to that container.
- `handle /api*` matches any path starting with `/api`. The first matching `handle` wins.
- An empty `handle` (or just a `reverse_proxy` directly) is the default — anything not matched by other handlers goes here.

Container names (`server`, `web`, `uptime-kuma`) work as hostnames because all services are on the same Docker network — exactly the DNS magic from Sections 2.2 and 3.1.
### Docker Compose integration

```yaml
services:
  caddy:
    image: caddy:latest
    container_name: caddy
    restart: unless-stopped
    ports:
      - "80:80"           # HTTP (also used for Let's Encrypt ACME challenge)
      - "443:443"         # HTTPS
    volumes:
      - ./Caddyfile:/etc/caddy/Caddyfile:ro    # mount config (read-only)
      - caddy_data:/data                       # persist TLS certificates
      - caddy_config:/config                   # persist config cache
    networks:
      - app-net

volumes:
  caddy_data:               # CRITICAL: stores SSL certs — must persist across restarts
  caddy_config:
```
#### Why persist `caddy_data`?

This volume stores the SSL certificates Caddy obtains from Let's Encrypt. If you lose this volume, Caddy re-requests certificates on the next start. Let's Encrypt has **rate limits** — 5 certificates per domain per week — so losing certs frequently can lock you out for days.

Treat `caddy_data` like a database volume — never delete it casually.

> ⚠️ The single most painful Caddy/Let's Encrypt mistake: running `docker compose down -v` (which removes volumes) → losing `caddy_data` → next restart hits the rate limit → site stuck on HTTP for a week. Always reach for `docker compose down` (no `-v`) unless you specifically intend to wipe volumes.
### Deployment via Ansible

Run the deploy playbook (from Section 4.4) after updating the `Caddyfile` and `compose.yaml`:

```bash
export ANSIBLE_CONFIG="$PWD/ansible.cfg"
ansible-playbook deploy.yml
```
### Troubleshooting Caddy TLS

If the HTTPS domain isn't working after deployment, walk through this checklist:
#### Step 1 — confirm DNS

```bash
dig +short devopscourseankit.duckdns.org
# Should return your VPS IP. If empty or wrong, fix DNS first.
```
#### Step 2 — confirm port 80 is reachable

ACME HTTP-01 challenge requires Let's Encrypt to reach your server on port 80. If your firewall blocks port 80, certs can't be issued.

```bash
sudo ufw allow 80
sudo ufw allow 443
```
#### Step 3 — restart Caddy and watch logs

```bash
ssh root@<VPS_IP>
cd /opt/statusboard
docker compose restart caddy
docker compose logs -f caddy
# Look for messages like "obtaining certificate" and "certificate obtained successfully"
```
#### Step 4 — if Caddy is stuck (failed ACME attempts are cached)

Sometimes Caddy caches failed cert attempts and refuses to retry for a while:

```bash
docker compose down
docker volume rm statusboard_caddy_data    # Clears old/failed cert cache
docker compose up -d
# Caddy requests fresh certificates on startup
```

> ⚠️ Use this only if you're sure the original cert was bad — wiping `caddy_data` on a working Caddy is exactly the rate-limit-trap mistake described above.
#### Step 5 — remotely via Ansible (without SSH-ing in manually)

```bash
ansible prod -b -m shell -a "cd /opt/statusboard && docker compose -f compose.yaml down && docker compose -f compose.yaml up -d --build"
```
### What you'd see on success

After a fresh Caddy start, the logs include lines like:

```
{"level":"info","ts":...,"logger":"tls","msg":"finished provisioning"}
{"level":"info","ts":...,"logger":"tls.obtain","msg":"obtaining certificate","identifier":"devopscourseankit.duckdns.org"}
{"level":"info","ts":...,"msg":"certificate obtained successfully"}
{"level":"info","ts":...,"msg":"served key authentication","identifier":"devopscourseankit.duckdns.org"}
```

And `https://devopscourseankit.duckdns.org/` loads with a green padlock in the browser. Done.
### Where Caddy fits in the bigger picture

Caddy is the only component on the public-facing edge:

- **Mongo** is internal-only (no published port).
- **Backend (server)** is internal-only.
- **Frontend (web)** is internal-only.
- **Uptime Kuma** is internal-only.
- **Caddy** is the only container with `ports: 80, 443`. Everything else routes through it.

That single architectural choice makes the entire stack defensible — only port 443 needs to be hardened, and TLS termination is centralized.

> 🔁 **Recap (4.5 — Caddy):** Caddy = automatic HTTPS reverse proxy. Caddyfile = ~10 lines for full path-and-host routing. The `caddy_data` volume holds your certs — never wipe it casually. ACME flow happens automatically on first start and at renewal. Caddy is the only public-facing service in the stack; everything else is internal.

---
## 4.6 — Monitoring and Uptime Kuma
### The three concerns of production monitoring

Production monitoring has three distinct concerns — each tool addresses a different question:

|Tool|Question it answers|
|---|---|
|**Uptime Kuma**|"Is the service reachable right now? Up or down?"|
|**Prometheus + Exporters**|"What are the system's numeric metrics? CPU, RAM, container stats?"|
|**Grafana**|"What do those metrics look like visually over time?"|
|**Sublyzer**|"What errors are real users experiencing inside the app?"|
### The 3 pillars of observability (industry-standard model)

Every modern observability discussion rests on this triad:

|Pillar|What it captures|Typical tools|
|---|---|---|
|**Metrics**|Numeric values over time (CPU%, request count, error rate)|Prometheus, Datadog, CloudWatch|
|**Logs**|Detailed text records of events|Loki, ELK / EFK Stack, Splunk, Datadog Logs|
|**Traces**|Path of a single request through distributed services|Jaeger, Tempo, OpenTelemetry, Datadog APM|

Plus a fourth, cross-cutting concern:

| **Alerting** | "Wake somebody up when something is wrong" | Alertmanager, PagerDuty, Opsgenie, Slack notifications |

This course focuses on the **Metrics pillar** (Prometheus + Grafana). Logs and Traces are covered as added context in the Bonus section.
### What Uptime Kuma is

**Uptime Kuma** is a self-hosted monitoring tool that repeatedly sends HTTP/HTTPS requests to your endpoints at regular intervals to check if they are up or down. It's the _first line of defense_ — it tells you immediately when something becomes unreachable.

The key distinction from Prometheus: Uptime Kuma is **synthetic monitoring** — it only knows if an endpoint returns a successful response **from the outside**. It cannot tell you _why_ it failed (was it CPU? RAM? A crashed container?). For the "why," you need Prometheus.
### What Uptime Kuma tracks

|Signal|Detail|
|---|---|
|**Up/Down status**|Real-time endpoint availability|
|**Response time**|Latency trends over time|
|**Incident history**|Timestamped record of every outage|
|**Certificate expiry**|Days until your SSL cert expires|
|**Alerting**|Push notifications via Slack, Email, Discord, Telegram|
### Uptime Kuma in Docker Compose

```yaml
services:
  uptime-kuma:
    image: louislam/uptime-kuma:latest
    container_name: uptime-kuma
    restart: unless-stopped
    volumes:
      - uptime-kuma-data:/app/data       # persist monitor configs and history
    networks:
      - app-net

volumes:
  uptime-kuma-data:
```

The status subdomain routes through Caddy (from 4.5):

```
statusdevopscourseankit.duckdns.org → Caddy → uptime-kuma:3001
```

Notice Uptime Kuma has no `ports:` block — only Caddy is exposed publicly. Same defensive pattern as everything else.
### Using Uptime Kuma

1. Navigate to `https://statusdevopscourseankit.duckdns.org`.
2. Create an admin account with a strong password.
3. Add monitors: **New Monitor → HTTP(s)** → enter endpoint URL → set check interval.
4. Add monitors for:
    - `https://devopscourseankit.duckdns.org` (frontend)
    - `https://devopscourseankit.duckdns.org/api/health` (backend API)

> 💡 **Healthcheck endpoint pattern:** Always expose a dedicated `/api/health` (or `/healthz`) endpoint that returns `200 OK` if the app is alive. A successful HTTP request to `/` doesn't prove your DB is reachable; a thoughtful health check does. In Express:
> 
> ```javascript
> app.get('/api/health', async (req, res) => {
>   try {
>     await mongoose.connection.db.admin().ping();
>     res.json({ status: 'ok' });
>   } catch (err) {
>     res.status(503).json({ status: 'unhealthy', error: err.message });
>   }
> });
> ```
### Uptime Kuma dashboard — healthy state

The dashboard shows the `web` monitor at 100% uptime. Key fields visible:

- **Response (Current):** 12 ms — extremely fast.
- **Avg. Response (24-hour):** 207 ms.
- **Uptime (24-hour):** 100%.
- **Uptime (30-day):** 100%.
- **Cert Exp.:** 90 days — the SSL certificate is valid for another 90 days.
- The response time graph shows a spike when the service first started (the container was initializing), then the latency dropped and stabilized.

![[Pasted image 20260427235918.png | 900]]

### Uptime Kuma dashboard — with a failure

A real incident looks like:

- **`server` monitor:** 98.77% uptime (not 100% — there was a failure).
- **`web` monitor:** 100% uptime (frontend stayed up).
- The response time graph shows a red bar at 22:33 — the exact moment of failure.
- The incident log at the bottom shows:
    
    ```
    2026-02-28 22:34:56 — Up    (200 OK)2026-02-28 22:33:56 — Down  (Request failed with status code 502)2026-02-28 22:31:56 — Up    (200 OK)
    ```
    

A **502 Bad Gateway** from the server endpoint means Caddy received the request but the backend container was not responding — typically a container crash or restart.

![[Pasted image 20260428005121.png | 900]]

### Homework

Stop and restart a container while watching Uptime Kuma:

```bash
docker compose stop server
# Wait 60 seconds — Uptime Kuma marks the endpoint Down
docker compose start server
# Uptime Kuma detects recovery on the next probe
```

Observe how it detects the outage, marks the exact timestamp as "Down", then recovers automatically when the container restarts.

> 🔁 **Recap (4.6 — Uptime Kuma):** Synthetic external monitoring of HTTP(S) endpoints. Tells you _that_ something is down, not _why_. Pair with Prometheus for the why. Persist the data volume to keep history. Always expose dedicated `/health` endpoints to monitor.

---
## 4.7 — Prometheus and Grafana
### Why Prometheus and Grafana alongside Uptime Kuma?

Uptime Kuma is synthetic — it only checks if an endpoint returns a successful response from the outside. It has no access to internal system data.

Prometheus and Grafana are needed for:

- **Numeric observability** — CPU saturation, RAM growth, memory usage, container-level CPU throttling.
- **Latency tracking** — identifying performance bottlenecks and slow endpoints.
- **Trend analysis** — historical data to identify patterns like peak traffic times or resource spikes.
### The roles split clearly

```
   ┌─────────────────────────────────────────────────────────┐
   │  PROMETHEUS                                             │
   │  • Time-series database                                 │
   │  • Pull model: scrapes targets every 15s                │
   │  • Stores numbers with timestamps and labels            │
   │  • Has its own query language (PromQL)                  │
   │  • Has its own basic UI for ad-hoc queries              │
   └─────────────────────────────────────────────────────────┘
                            ↓ data
   ┌─────────────────────────────────────────────────────────┐
   │  GRAFANA                                                │
   │  • Pure visualization layer (NO storage)                │
   │  • Connects to Prometheus, runs PromQL                  │
   │  • Renders graphs, charts, dashboards                   │
   │  • Sets up alerts (or hands off to Alertmanager)        │
   └─────────────────────────────────────────────────────────┘
```

> 💡 **One critical fact to memorize:** Grafana does **not** store metrics. It is a display. It reads from Prometheus every time you load a dashboard. If you delete the Prometheus volume, Grafana shows blank dashboards even though Grafana's own volume is intact.
### The exporter / adapter concept

Prometheus speaks one protocol: HTTP GET to a `/metrics` endpoint that returns text in a specific format. But MongoDB doesn't expose `/metrics`. Linux doesn't expose `/metrics`. Docker doesn't either. So we use **exporters**: small adapter programs that translate something into the Prometheus format.

|Exporter|Monitors|Port|Category|
|---|---|---|---|
|**Node Exporter**|Host: CPU, RAM, disk I/O, network|9100|White Box (internal)|
|**cAdvisor**|Per-container: CPU, memory, network per container|8080|White Box (internal)|
|**Blackbox Exporter**|External HTTP/HTTPS probes, DNS, TCP|9115|Black Box (external)|
#### Node Exporter vs cAdvisor — why you need both

- **Node Exporter** tells you "_the host CPU is at 90%_" — the big picture.
- **cAdvisor** tells you "_the backend container is consuming 85% of CPU_" — the drill-down.

You need both: Node Exporter shows there is a problem; cAdvisor shows which container is causing it.
### White Box vs Black Box monitoring

```
   ┌─────────────────────────────────────────────────────┐
   │  WHITE BOX MONITORING                               │
   │  Observe internal metrics from inside the system    │
   │                                                     │
   │  Tools: Node Exporter, cAdvisor                     │
   │  Catches: CPU spikes, memory leaks, container       │
   │           crashes, disk filling up                  │
   │  Misses:  DNS failures, network routing, reverse    │
   │           proxy issues                              │
   └─────────────────────────────────────────────────────┘

   ┌─────────────────────────────────────────────────────┐
   │  BLACK BOX MONITORING                               │
   │  Probe the system from the outside, like a user     │
   │                                                     │
   │  Tool: Blackbox Exporter                            │
   │  Catches: DNS failures, routing problems,           │
   │           certificate errors, externally-visible    │
   │           issues                                    │
   │  Misses:  Internal resource bottlenecks             │
   └─────────────────────────────────────────────────────┘
```

You need both. Black box tells you the user is experiencing a problem. White box tells you what internally is causing it.
### `prometheus.yml` — the master config

```yaml
global:
  scrape_interval:     15s            # poll targets every 15 seconds
  evaluation_interval: 15s            # evaluate alerting rules every 15 seconds

scrape_configs:
  # ─── Prometheus self-monitoring (confirms it is alive and scraping correctly) ───
  - job_name: "prometheus"
    static_configs:
      - targets: ["prometheus:9090"]
        labels:
          instance: "prometheus"

  # ─── Node Exporter — host hardware and OS metrics ───
  - job_name: "node_exporter"
    static_configs:
      - targets: ["node-exporter:9100"]
        labels:
          instance: "node_exporter"

  # ─── cAdvisor — per-container resource metrics ───
  - job_name: "cadvisor"
    static_configs:
      - targets: ["cadvisor:8080"]
        labels:
          instance: "cadvisor"

  # ─── Blackbox Exporter — external HTTP probes of public URLs ───
  - job_name: "blackbox_http"
    metrics_path: /probe
    params:
      module: [http_2xx]
    static_configs:
      - targets:
          - "https://devopscourseankit.duckdns.org"
          - "https://devopscourseankit.duckdns.org/api/health"
        labels:
          job: "blackbox_http"
    relabel_configs:
      - source_labels: [__address__]
        target_label:  __param_target
      - source_labels: [__param_target]
        target_label:  instance
      - target_label: __address__
        replacement:   blackbox:9115
```
#### Why the relabel_configs for Blackbox specifically?

Node Exporter and cAdvisor expose a `/metrics` endpoint at their own port — Prometheus just scrapes those ports directly. The Blackbox Exporter is different: it's an external probe that needs to be **told which URLs to probe**. By passing the URLs in `prometheus.yml` and using `relabel_configs`, you tell Prometheus: _"Use the Blackbox Exporter to make an HTTP request against this URL and report whether it succeeded."_

The relabeling rewrites the request:

1. Take the configured `target` (`https://devopscourseankit.duckdns.org`).
2. Move it into the `target` URL parameter (`?target=https://...`).
3. Rewrite the actual address Prometheus connects to → `blackbox:9115` (the exporter container).
4. Set `instance` to the original URL so the metrics are labeled cleanly.

So Prometheus actually GETs `http://blackbox:9115/probe?target=https://devopscourseankit.duckdns.org&module=http_2xx`, the exporter probes the URL, and returns `probe_success=1` or `0`.
### `blackbox.yml` — exporter config

```yaml
modules:
  http_2xx:
    prober: http
    timeout: 5s
    http:
      valid_http_versions:  ["HTTP/1.1", "HTTP/2.0"]
      valid_status_codes:   [200, 201, 204]    # what counts as "success"
      method:               GET
      follow_redirects:     true
      tls_config:
        insecure_skip_verify: false            # validate SSL properly
```

This defines a probe module called `http_2xx`. Prometheus references it by name in the scrape config.
### Time-series data format

Every metric Prometheus stores has three parts:

```
   <metric_name> {<labels>}  <value>  <timestamp>
```

Examples:

```
node_cpu_seconds_total {mode="idle", instance="node_exporter"}      1234567.89 1716000000
probe_success         {instance="https://...", job="blackbox_http"} 1          1716000015
container_memory_usage_bytes {name="backend", job="cadvisor"}       134217728  1716000015
```

Labels are what make Prometheus powerful — they let you query _all containers at once_ (`container_memory_usage_bytes`) or drill down to _one specific container_ (`container_memory_usage_bytes{name="backend"}`).
### Docker Compose for the observability stack

This is `observability/compose.yaml` — a separate Compose project from the app stack (recall 4.1).

```yaml
services:
  prometheus:
    image: prom/prometheus:latest
    container_name: prometheus
    restart: unless-stopped
    ports:
      - "9090:9090"
    volumes:
      - ./prometheus.yml:/etc/prometheus/prometheus.yml:ro
      - prometheus_data:/prometheus            # persist time-series database
    command:
      - '--config.file=/etc/prometheus/prometheus.yml'
      - '--storage.tsdb.retention.time=30d'    # keep 30 days of metrics
    networks:
      - monitoring

  grafana:
    image: grafana/grafana:latest
    container_name: grafana
    restart: unless-stopped
    ports:
      - "3000:3000"
    volumes:
      - grafana_data:/var/lib/grafana          # persist dashboards and settings
    environment:
      - GF_SECURITY_ADMIN_USER=admin
      - GF_SECURITY_ADMIN_PASSWORD=your_secure_password
    networks:
      - monitoring

  node-exporter:
    image: prom/node-exporter:latest
    container_name: node-exporter
    restart: unless-stopped
    ports:
      - "9100:9100"
    volumes:
      - /proc:/host/proc:ro                    # mount host's /proc and /sys
      - /sys:/host/sys:ro
      - /:/rootfs:ro
    command:
      - '--path.procfs=/host/proc'
      - '--path.sysfs=/host/sys'
    networks:
      - monitoring

  cadvisor:
    image: gcr.io/cadvisor/cadvisor:latest
    container_name: cadvisor
    restart: unless-stopped
    ports:
      - "8080:8080"
    volumes:
      - /:/rootfs:ro
      - /var/run:/var/run:ro
      - /sys:/sys:ro
      - /var/lib/docker/:/var/lib/docker:ro    # cAdvisor reads Docker's metadata
    networks:
      - monitoring

  blackbox:
    image: prom/blackbox-exporter:latest
    container_name: blackbox
    restart: unless-stopped
    ports:
      - "9115:9115"
    volumes:
      - ./blackbox.yml:/config/blackbox.yml:ro
    command:
      - '--config.file=/config/blackbox.yml'
    networks:
      - monitoring

volumes:
  prometheus_data:
  grafana_data:

networks:
  monitoring:
    driver: bridge
```
#### Why volumes for both Prometheus and Grafana?

- Without a Prometheus data volume, every container restart wipes all collected metrics — all your historical trends are gone.
- Without a Grafana data volume, all your custom dashboards and configured data sources are lost on restart.
#### Why Node Exporter and cAdvisor mount the host filesystems

Look at Node Exporter's volumes:

```yaml
volumes:
  - /proc:/host/proc:ro     # the host's process info
  - /sys:/host/sys:ro       # the host's hardware/kernel info
  - /:/rootfs:ro            # the host's root filesystem
```

Recall from 1.1.8: `/proc` and `/sys` are _virtual_ filesystems exposed by the Linux kernel that contain real-time CPU, memory, and process info. By default, a container only sees its own `/proc` (its own processes — useless for monitoring). Mounting the _host's_ `/proc` lets Node Exporter read the host's CPU and memory state.

Same idea for cAdvisor mounting `/var/lib/docker` — that's where Docker stores image and container metadata; cAdvisor reads it to enrich its container metrics with names and labels.
#### Why a shared `monitoring` network?

Within the `monitoring` Docker network, containers reference each other by service name (e.g., `prometheus:9090`, `cadvisor:8080`). Grafana uses `prometheus:9090` as the data source URL — this only resolves because both are on the same network.

> 🔁 **Recap from 2.2 / 3.1:** service names → DNS through the user-defined bridge → containers find each other by name. Same pattern you've now seen in app stack, K8s services, Prometheus targets, Grafana data sources.
### Deploying via Ansible (the playbook from 4.4)

```yaml
---
- name: Deploy observability stack
  hosts: prod
  become: true
  tasks:
    - name: Create observability directory on VPS
      file:
        path: /opt/observability
        state: directory
        mode: '0755'

    - name: Copy docker-compose file
      copy:
        src: ../observability/compose.yaml
        dest: /opt/observability/compose.yaml

    - name: Copy prometheus config
      copy:
        src: ../observability/prometheus.yml
        dest: /opt/observability/prometheus.yml

    - name: Copy blackbox config
      copy:
        src: ../observability/blackbox.yml
        dest: /opt/observability/blackbox.yml

    - name: Start observability stack
      shell: |
        cd /opt/observability
        docker compose up -d
```

```bash
# Deploy the monitoring stack
ansible-playbook observability.yml

# Access:
# Prometheus: http://187.127.157.4:9090/targets
# Grafana:    http://187.127.157.4:3000/
```
### Verifying — the Prometheus Targets page

Navigate to `http://<VPS-IP>:9090/targets`. You should see all five targets `UP`:

|Endpoint|Up?|Last Scrape|
|---|---|---|
|`https://devopscourseankit.duckdns.org/`|UP|~13s ago|
|`https://devopscourseankit.duckdns.org/api/health`|UP|~20s ago|
|`cadvisor:8080`|UP|~17s ago|
|`node-exporter:9100`|UP|~17s ago|
|`prometheus:9090`|UP|~17s ago|

This is the primary verification step after deploying the observability stack.

![[Pasted image 20260428022719.png | 900]]
### The `up` query

Running the query `up` in the Prometheus expression browser (`http://<VPS-IP>:9090/graph`) shows the health of every scrape target. A value of `1` = target is up and scraping successfully:

```
up{instance="https://devopscourseankit.duckdns.org/api/health", job="blackbox_http"} = 1
up{instance="node_exporter:9100", job="node_exporter"}                                = 1
up{instance="cadvisor:8080", job="cadvisor"}                                          = 1
up{instance="prometheus:9090", job="prometheus"}                                      = 1
up{instance="https://devopscourseankit.duckdns.org/", job="blackbox_http"}            = 1
```

Other useful verification queries:

```promql
probe_success                    # 1 = external probe succeeded, 0 = failed
probe_duration_seconds            # how long the HTTP probe took
probe_http_status_code            # what status code the probe received
```
![[Pasted image 20260428022348.png | 1000]]
### Grafana setup — the welcome screen

![[Pasted image 20260428022551.png]]

When you first access Grafana at `http://<VPS-IP>:3000/`, you see the welcome screen with three steps:

1. **Tutorial** — Grafana fundamentals walkthrough.
2. **Data Sources** — add your first data source (Prometheus).
3. **Dashboards** — create your first dashboard.
#### Step 1 — add Prometheus as a data source

- Go to **Connections → Data Sources → Add → Prometheus**.
- URL: `http://prometheus:9090` (container name works because both are on the `monitoring` network).
- Click **Save & Test** — should show _"Data source is working"_.
#### Step 2 — create a dashboard with `probe_success`

![[Pasted image 20260428023431.png | 1200]]

- **New Dashboard → Add Panel**.
- Panel title: _Panel 1: probe_success (up/down)_.
- Data source: Prometheus.
- Metric: `probe_success`.

The graph shows two time series:

- `probe_success` for `https://devopscourseankit.duckdns.org/` (one color).
- `probe_success` for `https://devopscourseankit.duckdns.org/api/health` (another color).

Both lines at value 1 — both endpoints are up and healthy.

> 💡 **Architectural note worth re-stating:** Grafana is getting the data from Prometheus _only_. It is purely a visualization layer — it does not store anything itself. If Prometheus loses its data, Grafana panels go blank.
#### Step 3 — add a second panel for probe latency

- **Add Panel → Time series**.
- Title: _Probe Latency (Seconds)_.
- Metric: `probe_duration_seconds`.

This panel shows how long each external probe took in seconds. A spike at any time corresponds to high server latency at that moment — for instance, when the k6 load test runs (Section 4.8), you'll see a clear spike here.

![[Pasted image 20260428023615.png | 1200]]
### The complete observability data flow

```
   Prometheus → Scrape /metrics
         ↓
   Node Exporter (Host CPU / RAM / Disk)
         ↓
   cAdvisor      (Container-level details)
         ↓
   Blackbox Exporter → probes public HTTPS URLs
```

Full end-to-end:

1. **Node Exporter** exposes host metrics at `:9100/metrics`. **cAdvisor** exposes container metrics at `:8080/metrics`. **Blackbox Exporter** probes `devopscourseankit.duckdns.org` every 15 s.
2. **Prometheus** scrapes all targets every 15 s → stores as time-series.
3. **Grafana** connects to Prometheus (`prometheus:9090` as data source) → runs PromQL queries → renders dashboards visually.
4. Engineers see CPU trends, container stats, and endpoint availability in one place.
### Common mistakes

1. **Forgetting the `prometheus_data` volume** — Prometheus restarts wipe all metrics history.
2. **Putting Prometheus on a different Docker network than its targets** — scrapes fail with "no such host" because DNS doesn't resolve cross-network.
3. **Setting `scrape_interval` very short** (1s) — cardinality explodes, disk usage balloons. 15s is the sane default.
4. **Mounting `/proc` without `:ro`** — security risk; container could potentially write to the host's `/proc`. Always mount read-only.
5. **Using Grafana's built-in "TestData DB" data source for production** — it's just a demo. The real one is Prometheus.

> 🔁 **Recap (4.7 — Prometheus & Grafana):** Prometheus = pull-model time-series DB. Exporters translate things into the Prometheus format. Node Exporter + cAdvisor = white box (internal). Blackbox Exporter = black box (external). Grafana = visualization only, queries Prometheus. The whole stack lives in `/opt/observability/` and is deployed by Ansible. Verify via `/targets` page and the `up` query.

---
## 4.8 — Grafana k6 — Load Testing
### What problem are we solving?

Your app works perfectly when _you_ visit it. But what about when 100 users hit it simultaneously? 1,000? You don't know — and finding out for the first time _in production_ is a great way to lose customers.

**Load testing** is the process of understanding how your system handles multiple users hitting it simultaneously, _before_ real users do. An application might work perfectly for 1 user but fail for 1,000. Load testing reveals this in a controlled environment so you can fix it.
### Three things load testing validates

1. **Latency** — does response time increase as traffic increases? Measured via P95 and P99 (the time within which 95% / 99% of requests complete).
2. **System stability** — does the app crash or return errors under stress?
3. **Resource behavior** — how do CPU and RAM respond to load? (Visible in Grafana dashboards while k6 runs — you literally watch the CPU spike.)
### What k6 is

**k6** is a scriptable load generator from Grafana Labs. Unlike basic tools (wrk, ab), it lets you write test logic in JavaScript to simulate real-world traffic patterns — different endpoints, realistic wait times between requests, gradual traffic ramp-up, structured assertions.
### Key metrics k6 collects

|Metric|Meaning|
|---|---|
|`http_req_duration p(50)`|Median response time (half of requests faster than this)|
|`http_req_duration p(95)`|95th percentile (95% of requests faster than this)|
|`http_req_duration p(99)`|99th percentile (worst-case experience for most users)|
|`http_req_failed`|Error rate (requests that returned non-2xx)|
|`http_reqs`|Total request count and rate (req/s)|
|`vus`|Virtual users — how many simulated concurrent users at any moment|
|`iteration_duration`|Time for one full loop of your test function|

> ⚙️ **Added Professional Context — what p99 actually means:** If your p99 is 800 ms, it means 99% of requests are faster than 800 ms — but 1% (one in every 100) are slower. At a million requests/day, that's 10,000 slow experiences per day. The "average" or "p50" can look fine while p99 is terrible. **In production at scale, p99 is the metric you optimize for.** We dedicate Bonus Section B.1 entirely to this.
### The test file — `ops/tests/k6/statusboard.js`

```javascript
import http from 'k6/http';
import { check, sleep } from 'k6';

// Traffic ramp-up configuration
export const options = {
    stages: [
        { duration: '15s', target: 5  },   // gradually increase to 5 virtual users over 15s
        { duration: '30s', target: 15 },   // ramp up to 15 virtual users over 30s
        { duration: '15s', target: 0  },   // ramp down to 0 (cooldown period)
    ],
};

const BASE_URL = __ENV.BASE_URL || 'https://devopscourseankit.duckdns.org';

export default function () {
    // Test the health endpoint
    const healthRes = http.get(`${BASE_URL}/api/health`);
    check(healthRes, {
        'health check status is 200': (r) => r.status === 200,
    });

    // Test the main page
    const webRes = http.get(`${BASE_URL}/`);
    check(webRes, {
        'web page status is 200': (r) => r.status === 200,
    });

    sleep(1);                              // 1 second pause between iterations (simulates real user behavior)
}
```
#### Why the gradual ramp-up?

Hitting a server with 100 users simultaneously from zero is **unrealistic and causes an artificial spike** that doesn't match how real traffic grows. Gradual ramp-up reveals at exactly what traffic level performance starts to degrade — which is much more actionable information.

Three stages produce a "pyramid" load shape:

```
   VUs
   15  ┤             ╱─────────╲
   10  ┤          ╱──           ──╲
    5  ┤      ╱──                   ──╲
    0  ┤───╱─                            ─╲────
        0s   15s         45s        60s
        ramp-up  steady-load     ramp-down
```
#### Reading the script

- **`stages`** — array describing how the number of virtual users changes over time.
- **`__ENV.BASE_URL`** — read from an environment variable when the test runs; falls back to the production URL.
- **`http.get`** — makes a real HTTP GET request. Each virtual user runs the `default` function in a loop.
- **`check`** — asserts on the response. Failures show up in the report but don't stop the test.
- **`sleep(1)`** — 1-second pause between iterations to simulate "thinking time." Without it, virtual users hammer the server far faster than real users would.
### Deploying the k6 test file via Ansible

Add this to `deploy.yml`:

```yaml
- name: Ensure k6 directory exists on VPS
  file:
    path: /opt/statusboard/ops/tests/k6
    state: directory

- name: Sync k6 tests to VPS
  copy:
    src: ops/tests/k6/
    dest: /opt/statusboard/ops/tests/k6/
```
### Running the load test from the VPS

```bash
# SSH into the VPS
ssh root@187.127.157.4
cd /opt/statusboard

# Run k6 using Docker (no installation needed on the server)
docker run --rm -i \
  -e BASE_URL="https://devopscourseankit.duckdns.org" \
  -v "$PWD:/work" \
  -w /work \
  grafana/k6 run ops/tests/k6/statusboard.js
```

The `--rm -i` runs the container interactively and removes it afterward. The volume mount at `/work` lets k6 read the test file from the host filesystem.
### Reading the k6 output

```
         /\      Grafana   /‾‾/
    /\  /  \     |\  __   /  /
   /  \/    \    | |/ /  /   ‾‾\
  /          \   |   (  |  (‾)  |
 / __________ \  |_|\_\  \_____/

  execution: local
     script: ops/tests/k6/statusboard.js
  scenarios: 1 scenario, 15 max VUs, 1m30s max duration
   * default: Up to 15 looping VUs for 1m0s over 3 stages

     ✓ health check status is 200
     ✓ web page status is 200

     checks.........................: 100.00% ✓ 458    ✗ 0
     http_req_duration..............: avg=207ms min=8ms med=12ms max=1.2s p(90)=890ms p(95)=1.1s
     http_reqs......................: 229 3.8/s
     vus............................: 1   min=1   max=15
```
#### Interpreting these numbers

|Field|Reading|
|---|---|
|`checks 100.00%`|All requests returned expected status codes — no errors|
|`http_req_duration p(95)=1.1s`|95% of requests completed within 1.1 s. **If your performance target is 500 ms, this tells you the system struggles under 15 concurrent users.**|
|`http_reqs 3.8/s`|The system handled 3.8 requests per second on average|
|`max=1.2s`|The slowest request took 1.2 seconds (the tail)|
|`min=8ms`|The fastest was 8 ms (when not under contention)|
|`med=12ms`|The median was 12 ms — half of requests completed faster|

The big gap between `med=12ms` and `p(95)=1.1s` is the story: most requests are fine, but the tail is bad. Real users hitting the slow tail get a frustrating experience even though averages look reasonable. This is exactly why p95/p99 are the metrics that matter, not averages — covered in depth in Bonus B.1.
### Watching Grafana while k6 runs

Open your Grafana CPU usage dashboard simultaneously. You will see the CPU spike visibly as virtual users ramp up — this is exactly the resource behavior k6 is designed to expose. The spike on the _Probe Latency_ panel and the spike on the _node_exporter CPU_ panel happen together, and the gap before the system recovers is your real-world recovery time.

This concrete loop (run k6 → watch Grafana) is **the** way to learn capacity-planning intuition. Run it on a deliberately undersized server first, watch it choke, then size up and re-run.
### What to do with k6 results

1. **Set a SLO (Service Level Objective).** Decide: "p95 latency must be under 500 ms at 100 concurrent users."
2. **Run k6 in CI.** Fail the pipeline if SLO is breached. (Run it against staging, not prod.)
3. **Investigate when limits are hit.** CPU? Memory? DB connections? Look at the matching Grafana panels.
4. **Scale up.** Add resources, optimize code, add caching — whichever gets you under the SLO.
5. **Repeat.** Capacity testing is not a one-time event; you re-run it as features and traffic grow.
### Production load-testing rules

- **Test against staging, not prod.** Real users on production should never be the load.
- **Coordinate with the team.** A surprise load test that takes down a service is a career event.
- **Test from outside the network.** Running k6 on the same VPS as your app means you're competing for the same CPU.
- **Test realistic patterns.** Don't just hit `/health` — hit the actual user flows (login, fetch tasks, write task).
- **Ramp gradually.** Realistic traffic grows; instant 1000-user spikes are unrealistic.

> 🔁 **Recap (4.8 — k6):** k6 simulates concurrent users with JavaScript test scripts. `stages` controls the load shape. `http_req_duration p(95)` is the metric that matters. Run k6 while watching Grafana to see resource spikes in real time. Always test against staging. The big gap between median and p95/p99 is the tail latency story.

---
---

This concludes **Section 4 — Infrastructure, Automation & Monitoring**. Coverage:

- ✅ **4.1 Tools Overview** — full architecture, producer-consumer chain, why two Compose projects on one VPS.
- ✅ **4.2 Sublyzer** — application-layer observability gap, setup, dashboard signals, GitHub-aware AI fixes, the 3-layer observability picture.
- ✅ **4.3 Terraform** — full IaC philosophy, Provisioning vs Config Mgmt, complete CLI lifecycle, all 5 block types, file structure, `terraform.tfstate` deep dive, remote backends (S3+DynamoDB), `TF_VAR_*` env-var pattern, `terraform import`.
- ✅ **4.4 Ansible** — agentless model, comparison with Chef/Puppet/Salt, inventory/playbooks/modules/idempotency, full bootstrap+deploy+observability playbooks, Jinja2 templates, ansible-vault for secrets, the remote one-liner trick.
- ✅ **4.5 Caddy** — automatic HTTPS, ACME flow, Caddyfile, Compose integration, `caddy_data` rate-limit warning, full troubleshooting playbook.
- ✅ **4.6 Uptime Kuma** — synthetic monitoring, the 3 pillars of observability (Metrics/Logs/Traces) + Alerting, dashboard interpretation (healthy + 502 incident), homework.
- ✅ **4.7 Prometheus & Grafana** — pull-model time-series, exporters (Node/cAdvisor/Blackbox), white vs black box, full annotated `prometheus.yml` and `blackbox.yml`, full observability `compose.yaml`, why the host's `/proc` is mounted, Grafana setup with two panels, common mistakes.
- ✅ **4.8 Grafana k6** — full `statusboard.js`, ramp-up philosophy, output interpretation, the median-vs-p95 tail story, watch-Grafana-while-running, production load-testing rules.

This completes the four-section coverage of your raw notes. Everything you wrote — explained, expanded, and connected.

Next: **Bonus Section** — the industry-depth additions your senior asked for: alerting flow (server down → who gets paged), p99 deep dive, ArgoCD & GitOps, Helm, ELK/EFK, APM/tracing, Datadog & Splunk, AWS networking deep, security essentials, full interview question bank, master cheat-sheet.

---
# BONUS — INDUSTRY DEPTH

This section extends the raw-notes content with topics that weren't worked out in detail in the original notes — plus a few add-ons that any real DevOps engineer needs to know. Treat this as "what they expect you to know in interviews and on the job, beyond the course."

---
## B.1 — Latency, p50/p95/p99 — what you actually optimize for

You met `p(95)` and `p(99)` in the k6 output (Section 4.8). They look innocent. They're the most important latency metrics you'll ever measure, and beginners almost universally use the wrong one (the **average**) and ship slow products as a result.
### What "p99" actually means

If you take _every_ request that hit your API today, sort them by response time fastest-to-slowest, and look at the request that's 99% of the way down the list, _that_ request's response time is your **p99**.

```
   1,000 requests today, sorted by response time:
   ┌──────────────────────────────────────────────────┐
   │ Request 1   →  3 ms     ← fastest                │
   │ Request 2   →  4 ms                              │
   │   ...                                            │
   │ Request 500 →  45 ms    ← p50 (median)           │
   │   ...                                            │
   │ Request 950 →  280 ms   ← p95                    │
   │   ...                                            │
   │ Request 990 →  820 ms   ← p99                    │
   │   ...                                            │
   │ Request 999 →  1.2 s                             │
   │ Request 1000→  4.5 s    ← slowest (p100)         │
   └──────────────────────────────────────────────────┘
```

So:

- **p50 = 45 ms** means _half_ of users got responses faster than 45 ms.
- **p95 = 280 ms** means _95%_ of users got responses faster than 280 ms; the slowest 5% were slower.
- **p99 = 820 ms** means _99%_ of users got responses faster than 820 ms; the slowest 1% were slower.
### Why "average" lies

Imagine 999 fast requests at 50 ms and 1 awful request at 30 seconds:

```
   Average  = (999 × 50 + 30,000) / 1000 = 80 ms     ← looks "fine"
   p99      = 30,000 ms = 30 s                        ← reality
```

The average is dragged down by all the fast requests. It hides the 30-second nightmare. **Averages are a lie at scale.** If the average says 80 ms but p99 says 30 seconds, real users are having a terrible time and you don't know it.
### Why the **tail** is what users notice

A user navigates your site. They click around — maybe make 20 actions. Each action is a request. With p99 = 820 ms, there's a 1-in-100 chance any single request hits the slow tail. Across 20 actions, the probability that _at least one_ will be slow is:

```
   P(at least one slow) = 1 - (0.99)^20 = 18.2%
```

Roughly 1 in 5 user sessions hits a slow request. That's a _lot_ of users complaining about a "slow site" even when your average looks great.

This is why senior engineers care about p99 (and p99.9, and p99.99 at FAANG-scale): **the tail is the user experience**. Optimizing the median while ignoring the tail is optimizing the wrong thing.

> 💡 **The Amazon study, often quoted in DevOps talks:** every 100 ms of additional latency cost Amazon ~1% in sales. That's not the median — that's the tail crossing into "noticeable." Latency at scale is money.
### What causes a bad tail

The fast 99% of requests are usually doing the same thing — fetching the same cached page, hitting the same indexed query. The slow 1% are the _outliers_:

|Tail cause|Example|
|---|---|
|**Cold cache**|First request after a deployment or cache eviction|
|**Garbage collection pause**|JVM/Node.js GC stops the world for 200 ms|
|**Database lock contention**|Another query is holding a row lock you need|
|**Unindexed query**|A rarely-used filter triggers a full table scan|
|**Network jitter**|A switch between zones hiccups for 80 ms|
|**Noisy neighbor**|Another tenant on the same hypervisor saturates a shared resource|
|**Slow downstream dependency**|An external API or DB takes 600 ms instead of 5 ms|
|**Connection pool exhausted**|App waits for a free DB connection|
### Setting an SLO around p99 (the practical use)

A **Service Level Objective** is a target you set:

```
   "99% of API requests must complete in under 500 ms,
    measured over a rolling 30-day window."
```

This is your contract with users. Now everyone in the org has a single number to optimize against. CI runs k6 against staging; the build fails if `p(99) > 500ms` over the test run. Production is monitored via Prometheus; an alert fires if p99 over the last 5 minutes exceeds the threshold.
### PromQL for percentiles

Prometheus stores latencies as **histograms** (buckets like "<= 100 ms", "<= 250 ms", etc.). You query percentiles with `histogram_quantile`:

```promql
# p95 latency over the last 5 minutes
histogram_quantile(0.95, sum(rate(http_request_duration_seconds_bucket[5m])) by (le))

# p99 latency, broken down by route
histogram_quantile(0.99, sum(rate(http_request_duration_seconds_bucket[5m])) by (le, route))
```

That second query is the gold standard: _"Show me the p99 latency, but per route, so I know which endpoint is the slow one."_
### Latency in the rest of this guide

|Where it appeared|What it told you|
|---|---|
|k6 output (Section 4.8)|`p(95)=1.1s` — the system struggles under 15 concurrent users|
|Uptime Kuma response time graph|Probe latency over time — basic external view|
|Grafana panel for `probe_duration_seconds`|Latency rises during k6 runs, then recovers|
|Sublyzer Performance score|An app-side aggregate of perceived speed|

> 🔁 **Recap (B.1):** Average lies; percentiles tell the truth. p99 is what real users feel. Set an SLO around p99. Use `histogram_quantile` in PromQL to query it. The tail (slowest 1%) is where almost all "the site is slow" complaints come from.

---
## B.2 — Alerting — the full "server down → on-call gets paged" flow

You asked specifically how alerts get sent when a server goes down. Here is the complete chain, end to end.
### The problem

We have all this monitoring data. But monitoring is _passive_ — it sits in dashboards waiting for someone to look. At 3 AM, when production goes down, **nobody is looking**. You need _active_ notification: a phone ringing, an email firing, a Slack message arriving.
### The 4-stage alert chain

```
   ┌────────────────────────────────────────────────────────────┐
   │  1. METRIC SOURCE                                          │
   │     Prometheus is scraping `up` every 15s.                 │
   │     The backend probe returns probe_success=1, then 0.     │
   └────────────────────────────────────────────────────────────┘
                               │
                               ▼
   ┌────────────────────────────────────────────────────────────┐
   │  2. ALERTING RULE (in Prometheus)                          │
   │     "If probe_success == 0 for 1 minute, fire an alert."   │
   │     This becomes an active "Alert" inside Prometheus.      │
   └────────────────────────────────────────────────────────────┘
                               │
                               ▼
   ┌────────────────────────────────────────────────────────────┐
   │  3. ALERTMANAGER                                           │
   │     Receives the alert from Prometheus.                    │
   │     • Groups related alerts together                       │
   │     • Deduplicates                                         │
   │     • Silences during maintenance windows                  │
   │     • Routes to the right notification channel             │
   └────────────────────────────────────────────────────────────┘
                               │
                               ▼
   ┌────────────────────────────────────────────────────────────┐
   │  4. NOTIFICATION CHANNEL                                   │
   │     • Slack message in #alerts-prod                        │
   │     • Email to on-call list                                │
   │     • SMS / Phone via PagerDuty / Opsgenie                 │
   │     • Webhook into a custom system                         │
   └────────────────────────────────────────────────────────────┘
```
### Stage 1 — the metric

Prometheus already collects `probe_success` (Section 4.7). When the backend goes down, the Blackbox Exporter's probe fails on the next 15-second cycle, and Prometheus stores `probe_success=0`. The data is there; we now need to _act_ on it.
### Stage 2 — the alerting rule

Alerting rules live in YAML files and are loaded by Prometheus. Add this file: `observability/rules/alerts.yml`:

```yaml
groups:
  - name: availability
    interval: 30s
    rules:
      - alert: BackendDown
        expr: probe_success{instance="https://devopscourseankit.duckdns.org/api/health"} == 0
        for: 1m                            # alert only if the condition holds for 1 minute
        labels:
          severity: critical
        annotations:
          summary: "Backend API is down"
          description: "The /api/health endpoint has been failing for over 1 minute."
          runbook_url: "https://wiki.company.com/runbooks/backend-down"

      - alert: HighErrorRate
        expr: rate(http_requests_total{status=~"5.."}[5m]) > 0.05
        for: 2m
        labels:
          severity: warning
        annotations:
          summary: "5xx error rate above 5%"
```

Wire it into `prometheus.yml`:

```yaml
rule_files:
  - "rules/*.yml"

alerting:
  alertmanagers:
    - static_configs:
        - targets: ["alertmanager:9093"]
```

Reload Prometheus (`docker compose restart prometheus`). Now Prometheus continuously evaluates these expressions and emits firing alerts to Alertmanager whenever they're true.
#### The `for:` clause is critical

`for: 1m` means: "fire only if the condition has been true continuously for 1 minute." Without it, a single 15-second blip pages you. With it, transient failures are filtered out and only sustained outages alert.

Tune `for:` per alert:

- Quick reboot on a non-critical service: `for: 5m` (don't page for self-healing blips).
- Backend totally down: `for: 1m` (real outage, real users affected, page fast).
- Disk almost full: `for: 30m` (slow problem; 30 min lead time is fine).
### Stage 3 — Alertmanager

Alertmanager is a separate Prometheus-ecosystem service. Its job: take raw firing alerts and turn them into well-routed notifications.
#### Add it to `observability/compose.yaml`

```yaml
alertmanager:
  image: prom/alertmanager:latest
  container_name: alertmanager
  restart: unless-stopped
  ports:
    - "9093:9093"
  volumes:
    - ./alertmanager.yml:/etc/alertmanager/alertmanager.yml:ro
    - alertmanager_data:/alertmanager
  networks:
    - monitoring
```
#### `alertmanager.yml` — minimal Slack + email config

```yaml
global:
  resolve_timeout: 5m
  slack_api_url: "https://hooks.slack.com/services/T0000/B0000/XXXXXXXX"

route:
  group_by:        ['alertname']
  group_wait:      30s            # wait this long before firing the first notification
  group_interval:  5m             # if more alerts in same group, wait this long between batches
  repeat_interval: 3h             # if still firing, repeat every 3 hours
  receiver: 'slack-default'
  routes:
    - match:
        severity: critical
      receiver: 'pagerduty-oncall'

receivers:
  - name: 'slack-default'
    slack_configs:
      - channel: '#alerts-prod'
        title:   '{{ .GroupLabels.alertname }}'
        text:    '{{ range .Alerts }}{{ .Annotations.summary }}{{ "\n" }}{{ .Annotations.description }}{{ end }}'

  - name: 'pagerduty-oncall'
    pagerduty_configs:
      - service_key: 'YOUR-PAGERDUTY-KEY'

inhibit_rules:
  # If the whole VPS is down, don't also page about every individual service on it.
  - source_match: { alertname: 'NodeDown' }
    target_match: { alertname: 'BackendDown' }
    equal: ['instance']
```
#### What Alertmanager actually does

|Feature|What it gives you|
|---|---|
|**Grouping**|Five alerts at once? Send one consolidated message, not five separate ones.|
|**Routing**|`severity: critical` → PagerDuty (wakes someone up). `severity: warning` → Slack only.|
|**Deduplication**|Same alert firing on multiple Prometheus instances? Send it once.|
|**Silencing**|"Maintenance from 2-3 AM, suppress all alerts for 1 hour."|
|**Inhibition**|If the whole node is down, suppress the per-service alerts on it.|
### Stage 4 — the notification channels
#### Slack

The simplest channel. Almost every team uses it. Set up an incoming webhook in your Slack workspace, paste the URL into Alertmanager, and you'll see alerts appear in `#alerts-prod`.
#### Email

Configure SMTP in Alertmanager. Useful for non-urgent alerts. Don't rely on email for "wake the on-call up" — emails are easy to miss.
#### PagerDuty / Opsgenie / VictorOps

These are **incident management** services. They:

- Phone-call the on-call engineer until they acknowledge.
- Escalate to the secondary engineer if the primary doesn't respond in N minutes.
- Provide the on-call rotation schedule.
- Track MTTA (mean time to acknowledge) and MTTR (mean time to resolve).
- Run incident postmortems.

A typical on-call rotation:

```
   Week of April 20-26:  Ankit  (primary), Bob (secondary)
   Week of April 27-May 3: Carol (primary), Dave (secondary)
   ...etc
```

When `severity: critical` fires:

1. PagerDuty phones Ankit's mobile.
2. If Ankit doesn't acknowledge within 5 minutes → phones Bob.
3. If neither responds in 15 minutes → escalates to the team lead.

This is how "my server went down at 3 AM" actually wakes someone up.
### A complete alert lifecycle, step by step

Let's trace what happens when our backend container crashes at 3:14 AM.

```
3:14:00  Backend container OOMKilled by Docker (out of memory).
3:14:01  Caddy starts returning 502 to /api/* requests.
3:14:15  Blackbox Exporter probes /api/health → fails. Prometheus stores probe_success=0.
3:14:30  Next probe → still 0.
3:14:45  Next probe → still 0.
3:15:15  60 seconds elapsed. The `for: 1m` condition is satisfied.
         Prometheus marks `BackendDown` as FIRING.
3:15:15  Prometheus pushes the alert to Alertmanager (port 9093).
3:15:15  Alertmanager applies group_wait (30s) — waits in case more alerts arrive.
3:15:45  No more alerts in the group. Alertmanager sends:
            • Slack message to #alerts-prod
            • PagerDuty incident → phone calls on-call (Ankit)
3:16:01  Ankit's phone rings. He acknowledges in PagerDuty. MTTA = 46s.
3:16:01  PagerDuty notifies the team in Slack: "Ankit is on it."
3:17:00  Ankit SSHs in, sees the OOMKill, raises the container memory limit, restarts.
3:17:30  Backend recovers. probe_success returns to 1.
3:18:30  After 1 minute of green, Prometheus marks the alert RESOLVED.
3:18:30  Alertmanager sends "Resolved" notification to Slack.
3:18:30  PagerDuty closes the incident. MTTR = 4m 30s.
```

That whole sequence is automated end-to-end. A human is only in the loop from 3:16:01 to 3:17:00. Without this chain, the outage would last until someone at the office noticed at 9 AM — _six hours_ of downtime instead of four minutes.
### What a good alert looks like

Bad alert:

> _"CPU > 80%."_

What's wrong: doesn't tell you what's affected, who should care, or what to do.

Good alert:

> **`HighBackendErrorRate`**
> 
> - **Summary:** Backend 5xx error rate is 8.2% over last 5 minutes (threshold 5%)
> - **Description:** The `/api/tasks` endpoint returns 500s. Likely DB connection pool exhausted.
> - **Runbook:** https://wiki.company.com/runbooks/backend-error-rate
> - **Dashboard:** https://grafana.example.com/d/backend
> - **Started:** 03:14 UTC
> - **Severity:** critical

Five rules for good alerts:

1. **Actionable.** If the on-call can't _do_ something about it, it's not an alert — it's a notification.
2. **Symptom, not cause.** Alert on "users are seeing errors" rather than "container restarted." A restart that doesn't affect users is fine.
3. **Has a runbook.** Link to documentation explaining how to diagnose and fix.
4. **Severity-tagged.** `critical` pages someone; `warning` posts to Slack; `info` is a heads-up.
5. **Has a clear "resolved."** When the condition stops being true, the resolution notification fires automatically.

> 🔁 **Recap (B.2 — Alerting):** Prometheus rule fires → Alertmanager groups/routes → Slack/PagerDuty notifies → on-call rotation kicks in. The `for:` clause prevents flapping. Severity routing splits "wake someone up" from "post to Slack." Good alerts are actionable, symptom-based, runbook-linked, and severity-tagged.

---
## B.3 — ArgoCD & GitOps
### What problem are we solving?

Our CD pipeline so far (Section 1.10) **pushes** changes to the server: GitHub Actions runs SSH into EC2 and runs `git pull && docker compose up -d`. That's the **push model**.

It works at one server. It breaks at scale:

|Push-model problem|Effect|
|---|---|
|Long-lived SSH credentials in CI|Stolen GitHub Actions token = production access|
|5 servers? CI has to push to all 5|Race conditions, half-deployed state|
|What's actually running in prod right now?|"Whatever was last pushed" — no audit trail of cluster state|
|Drift|Someone SSHs in and edits something. Nobody knows.|
|Rollback|Manual `git revert + push + redeploy + hope`|

Modern teams flip the model: production **pulls** from Git rather than CI pushing into production. That's **GitOps**.
### What GitOps is

**GitOps** is a deployment philosophy with three rules:

1. **Git is the single source of truth.** The desired state of production is fully described in a Git repo (Kubernetes YAML, Helm values, Terraform config). If it's not in Git, it doesn't exist in prod.
2. **A controller in the cluster watches Git** and continuously reconciles the live cluster to match what's in Git. Drift detected? Auto-corrected.
3. **All changes happen via Git.** Want to deploy a new version? Open a PR that bumps the image tag in Git. Merge it. The controller picks it up. No CI step pushes anything into the cluster.

This flips the trust direction:

```
   Push model (old):                 Pull model (GitOps):
   CI ──→ pushes ──→ Cluster         Git Repo  ←── pulls ←── Cluster
                                        ↑
                                        │
                                        CI builds images and updates the repo,
                                        but never touches the cluster directly
```

The cluster never trusts CI. CI never has cluster credentials.
### What ArgoCD is

**ArgoCD** is the most popular GitOps controller for Kubernetes. You install it inside your cluster. You point it at a Git repo containing Kubernetes manifests. ArgoCD continuously syncs the cluster to match the repo.

```
   ┌────────────────────────────────────────────────────────────┐
   │  Git Repo (the desired state)                              │
   │     k8s/                                                   │
   │     ├── backend-deployment.yaml                            │
   │     ├── backend-service.yaml                               │
   │     └── ingress.yaml                                       │
   └────────────────────────────────────────────────────────────┘
                            │
                            │ poll every 3 minutes
                            ▼
   ┌────────────────────────────────────────────────────────────┐
   │  ArgoCD (running in the cluster)                           │
   │  • Reads manifests from Git                                │
   │  • Compares to live cluster state                          │
   │  • If different → reconciles (kubectl apply)               │
   │  • Displays drift visually in its dashboard                │
   └────────────────────────────────────────────────────────────┘
                            │
                            ▼
   ┌────────────────────────────────────────────────────────────┐
   │  Kubernetes Cluster (the actual state)                     │
   └────────────────────────────────────────────────────────────┘
```
### How a deployment looks under GitOps

The deploy flow becomes:

```
   1. Dev opens PR: bump image tag in k8s/backend-deployment.yaml from 1.4 to 1.5
   2. PR review + CI checks
   3. PR merged to main
   4. ArgoCD (within ~3 minutes) detects the change in Git
   5. ArgoCD applies the new manifest → K8s rolls out v1.5
   6. ArgoCD UI shows "Synced" green status
```

No SSH key. No CI credentials in the cluster. No manual `kubectl apply`. The PR is the deployment.
### ArgoCD `Application` resource

ArgoCD's central object is an `Application` — a "this-Git-folder maps to this-cluster-namespace" link.

```yaml
apiVersion: argoproj.io/v1alpha1
kind: Application
metadata:
  name: mern-app
  namespace: argocd
spec:
  project: default
  source:
    repoURL: https://github.com/ankitsangwan/mern-k8s.git
    targetRevision: main
    path: k8s/                              # subfolder containing manifests
  destination:
    server: https://kubernetes.default.svc  # the cluster ArgoCD is running in
    namespace: production
  syncPolicy:
    automated:
      prune: true                           # delete resources no longer in Git
      selfHeal: true                        # revert manual changes back to Git state
    syncOptions:
      - CreateNamespace=true
```

`automated.selfHeal: true` is the magic. If someone `kubectl edit`s a Deployment by hand, ArgoCD notices within minutes and reverts it. **Drift becomes literally impossible.**
### Image tag updates — the missing piece

Q: "If Git is the source of truth, who updates the image tag in Git when CI builds a new image?"

A: A separate tool, called an **image updater**. ArgoCD has one (`argocd-image-updater`). Its job:

1. Watch a Docker registry for new image versions.
2. When it sees a new image matching a configured tag pattern, commit a change to the Git repo bumping the tag.
3. ArgoCD then sees the Git change and rolls out the new image.

The whole automation is now: CI builds → pushes image → image-updater commits to Git → ArgoCD deploys. CI never touched the cluster.
### Where ArgoCD shines (and where it doesn't)

**Use ArgoCD when:**

- You're on Kubernetes.
- You have multiple clusters (dev / staging / prod).
- You want auditable, drift-free deployments.
- You're scaling to many microservices.

**You probably don't need ArgoCD when:**

- You have one VPS and Docker Compose. (The pull-from-Git pattern can still apply manually with cron + `git pull`, but the value-add is small.)
- You're a one-engineer team. (The push model from Section 1.10 is fine.)
### GitOps competitors

|Tool|Vendor|Notes|
|---|---|---|
|**ArgoCD**|CNCF / Intuit|The most popular; great UI; great multi-cluster support|
|**Flux**|CNCF / Weave|Lighter, more CLI-driven, often used with Helm|
|**Spinnaker**|Netflix|Full multi-cloud delivery platform; heavyweight|
|**Jenkins X**|CD Foundation|Jenkins reimagined for K8s + GitOps|

ArgoCD and Flux are interchangeable for most teams. Pick ArgoCD if you want the visual dashboard.

> 🔁 **Recap (B.3 — ArgoCD/GitOps):** Git is the source of truth. A cluster-side controller pulls from Git and reconciles. Reversed credential flow makes prod safer. Drift is auto-corrected by `selfHeal`. Image-updater bridges the "new image build → tag bump in Git" gap. ArgoCD is the dominant GitOps tool for Kubernetes; Flux is the lighter alternative.

---
## B.4 — Helm — package manager for Kubernetes
### What problem are we solving?

Take our Kubernetes MERN stack from Section 3.3. To deploy it, you have:

```
k8s/
├── mongo-deployment.yaml
├── mongo-service.yaml
├── mongo-pvc.yaml
├── backend-deployment.yaml
├── backend-service.yaml
├── backend-configmap.yaml
├── backend-secret.yaml
├── frontend-deployment.yaml
├── frontend-service.yaml
└── ingress.yaml
```

That's already 10 files. Now you want to deploy _the same thing_ to dev, staging, and prod with different image tags, different replica counts, different domains. You'd be tempted to copy all 10 files three times — and now you have 30 files that need to stay in sync.

This doesn't scale. We need a **template engine** for Kubernetes manifests. That's Helm.
### What Helm is

**Helm** is a package manager for Kubernetes. It does two things:

1. **Templates manifests** — write your YAML once with placeholders; render with different values per environment.
2. **Distributes packages** — community "charts" (Helm packages) for everything: Postgres, Redis, Prometheus, ingress-nginx, cert-manager. `helm install ingress-nginx ingress-nginx/ingress-nginx` and you're done.

A Helm chart is to Kubernetes what an `apt install` is to Linux.
### Helm vocabulary

| Term           | Meaning                                                                      |
| -------------- | ---------------------------------------------------------------------------- |
| **Chart**      | A package of templated K8s manifests + a `values.yaml` for configuration.    |
| **Release**    | A specific _deployed instance_ of a chart in a cluster (e.g., "mern-prod").  |
| **Values**     | The configuration variables passed to a chart at install time.               |
| **Repository** | A collection of charts hosted on the internet. (Like Docker Hub for charts.) |
### Chart structure

```
mern-chart/
├── Chart.yaml                 # metadata (name, version, app version)
├── values.yaml                # default config values
├── templates/
│   ├── backend-deployment.yaml
│   ├── backend-service.yaml
│   ├── frontend-deployment.yaml
│   ├── frontend-service.yaml
│   ├── mongo-deployment.yaml
│   ├── mongo-pvc.yaml
│   └── ingress.yaml
└── charts/                    # subcharts (optional dependencies)
```
### Templated manifest example

`templates/backend-deployment.yaml`:

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: {{ .Release.Name }}-backend          # ← templated; substituted at install
  labels:
    app: {{ .Release.Name }}-backend
spec:
  replicas: {{ .Values.backend.replicas }}    # ← from values.yaml
  selector:
    matchLabels:
      app: {{ .Release.Name }}-backend
  template:
    metadata:
      labels:
        app: {{ .Release.Name }}-backend
    spec:
      containers:
        - name: backend
          image: {{ .Values.backend.image.repository }}:{{ .Values.backend.image.tag }}
          ports:
            - containerPort: {{ .Values.backend.port }}
          env:
            - name: MONGO_URI
              value: {{ .Values.mongo.uri | quote }}
```

`values.yaml` (defaults):

```yaml
backend:
  replicas: 1
  port: 5000
  image:
    repository: ankitsangwan/mern-backend
    tag: 1.0
mongo:
  uri: mongodb://mongo:27017/dev_db
```
### Per-environment overrides

`values-prod.yaml`:

```yaml
backend:
  replicas: 3
  image:
    tag: 1.5
mongo:
  uri: mongodb://mongo-prod:27017/prod_db
```

Install:

```bash
# Dev — uses defaults from values.yaml
helm install mern-dev ./mern-chart

# Production — defaults overridden by values-prod.yaml
helm install mern-prod ./mern-chart -f values-prod.yaml -n production --create-namespace
```

Now the _same_ chart produces different rollouts for different environments.
### Helm CLI essentials

```bash
helm install <release-name> <chart-path-or-repo>    # install
helm upgrade <release-name> <chart>                  # update
helm upgrade --install <name> <chart>                # install or upgrade (idempotent)
helm uninstall <release-name>                        # delete
helm list                                            # list releases
helm history <release-name>                          # rollback history
helm rollback <release-name> 3                       # roll back to revision 3
helm template ./mern-chart                           # render templates without installing (debug)
helm lint ./mern-chart                               # validate
```
### Public Helm charts you'll meet

```bash
# Add a public repo
helm repo add bitnami https://charts.bitnami.com/bitnami

# Install Postgres in seconds
helm install my-postgres bitnami/postgresql

# Install Redis
helm install my-redis bitnami/redis

# Install ingress-nginx
helm install ingress-nginx ingress-nginx/ingress-nginx -n ingress-nginx --create-namespace

# Install cert-manager
helm install cert-manager jetstack/cert-manager -n cert-manager --create-namespace --set installCRDs=true

# Install Prometheus + Grafana stack
helm install kube-prometheus-stack prometheus-community/kube-prometheus-stack -n monitoring --create-namespace
```

That last one is the **kube-prometheus-stack** — a curated bundle of Prometheus, Grafana, Alertmanager, Node Exporter, kube-state-metrics, and pre-built dashboards. One command, full observability stack on Kubernetes. This is what production teams actually use instead of the manual setup we walked through in 4.7.
### Helm + ArgoCD = the production combo

Most production K8s setups combine them: charts are templated with Helm, then ArgoCD references the chart and `values-<env>.yaml` from a Git repo. Bumping `values-prod.yaml` to a new image tag triggers an ArgoCD sync, which renders the chart and applies it.

> 🔁 **Recap (B.4 — Helm):** Helm = templating + packaging for Kubernetes. Charts have `templates/` + `values.yaml`. Per-env overrides via `-f values-<env>.yaml`. Public charts (`kube-prometheus-stack`, `cert-manager`, `ingress-nginx`, `bitnami/postgresql`) are the fastest path to running infrastructure. Helm + ArgoCD is the standard production combo.

---
## B.5 — Logging stacks: ELK / EFK

We covered Metrics (Section 4.7) thoroughly. Now the **second pillar of observability — Logs**.
### What problem are we solving?

Your backend app writes logs to stdout. With one container on one server you `docker logs backend` and read them. With:

- 50 backend pods on 5 nodes,
- across dev / staging / prod,
- generating thousands of log lines per second,

you can't `docker logs` your way out of an incident. You need centralized, searchable, queryable log storage. That's a **logging stack**.
### The three pieces of a logging stack

```
   App Containers      →   Collector / Shipper   →   Searchable Storage   →   UI / Search
   (stdout, files)         (Filebeat, Fluentd,         (Elasticsearch,         (Kibana, Grafana)
                            Fluent Bit, Promtail,       Loki, Splunk,
                            Logstash)                   CloudWatch Logs)
```

The names you'll meet, organized by stack:

|Stack|Collector|Storage|UI|
|---|---|---|---|
|**ELK**|Logstash|Elasticsearch|Kibana|
|**EFK**|Fluentd / Fluent Bit|Elasticsearch|Kibana|
|**PLG**|Promtail|Loki|Grafana|
|**Splunk**|Splunk Forwarder|Splunk Indexer|Splunk Web|
|**Datadog Logs**|Datadog Agent|Datadog cloud|Datadog UI|
### ELK — the original elephant

**E**lasticsearch + **L**ogstash + **K**ibana, all from Elastic NV.

- **Logstash** — collects logs from many sources, parses/transforms them, sends to Elasticsearch. Heavy (JVM-based).
- **Elasticsearch** — full-text search engine. Indexes every log line so you can search across billions of lines in milliseconds.
- **Kibana** — web UI for searching, dashboards, alerting.

ELK was the dominant logging stack for the last decade.
### EFK — the Kubernetes-friendly variant

**E**lasticsearch + **F**luentd + **K**ibana.

Fluentd is much lighter than Logstash, written in Ruby/C, and integrates beautifully with K8s — a Fluentd DaemonSet runs one pod per node, automatically picks up every container's stdout, parses Kubernetes labels, and ships everything to Elasticsearch.

For Kubernetes, EFK is preferred over ELK.
### PLG — the modern challenger

**P**romtail + **L**oki + **G**rafana.

Loki is the new kid (from Grafana Labs). Its insight: _don't index the logs — index only the labels_. This makes it dramatically cheaper than Elasticsearch. You search by label first (`{namespace="prod", app="backend"}`), then grep within the matching logs.

Loki + Grafana also means: same UI for logs and metrics, same alerting engine, same login. For a Prometheus-using shop, PLG is the natural choice.
### A tiny example with Loki

```yaml
# loki-compose.yaml (snippet)
services:
  loki:
    image: grafana/loki:latest
    ports: ["3100:3100"]

  promtail:
    image: grafana/promtail:latest
    volumes:
      - /var/log:/var/log
      - /var/lib/docker/containers:/var/lib/docker/containers:ro
      - ./promtail-config.yaml:/etc/promtail/config.yaml
    command: -config.file=/etc/promtail/config.yaml
```

Add Loki as a Grafana data source (`http://loki:3100`). Now in Grafana → Explore → Loki, you can query:

```
{container="backend"} |= "ERROR"
```

This finds every line containing "ERROR" in the backend container's logs across the entire cluster.
### Cost realities

Logging is expensive. Production apps produce gigabytes of logs per day, indexed across weeks of retention. Naïve ELK setups can cost more than the apps themselves. Cost controls every team uses:

1. **Sample logs.** Keep 100% of ERRORs, 10% of INFOs, 1% of DEBUGs.
2. **Drop noisy lines.** "Health check passed" 4× per second × 1000 pods = nothing useful, gigs of waste.
3. **Set retention.** 7-day hot, 30-day warm, 90-day cold (cheap S3) for compliance.
4. **Use Loki instead of Elasticsearch** for high-volume / low-search-frequency workloads.
### Where this fits with your existing stack

Recall Sections 4.7 — we set up Prometheus and Grafana for metrics. To add logs, install Loki + Promtail and add Loki as a Grafana data source. You now have **metrics and logs in one Grafana**, queryable side by side. That's a real production observability stack.

> 🔁 **Recap (B.5 — Logging):** Logs are the second pillar of observability. ELK/EFK use Elasticsearch (heavy, expensive, full-text search). PLG (Promtail + Loki + Grafana) is the modern lightweight alternative — labels-only indexing, cheap. For Kubernetes, EFK or PLG are the standard choices. Always plan for sampling and retention; raw log volume is a budget killer.

---
## B.6 — APM & distributed tracing — the third pillar

The third pillar of observability is **traces**.
### What problem are we solving?

A single user request flows through your system: browser → load balancer → frontend → backend → database. At any of those hops, latency can sneak in. Metrics tell you "the system was slow at 10:34." Logs tell you "this line was logged at 10:34." But neither tells you the _path_ a particular slow request took, with timing at every hop.

Traces do.
### What a trace is

A **trace** is the recorded path of a single request across services, timestamped at each hop. It looks like a flame graph:

```
   Trace ID: abc123  (Total: 847 ms)

   ├─ frontend.render                           [ 80ms]
   ├─ backend.GET /api/tasks                    [340ms]
   │  ├─ db.query("SELECT * FROM tasks")        [ 25ms]
   │  ├─ external_api.fetch_user_avatar         [280ms]   ← THE SLOW PART
   │  └─ json.serialize                         [ 15ms]
   └─ frontend.hydrate                          [ 60ms]
```

You instantly see: 280 of the 340 ms spent in the backend was waiting on an external API call. _That's_ the bottleneck. No amount of CPU/RAM metrics would have told you that — only the trace did.
### Vocabulary

|Term|Meaning|
|---|---|
|**Trace**|The full record of a single request, end to end|
|**Span**|One operation within a trace (a DB query, an HTTP call, a function call)|
|**Trace ID**|A unique ID generated at the entry point and propagated through every span|
|**Span ID**|Each span has its own ID; spans link to their parent span|
|**Context propagation**|Passing the trace ID across service boundaries (usually HTTP headers)|
### How tracing works

Each service in your system uses an **instrumentation library** that:

1. Generates a Trace ID for incoming requests (or reads it from a header if upstream already has one).
2. Records each operation as a span: function name, start time, duration, attributes.
3. Sends spans to a collector.
4. Propagates the trace context downstream via HTTP headers (`traceparent`, `b3-trace-id`).

Without context propagation, you'd get separate disconnected traces in each service. With it, you get one continuous trace through the whole stack.
### The OpenTelemetry standard

**OpenTelemetry (OTel)** is a CNCF project — the vendor-neutral standard for instrumentation. Instead of writing code that's locked to Datadog or Jaeger, you write code that emits OTel spans, then point them at any backend.

```
   Your code
     ↓
   OpenTelemetry SDK (Node.js, Python, Go, Java...)
     ↓
   OTel Collector
     ↓                  ↓                ↓
   Jaeger            Tempo            Datadog
```

Switching backends becomes a config change, not a code rewrite. Use OTel from day one.
### Common tracing backends

|Backend|Vendor|Notes|
|---|---|---|
|**Jaeger**|CNCF|Open-source. Originally from Uber.|
|**Tempo**|Grafana Labs|Open-source. Cheaper storage. Integrates with Grafana.|
|**Zipkin**|Open Source / Twitter|Older but still in use.|
|**Datadog APM**|Datadog|Commercial. Plug-and-play.|
|**New Relic**|New Relic|Commercial. Strong UI.|
### Quick example — instrumenting an Express app

```javascript
// server/tracing.js
import { NodeSDK } from '@opentelemetry/sdk-node';
import { OTLPTraceExporter } from '@opentelemetry/exporter-trace-otlp-http';
import { getNodeAutoInstrumentations } from '@opentelemetry/auto-instrumentations-node';

const sdk = new NodeSDK({
  traceExporter: new OTLPTraceExporter({
    url: 'http://otel-collector:4318/v1/traces'
  }),
  instrumentations: [getNodeAutoInstrumentations()],
});

sdk.start();
```

Just `import './tracing.js'` at the very top of your entrypoint. Auto-instrumentations capture HTTP, Express, Mongo/Postgres, fetch — all without touching business code. Spans flow to the OTel Collector, which forwards them to Jaeger/Tempo/Datadog.

> ⚙️ **Added Professional Context — what "APM" means:** **APM (Application Performance Monitoring)** is the broader category — the umbrella over tracing, error tracking, transaction monitoring, and code-level profiling. Datadog APM, New Relic, AppDynamics, Dynatrace are full APM products. Jaeger / Tempo / Sublyzer are narrower tools that fit inside the APM space.

> 🔁 **Recap (B.6 — Tracing):** Traces show the path of one request through your system, with timings per span. OpenTelemetry is the vendor-neutral standard. Jaeger / Tempo / Datadog APM are common backends. Tracing reveals problems metrics and logs can't — like "this slow API is slow because of an external dependency." The third pillar of observability.

---
## B.7 — Datadog & Splunk — the commercial alternatives
### Why these tools exist

Everything we built in Section 4 (Prometheus + Grafana + Loki + Jaeger) is open-source. It works. But it requires:

- **Operational overhead** — you run, scale, back up, secure those services yourself.
- **Engineer time** — wiring exporters, dashboards, alert rules.
- **Specialist knowledge** — PromQL, Loki LogQL, OTel.

For a 5-person startup with one DevOps engineer, that's months of work that _isn't_ shipping product. The commercial alternative: pay a SaaS vendor to do all of it.

The two giants are **Datadog** and **Splunk**. Different histories, similar present-day positioning.
### Datadog — the modern unified platform

Datadog provides metrics + logs + traces + RUM (Real User Monitoring) + synthetic checks + security + CI visibility — _all in one product, one UI, one bill_.

How it works:

1. You install the **Datadog Agent** on every server / Kubernetes node.
2. The agent auto-detects what's running (Docker? Mongo? Postgres? Nginx?) and starts collecting metrics, logs, and traces from each.
3. Data goes to Datadog's cloud.
4. You get pre-built dashboards for nearly every popular service.

```
   Your servers           Datadog Agent          Datadog Cloud
   ─────────────          ──────────────         ─────────────
   App + Mongo +     ───→ One process per   ───→ Metrics, Logs,
   Nginx + Redis          host, auto-          Traces, Alerts
                          detects services     all unified
```

Properties:

- **Auto-instrumentation.** Drop the agent in; everything starts working.
- **400+ integrations.** Anything popular is supported out of the box.
- **APM, RUM, synthetic monitoring** all in one product.
- **Expensive at scale.** $15-$30 per host per month, plus ingest fees for logs. A 100-host team can pay $50k/year easily.

Use Datadog if you can pay for it and want to skip months of operational work.
### Splunk — the log-search behemoth

Splunk started as a log-management product 20 years ago and remains dominant in _enterprise_ and _security_ contexts (banks, healthcare, gov).

How it works:

1. **Splunk Forwarders** ship logs from every server.
2. **Splunk Indexers** index and store them.
3. **Splunk Search Head** — query UI with the powerful **SPL** (Search Processing Language).
4. Users query, build dashboards, set alerts.

What it's especially good at:

- **Massive log volumes** with full-text search.
- **Security analytics** (Splunk Enterprise Security is a major SIEM product).
- **Compliance** (banks and regulated industries trust Splunk's audit trail).
- **Complex SPL queries** that perform multi-stage transformations.

Properties:

- **Mature.** 20+ years of features. Probably has the answer to your weird requirement.
- **Expensive.** Pricing is volume-based; gigs of logs add up fast.
- **Steep learning curve.** SPL is powerful but not gentle.
- **Less unified than Datadog.** Logs are first-class; metrics/traces are a newer add-on (Splunk Observability Cloud, formerly SignalFx).
### Quick comparison

| Datadog        | Splunk                    | Open-source DIY (Prometheus + Loki + Jaeger) |                                    |
| -------------- | ------------------------- | -------------------------------------------- | ---------------------------------- |
| Best for       | Modern cloud / Kubernetes | Enterprise / Security / Compliance           | Cost-sensitive teams, full control |
| Onboarding     | Hours                     | Days                                         | Weeks                              |
| Cost           | High                      | Highest                                      | Just infrastructure cost           |
| Vendor lock-in | Significant               | Significant                                  | None                               |
| Customization  | Moderate                  | Very high (SPL)                              | Total                              |
### A practical recommendation

|Your situation|Pick|
|---|---|
|Small team, tight budget, technical skills|Prometheus + Grafana + Loki + Jaeger (DIY)|
|Growing startup, wants to ship fast, has budget|Datadog|
|Enterprise, heavy compliance, security-driven|Splunk + maybe Datadog for app metrics|
|Already uses one of them well|Don't switch lightly|

Most teams use **a hybrid** — open-source for some pillars, commercial for others. Common pattern: Prometheus for metrics (cheap, easy, in-cluster), Datadog or Splunk for logs (where commercial features pay off).

> 🔁 **Recap (B.7 — Datadog/Splunk):** Datadog = modern unified observability SaaS, plug-and-play but expensive. Splunk = enterprise log-search king, dominant in regulated industries. Both pay you back in operational time but cost real money. Open-source DIY is cheaper but takes effort. Hybrid setups are common.

---
## B.8 — AWS Networking Deep (VPC, Subnets, NACL, SG, NAT, Bastion)

We touched on Security Groups in Section 1.9. Real AWS networking has a lot more — and these are exactly the topics your senior listed (VPC, Subnets, Route Tables, NACLs, NAT Gateway, public/private subnets, bastion). Mastering them separates a junior who "deployed once on EC2" from a real cloud engineer.
### The big picture

```
   ┌──────────────────────────────────────────────────────────────────┐
   │                          AWS REGION                              │
   │                       (e.g. us-east-1)                           │
   │                                                                  │
   │   ┌──────────────────────────────────────────────────────────┐   │
   │   │                       VPC (10.0.0.0/16)                  │   │
   │   │           Your private network in AWS, completely        │   │
   │   │           isolated from other AWS customers.             │   │
   │   │                                                          │   │
   │   │   ┌─────────────── Availability Zone us-east-1a ─────┐   │   │
   │   │   │                                                  │   │   │
   │   │   │  ┌────── PUBLIC SUBNET (10.0.1.0/24) ────────┐   │   │   │
   │   │   │  │  • Has a route to Internet Gateway        │   │   │   │
   │   │   │  │  • Resources here CAN have public IPs     │   │   │   │
   │   │   │  │  ┌──────────┐    ┌──────────┐             │   │   │   │
   │   │   │  │  │ ALB / LB │    │  Bastion │             │   │   │   │
   │   │   │  │  └──────────┘    └──────────┘             │   │   │   │
   │   │   │  └───────────────────────────────────────────┘   │   │   │
   │   │   │                                                  │   │   │
   │   │   │  ┌────── PRIVATE SUBNET (10.0.2.0/24) ───────┐   │   │   │
   │   │   │  │  • NO route to IGW                        │   │   │   │
   │   │   │  │  • Outbound via NAT Gateway only          │   │   │   │
   │   │   │  │  ┌──────────┐    ┌──────────┐             │   │   │   │
   │   │   │  │  │   App    │    │ Database │             │   │   │   │
   │   │   │  │  │  Server  │    │   (RDS)  │             │   │   │   │
   │   │   │  │  └──────────┘    └──────────┘             │   │   │   │
   │   │   │  └───────────────────────────────────────────┘   │   │   │
   │   │   └──────────────────────────────────────────────────┘   │   │
   │   │                                                          │   │
   │   │   (similar AZ us-east-1b, us-east-1c with their own      │   │
   │   │    public + private subnets, for high availability)      │   │
   │   │                                                          │   │
   │   └──────────────────────────────────────────────────────────┘   │
   │                                                                  │
   │              ▲                                  ▲                │
   │              │                                  │                │
   │   ┌──────────────────┐              ┌────────────────────┐       │
   │   │ Internet Gateway │              │   NAT Gateway      │       │
   │   │   (IGW)          │              │ (in public subnet) │       │
   │   └──────────────────┘              └────────────────────┘       │
   │              │                                  │                │
   └──────────────┼──────────────────────────────────┼────────────────┘
                  │                                  │
                  ▼                                  ▼
              Internet                         Internet (egress only)
```
### VPC — Virtual Private Cloud

A **VPC** is your private, isolated network in AWS. Every AWS account starts with a "default VPC" in each region; in production you create your own with explicit settings.

A VPC is defined by a **CIDR block** — a range of private IP addresses. Common choices:

|CIDR|IP count|Use|
|---|---|---|
|`10.0.0.0/16`|65,536|Very common; lots of room|
|`10.0.0.0/8`|16 million|Massive enterprise VPC|
|`172.16.0.0/12`|1 million|Default RFC 1918 alternative|
|`192.168.0.0/16`|65,536|Familiar from home routers|

You then carve the VPC into subnets. Pick a `/16` and you have plenty of room.
### Subnets — slices of a VPC

A **subnet** is a slice of the VPC's CIDR, scoped to **one Availability Zone (AZ)**. AZs are physical data centers within a region (us-east-1a, us-east-1b, us-east-1c).

Subnets divide into two types based on routing:

|Type|Has route to Internet Gateway?|Hosts get public IP?|Examples|
|---|---|---|---|
|**Public**|Yes|Optional (auto-assign)|Load balancers, bastions|
|**Private**|No|No|App servers, databases|

The "public" vs "private" distinction is **not** about the IP range — both use private CIDR blocks. The difference is the **Route Table** attached to the subnet.
### Route Tables

A **route table** tells AWS where to send traffic. Each subnet has exactly one route table.

A typical **public subnet** route table:

|Destination|Target|Meaning|
|---|---|---|
|`10.0.0.0/16`|local|Traffic within the VPC stays in VPC|
|`0.0.0.0/0`|igw-abc123|Everything else → Internet Gateway|

A typical **private subnet** route table:

|Destination|Target|Meaning|
|---|---|---|
|`10.0.0.0/16`|local|Traffic within the VPC stays in VPC|
|`0.0.0.0/0`|nat-abc123|Everything else → NAT Gateway|

Note: the private subnet has _no route to the IGW_. That's literally what makes it private.
### Internet Gateway (IGW)

An **Internet Gateway** is a VPC component that connects the VPC to the public internet. One per VPC. Resources in subnets routed to the IGW can:

- Receive incoming traffic from the internet (if they have public IPs).
- Send outgoing traffic to the internet.
### NAT Gateway

A **NAT Gateway** lets resources in **private subnets** make outbound connections to the internet (e.g., to download package updates, hit external APIs) **without** being directly reachable from the internet.

```
   Private app server
     │  outbound request: GET https://npm.org
     ▼
   NAT Gateway (in public subnet, has a public IP)
     │  rewrites source IP to NAT's public IP
     ▼
   Internet Gateway
     ▼
   npmjs.org
```

When the response comes back:

```
   npmjs.org responds
     ▼
   Internet Gateway
     ▼
   NAT Gateway (knows this response belongs to the app server)
     │  rewrites destination back to private IP
     ▼
   Private app server
```

The app server gets the response without ever exposing itself to inbound internet traffic. That's the asymmetric "outbound but not inbound" property.

> ⚠️ **NAT Gateway is expensive.** ~$0.045/hour + data transfer fees → ~$32/month minimum, plus per-GB egress. For tiny dev environments, use a "NAT Instance" (a small EC2 doing the same job, ~$5/month) or skip the private-subnet pattern entirely.
### Security Groups (SG) — stateful, instance-level firewalls

A **Security Group** is a virtual firewall attached to an EC2 instance (or RDS, ELB, etc.).

Properties:

- **Stateful.** If you allow inbound traffic from an IP, the response goes back automatically (no need for an explicit outbound rule).
- **Default deny.** Nothing is allowed except what you explicitly open.
- **Source can be a CIDR or another SG.** This is powerful — "allow inbound from any instance in `web-sg`" is more flexible than IPs.

Example for a backend instance:

|Type|Port|Source|Purpose|
|---|---|---|---|
|Custom TCP|5000|sg-frontend|Frontend SG can talk to backend|
|SSH|22|sg-bastion|Only the bastion can SSH in|
|(Outbound: by default, allow all)||||

Compare to the typical SG we wrote in Section 1.9, which used `0.0.0.0/0` for everything — that's fine for a demo on a single EC2; in production you scope sources tightly.
### Network ACL (NACL) — stateless, subnet-level firewalls

A **NACL** is a second layer of firewalling at the **subnet** level (one NACL per subnet).

Properties:

- **Stateless.** If you allow inbound on port 80, you must _also_ allow outbound on the ephemeral return ports (1024-65535). Otherwise responses are blocked.
- **Default-allow** in the default NACL; default-deny if you create a custom one.
- **Numbered rules** evaluated in order (lower numbers first).
- Rules can explicitly **deny**, unlike SGs.

When to use:

|Need|Use|
|---|---|
|Allow specific apps/instances to talk to each other|SG|
|Block a known-bad IP range from your whole subnet|NACL|
|Compliance requires "explicit deny rules" auditable per subnet|NACL|

In practice, **most teams rely almost entirely on SGs**. NACLs are used as a defense-in-depth supplement, not the primary tool.
### SG vs NACL — the comparison table

| Security Group | NACL               |                                  |
| -------------- | ------------------ | -------------------------------- |
| Scope          | Instance / ENI     | Subnet                           |
| Stateful       | Yes                | No (must define both directions) |
| Default        | Deny all           | (Default NACL): allow all        |
| Rules          | Allow only         | Allow + Deny                     |
| Evaluation     | All rules combined | Numbered, first match wins       |
| Source         | CIDR or SG         | CIDR only                        |
### Bastion host — the secure SSH entry point

In the diagram above, app servers live in the **private subnet** — they have no public IP and aren't reachable from the internet. So how do you SSH in?

You don't, directly. You go through a **bastion host** (also called "jump box"):

```
   You ─────────SSH─────────→ Bastion (public subnet, public IP)
                                  │
                                  └──SSH──→ App Server (private subnet)
```

The bastion is a small EC2 in the public subnet. Hardened (only port 22 from your office IP), MFA-enabled, audited. From your laptop, you SSH to the bastion. From the bastion, you SSH onward to the private app servers.

Properties:

- **Single audit point.** All admin access funnels through one host. Logs are centralized.
- **Tiny attack surface.** The bastion runs nothing else — no app, no DB.
- **Easy to harden.** Put it on a per-developer SG; rotate keys; require MFA.
#### SSH agent forwarding — the trick that makes bastions usable

Without help, you'd need to copy your SSH key onto the bastion to SSH onward. That defeats the security benefit (key on a server you don't fully control).

The fix: **SSH agent forwarding** (`-A`). Your laptop's SSH agent holds the key; the bastion borrows it without ever seeing the key file.

```bash
# One-time: load your key into the agent
ssh-add ~/.ssh/aws-prod-key.pem

# Connect to the bastion with agent forwarding
ssh -A ec2-user@bastion.example.com

# From the bastion, hop onward — key still works
ssh ec2-user@10.0.2.50
```

Even better: use **SSH ProxyJump** to do both hops in one command:

```bash
ssh -J ec2-user@bastion.example.com ec2-user@10.0.2.50
```
#### Modern alternative — AWS Systems Manager Session Manager

AWS now provides **SSM Session Manager**, which gives you a shell on a private EC2 _without_ SSH or a bastion at all. The instance has the SSM agent (default on modern AMIs), and you authenticate via IAM. No port 22 open anywhere. This is increasingly the production standard.

```bash
aws ssm start-session --target i-0123456789abcdef
```
### A complete production VPC design (3 AZ, HA)

A typical production setup spans 3 AZs for fault tolerance:

```
   VPC 10.0.0.0/16

   us-east-1a:    public 10.0.1.0/24 + private 10.0.11.0/24
   us-east-1b:    public 10.0.2.0/24 + private 10.0.12.0/24
   us-east-1c:    public 10.0.3.0/24 + private 10.0.13.0/24

   • One Application Load Balancer spans all 3 public subnets
   • App auto-scaling group spans all 3 private subnets
   • RDS Multi-AZ deployed in private subnets
   • NAT Gateway in each public subnet (one per AZ for HA)
   • Bastion in public subnet (or use SSM)
```

Lose an entire AZ? The other two still serve traffic. This is what real teams build, and Terraform modules like `terraform-aws-modules/vpc/aws` produce exactly this layout from a few lines of HCL.

> 🔁 **Recap (B.8 — AWS Networking):** VPC = private network. Subnets = AZ-scoped slices, public if routed to IGW, private if routed to NAT. Route tables decide where traffic goes. Security Groups = stateful, instance-level. NACLs = stateless, subnet-level. Bastion or SSM Session Manager for admin access into private subnets. Production designs span 3 AZs for HA.

---
## B.9 — Cloud Comparison (AWS vs GCP vs Azure)
### Why this matters

Every team eventually picks a cloud. The picks differ by industry: startups gravitate toward AWS, data-heavy companies toward GCP, enterprises with Microsoft footprints toward Azure. As a DevOps engineer, you'll likely touch all three over a career — the _concepts_ port directly between them; the _names_ don't.
### Conceptual mapping

|Concept|AWS|GCP|Azure|
|---|---|---|---|
|Virtual machine|EC2|Compute Engine (GCE)|Virtual Machine (VM)|
|Object storage|S3|Cloud Storage (GCS)|Blob Storage|
|Block storage|EBS|Persistent Disk|Managed Disks|
|Managed Kubernetes|EKS|GKE|AKS|
|Serverless functions|Lambda|Cloud Functions|Azure Functions|
|Serverless containers|ECS / Fargate|Cloud Run|Container Apps|
|Container registry|ECR|Artifact Registry|ACR|
|Managed Postgres|RDS for PostgreSQL|Cloud SQL for PostgreSQL|Azure Database for PostgreSQL|
|Managed key-value|DynamoDB|Firestore / Bigtable|Cosmos DB|
|Object queue|SQS|Pub/Sub|Service Bus|
|Pub/sub messaging|SNS|Pub/Sub|Event Grid|
|Load balancer (L7)|ALB|Cloud Load Balancing|Application Gateway|
|Load balancer (L4)|NLB|TCP/UDP Load Balancing|Load Balancer|
|Private network|VPC|VPC|Virtual Network (VNet)|
|Identity / IAM|IAM|Cloud IAM|Azure AD + RBAC|
|Free TLS certs|Certificate Manager (ACM)|Certificate Authority Service|Azure Key Vault|
|Secrets manager|Secrets Manager / Parameter Store|Secret Manager|Key Vault|
|Monitoring metrics|CloudWatch|Cloud Monitoring|Azure Monitor|
|Logging|CloudWatch Logs|Cloud Logging|Azure Monitor Logs|
|CDN|CloudFront|Cloud CDN|Azure CDN / Front Door|
|DNS|Route 53|Cloud DNS|Azure DNS|

If you can use AWS effectively, you can pick up the GCP or Azure equivalent in days.
### Strengths and stereotypes

|Cloud|Stereotype|Strengths|
|---|---|---|
|**AWS**|The default; widest ecosystem; everything is on it|Maturity, services breadth, market share|
|**GCP**|Google-native; deep on K8s and data; calmer pricing|GKE (the original K8s), BigQuery, ML/AI tooling|
|**Azure**|Microsoft house; AD/Office integration; enterprise-friendly|.NET, Office 365 sync, government/regulated workloads|
### How to choose (practically)

You usually don't choose. The company already has a cloud. If you're starting fresh:

1. **Default to AWS.** Most jobs, most resources, most tutorials, biggest marketplace.
2. **Choose GCP** if your work is data-heavy or K8s-heavy.
3. **Choose Azure** if your company is Microsoft-aligned.

Multi-cloud is a real cost and operational burden — most teams pick one and stay.

> 🔁 **Recap (B.9 — Clouds):** Three giants, near-equivalent service catalogs with different names. EC2 ↔ GCE ↔ VM. S3 ↔ GCS ↔ Blob. EKS ↔ GKE ↔ AKS. Knowing one ports to the others.

---
## B.10 — Security Essentials

A complete DevOps engineer doesn't ship "it works" — they ship "it works _and_ it's safe." A short tour of the security topics every DevOps practitioner needs.
### The principle of least privilege (PoLP)

Every actor — humans, services, scripts — should have **only the permissions they need to do their job, and nothing more**.

Examples:

- Your CI/CD service account shouldn't have Admin access to all of AWS — just the resources it deploys.
- Your app's database user shouldn't have `DROP TABLE` rights — just `SELECT`/`INSERT` on its tables.
- A pod that doesn't write files shouldn't have a writable filesystem.

PoLP shrinks the blast radius of any compromise. Stolen CI token? It can only deploy, not delete S3 buckets.
### Secrets management

**Bad:** secrets in source code, in `.env` committed to Git, in plaintext anywhere.

**Better:** secrets in a vault.

|Tool|Where it lives|Notes|
|---|---|---|
|**AWS Secrets Manager**|AWS|Auto-rotates RDS / Aurora passwords|
|**AWS Parameter Store**|AWS|Cheaper, simpler, also encrypts|
|**HashiCorp Vault**|Anywhere (self-hosted)|The gold-standard secret store|
|**GCP Secret Manager**|GCP|GCP-native equivalent|
|**Azure Key Vault**|Azure|Azure-native equivalent|
|**Sealed Secrets**|Kubernetes|Encrypted Secrets that are safe to commit to Git|
|**External Secrets Operator**|Kubernetes|Pulls secrets from Vault/Secrets Manager into K8s|
|**GitHub Actions Secrets**|Within GitHub|Fine for CI; not for app runtime|

**Rotate regularly.** A leaked secret you never rotate is a leaked secret forever. Auto-rotation (Secrets Manager + RDS) eliminates this risk.
### Image and dependency scanning

Every Docker image and every npm/pip package you pull in is potential attack surface. CVEs (Common Vulnerabilities and Exposures) appear daily.

|Tool|What it scans|
|---|---|
|**Trivy**|Docker images, file systems, Git repos|
|**Snyk**|Open-source dependencies, container images|
|**Grype**|Container images (similar to Trivy)|
|**Dependabot**|GitHub-native dependency-update bot|
|**npm audit / pip-audit**|Native package-manager scans|

CI integration:

```yaml
- name: Scan image with Trivy
  uses: aquasecurity/trivy-action@master
  with:
    image-ref: 'ankitsangwan/mern-backend:${{ github.sha }}'
    severity: 'CRITICAL,HIGH'
    exit-code: '1'              # fail the build on critical/high vulns
```
### Network security — the layered approach

```
   Edge:        DDoS protection (CloudFront, Cloudflare)
   Gateway:     WAF (Web Application Firewall) — blocks SQL injection, XSS
   Network:     VPC + SGs + NACLs (Section B.8)
   Transport:   TLS / HTTPS everywhere
   Auth:        OAuth, JWT, mTLS for service-to-service
   Endpoint:    OS hardening, fail2ban, security updates
```

**Defense in depth.** No single control should be your only line. Each layer catches what the others miss.
### Kubernetes RBAC

K8s has a powerful Role-Based Access Control system. Roles map permissions ("can read pods"); RoleBindings link users/service accounts to roles.

```yaml
# A pod-reader Role
apiVersion: rbac.authorization.k8s.io/v1
kind: Role
metadata:
  namespace: production
  name: pod-reader
rules:
  - apiGroups: [""]
    resources: ["pods"]
    verbs: ["get", "list", "watch"]
---
# Bind the role to a user
apiVersion: rbac.authorization.k8s.io/v1
kind: RoleBinding
metadata:
  name: jane-can-read-pods
  namespace: production
subjects:
  - kind: User
    name: jane@example.com
roleRef:
  kind: Role
  name: pod-reader
  apiGroup: rbac.authorization.k8s.io
```

Don't grant `cluster-admin` to humans except in emergencies. Ever.
### Pod security and image hardening

Inside containers:

- **Run as non-root.** Add a `USER` line to your Dockerfile (Section 2.4 example).
- **Read-only root filesystem** in the Pod spec:
    
    ```yaml
    securityContext:  readOnlyRootFilesystem: true  runAsNonRoot: true  runAsUser: 1000
    ```
    
- **Drop unnecessary Linux capabilities:**
    
    ```yaml
    securityContext:  capabilities:    drop: ["ALL"]
    ```
    
- **Use minimal base images.** `node:18-alpine` (~50 MB) instead of `node:18` (~900 MB) — fewer libraries, fewer CVEs.
- **Distroless images** (Google's `gcr.io/distroless/...`) — no shell, no package manager, just the runtime. Hardest to attack.
### Compliance frameworks (which you'll meet on the job)

|Framework|Industry / Region|
|---|---|
|**SOC 2**|SaaS / B2B (US-led, very common)|
|**ISO 27001**|International / European|
|**HIPAA**|US healthcare|
|**PCI-DSS**|Payment cards (anyone handling card data)|
|**GDPR**|EU privacy regulation|
|**FedRAMP**|US federal contracts|

Each demands specific controls — audit logs, access reviews, encryption at rest, retention policies. As a DevOps engineer you implement what auditors require.

> 🔁 **Recap (B.10 — Security):** Least privilege everywhere. Secrets in vaults, never in Git. Scan images and dependencies in CI. Defense in depth — multiple network layers. K8s RBAC for human access; non-root containers for app safety. Compliance is a real engineering deliverable.

---
## B.11 — Master Interview Question Bank

A consolidated list of the questions you will actually be asked in DevOps interviews, organized by topic. Each one is answered above somewhere in the guide; this is the index for review.
### Linux & Networking

1. _"What's the difference between hard and soft links?"_ — Hard links share an inode; soft (symbolic) links point to a path.
2. _"How do you find the process listening on port 8080?"_ → `lsof -i :8080` or `netstat -tlnp | grep 8080` or `ss -tlnp`.
3. _"What does `chmod 600 ~/.ssh/id_ed25519` do and why is it needed?"_ → Owner-only read/write; SSH refuses to use keys with looser permissions.
4. _"Explain SSH key-based authentication."_ → Section 1.1.5.
5. _"What is `umask`?"_ → Default permission mask subtracted when files are created.
6. _"Difference between `kill` and `kill -9`?"_ → SIGTERM (graceful) vs SIGKILL (force).
7. _"What's CIDR notation? Explain `0.0.0.0/0`, `/32`, `/24`."_ → Section 1.9.
8. _"How does DNS resolution work?"_ → Recursive resolver → root → TLD → authoritative server → record.
### Git

9. _"`git fetch` vs `git pull`?"_ → fetch downloads only; pull = fetch + merge.
10. _"What's a fast-forward merge?"_ → No divergent commits; just moves the branch pointer forward.
11. _"How do you undo the last commit but keep changes?"_ → `git reset --soft HEAD~1`.
12. _"What does `.gitignore` not do?"_ → It doesn't untrack files already committed; you must `git rm --cached`.
13. _"Branch strategies — Git Flow vs GitHub Flow vs trunk-based?"_ → Section 1.3.
### Docker

14. _"What's the difference between a Docker image and a container?"_ → Image = template; container = running instance.
15. _"VM vs Container?"_ → Section 1.5.
16. _"Most common Dockerfile optimization?"_ → Layer ordering: copy `package.json` and install deps before copying source.
17. _"`CMD` vs `ENTRYPOINT`?"_ → ENTRYPOINT is the executable; CMD provides default args. Together: `ENTRYPOINT ["node"]` + `CMD ["index.js"]` runs `node index.js`.
18. _"Why use named volumes over bind mounts in production?"_ → Portable, permission-friendly, manageable via Docker tooling.
19. _"What does `EXPOSE` do?"_ → Pure metadata. The actual port-publishing is `-p`.
20. _"Why does my backend fail to connect to `mongodb://localhost:27017`?"_ → The localhost trap (Section 2.2). Use service name `mongodb://mongo:27017`.
21. _"What is `/var/run/docker.sock`?"_ → The Unix socket Docker daemon listens on. Mounting it gives a container CLI access to the host's daemon.
22. _"What's the danger of `docker compose down -v`?"_ → Deletes named volumes including DB data.
23. _"Multi-stage build benefits?"_ → Smaller final image; build tools not present in runtime image.
### CI/CD

24. _"`npm install` vs `npm ci` in CI?"_ → Section 1.8 — `ci` is reproducible, fast, lockfile-strict.
25. _"What is branch protection?"_ → GitHub setting that blocks merge until status checks pass.
26. _"Difference between Continuous Delivery and Continuous Deployment?"_ → Delivery = ready to deploy with a button; Deployment = automatic deploy.
27. _"How do you keep secrets out of CI logs?"_ → GitHub Secrets / Vault / `withCredentials` in Jenkins; the platforms auto-mask known secrets.
28. _"What's the difference between push-based deploy and GitOps pull-based deploy?"_ → Section B.3.
29. _"Why does Jenkins still exist if GitHub Actions exists?"_ → Self-hosted, regulated, air-gapped environments.
30. _"What's the Jenkins socket trick?"_ → Mount `/var/run/docker.sock` so Jenkins-in-a-container can build images on the host.
### Kubernetes

31. _"What's a Pod?"_ → 1+ tightly coupled containers sharing network and storage.
32. _"Imperative vs declarative — illustrate with a K8s example."_ → Section 3.3.
33. _"Readiness probe vs liveness probe?"_ → Readiness controls traffic routing; liveness restarts the pod.
34. _"How does a rolling update work in K8s?"_ → Section 3.3 walkthrough.
35. _"What are Service types?"_ → ClusterIP, NodePort, LoadBalancer, ExternalName.
36. _"What does Ingress do that Services don't?"_ → HTTP-level routing (host + path), shared LB, central HTTPS termination.
37. _"ConfigMap vs Secret?"_ → Both inject config; Secret is base64-encoded and RBAC-protected.
38. _"Why are Secrets not encrypted by default?"_ → They're base64 (encoding, not encryption). RBAC + etcd encryption-at-rest are the protections.
39. _"Deployment vs StatefulSet?"_ → Deployment = stateless interchangeable; StatefulSet = stable identity per pod, per-pod PVC.
40. _"How does a PVC find its PV?"_ → Dynamic provisioning via StorageClass.
41. _"What is RBAC?"_ → Roles + RoleBindings — Section B.10.
### Terraform / Ansible

42. _"Provisioning vs configuration management — when do you use which?"_ → Section 4.3.
43. _"What does `terraform.tfstate` store and why is it important?"_ → Mapping of HCL names ↔ real cloud resource IDs. Single source of truth for diffs.
44. _"Why is local state bad for a team?"_ → Split brain + security.
45. _"What's a remote backend?"_ → Shared state in S3 + DynamoDB lock or HCP Terraform.
46. _"How do you make a Terraform variable required vs optional?"_ → No default → required; default → optional.
47. _"What does `terraform import` do?"_ → Brings an existing cloud resource into Terraform state without recreating it.
48. _"Why is Ansible agentless an advantage?"_ → No agent install, no agent version skew, just SSH.
49. _"What is idempotency in Ansible?"_ → Re-running yields the same end state without harm.
50. _"How do you avoid hardcoded secrets in Ansible?"_ → `ansible-vault encrypt_string`.
### Observability

51. _"3 pillars of observability?"_ → Metrics, Logs, Traces. (+ Alerting as cross-cutting.)
52. _"Why p99 instead of average?"_ → Section B.1 — the tail is what users feel.
53. _"What's the difference between a metric and a log and a trace?"_ → Numeric over time / text events / per-request path.
54. _"White box vs black box monitoring?"_ → Section 4.7.
55. _"Why does Prometheus pull instead of push?"_ → Targets advertise their state; the central system reaches out — simpler service discovery, no firewall holes inbound.
56. _"What does Grafana store?"_ → Dashboards and config; **not metrics**.
57. _"Walk me through what happens when production goes down at 3 AM."_ → Section B.2 — full alerting chain.
58. _"What is an SLO and how does it relate to alerting?"_ → A target like "p99 < 500 ms"; alerts fire when it's at risk.
### Cloud / AWS

59. _"VPC, subnet, route table, IGW, NAT — explain each."_ → Section B.8.
60. _"Public vs private subnet?"_ → Routed to IGW vs routed to NAT.
61. _"Security Group vs NACL?"_ → Section B.8 table.
62. _"Why use a bastion?"_ → Single audit/entry point for SSH into private subnets.
63. _"What's SSM Session Manager and why might it replace bastions?"_ → Authenticates via IAM, no port 22.
64. _"What's Multi-AZ in RDS?"_ → Synchronous standby in a different AZ; automatic failover.
65. _"What's the difference between an ALB and an NLB?"_ → ALB = L7 (HTTP); NLB = L4 (TCP/UDP, very fast).
66. _"What's IaC and why does it matter?"_ → Section 4.3.
### Behavioral / Scenario

67. _"Tell me about a production incident you handled."_ — Have a story ready: detection → triage → fix → postmortem.
68. _"How do you approach a service that's suddenly running slow?"_ → Check Grafana dashboards (CPU/RAM), check logs for errors, check traces for slow spans, check downstream dependencies.
69. _"Your build is suddenly failing. How do you debug?"_ → Read CI logs for the first error; check if it's flaky or deterministic; bisect via Git.
70. _"How do you decide between a managed service and self-hosting?"_ → Cost + ops time + control trade-off; default to managed unless control is essential.

---
## B.12 — Master Command Cheat-Sheet

A single, dense reference card for the commands that matter most. Print this; tape it to your desk.
### Linux

```bash
# Files / nav
pwd                                  # where am I
ls -lah                              # list with sizes & hidden
cd ~                                 # go home
cd -                                 # previous directory
cp -r src dst                        # recursive copy
mv old new                           # rename / move
rm -rf dir                           # recursive force delete (CAREFUL)
find . -name "*.log"                 # find files
grep -r "TODO" .                     # recursive grep
chmod 600 file.pem                   # owner r/w only
chown -R user:group dir              # change owner recursively
df -h                                # disk usage per mount
du -sh /var/log                      # size of a directory
free -h                              # memory usage
top / htop                           # CPU/memory live
ps -ef | grep node                   # find process
kill -9 12345                        # force kill PID
systemctl status nginx               # service status
systemctl restart nginx              # restart service
journalctl -u nginx -f               # follow systemd logs

# Users / SSH
whoami / id / groups
sudo useradd -m -s /bin/bash deploy
sudo usermod -aG docker $USER
ssh-keygen -t ed25519 -a 64 -C "label"
ssh-copy-id user@host
ssh -i key.pem user@host
ssh -A user@bastion                  # agent forwarding
ssh -J user@bastion user@private     # ProxyJump
```
### Git

```bash
git status / git diff / git log --oneline --graph --all
git checkout -b feature/x
git add . / git commit -m "msg"
git push -u origin feature/x
git pull / git fetch
git stash / git stash pop
git reset --hard origin/main         # discard local changes
git revert <sha>                     # undo a merged commit safely
git rebase -i HEAD~5                 # interactive rebase
```
### Docker

```bash
docker build -t name:tag .
docker run -d --name n -p 8080:80 nginx
docker ps / docker ps -a
docker logs -f container
docker exec -it container sh
docker stop container && docker rm container
docker images / docker rmi image
docker volume ls / docker volume inspect v
docker network ls / docker network inspect n
docker system prune -af --volumes    # nuke unused (CAREFUL)
docker login / docker push name:tag
```
### Docker Compose

```bash
docker compose up -d --build
docker compose down                  # keep volumes
docker compose down -v               # delete volumes (CAREFUL)
docker compose logs -f service
docker compose exec service sh
docker compose restart service
docker compose pull && docker compose up -d
docker compose config                # validate
```
### Kubernetes

```bash
kubectl get pods -A
kubectl get pods -o wide
kubectl describe pod x
kubectl logs -f x                    # follow
kubectl logs x -c container          # specific container
kubectl exec -it x -- sh
kubectl apply -f file.yaml
kubectl apply -f dir/
kubectl delete -f file.yaml
kubectl scale deployment/x --replicas=5
kubectl rollout status deployment/x
kubectl rollout undo deployment/x
kubectl edit deployment/x
kubectl port-forward pod/x 5000:5000
kubectl get all -n namespace
kubectl explain pod.spec.containers
kubectl config use-context cluster
kubectl create secret generic n --from-literal=KEY=val
```
### Terraform

```bash
terraform init -upgrade
terraform fmt
terraform validate
terraform plan
terraform apply -auto-approve
terraform destroy
terraform output
terraform import resource.name id
terraform state list
terraform state show resource.name
terraform workspace new prod
TF_VAR_token=xxxx terraform apply
```
### Ansible

```bash
ansible --version
ansible prod -m ping
ansible-playbook bootstrap.yml
ansible-playbook deploy.yml --check  # dry run
ansible-playbook deploy.yml --tags=docker
ansible-vault encrypt_string 'value' --name 'db_pwd'
ansible-vault edit secrets.yml
ansible prod -b -m shell -a "uptime"
```
### Prometheus / Grafana / Alerting

```promql
up                                    # all targets up?
rate(http_requests_total[5m])         # req/s
histogram_quantile(0.99,
  sum(rate(http_request_duration_seconds_bucket[5m])) by (le))
node_memory_MemAvailable_bytes / node_memory_MemTotal_bytes
container_memory_usage_bytes{name="backend"}
probe_success{job="blackbox_http"}
probe_duration_seconds
```
### k6

```bash
docker run --rm -i \
  -e BASE_URL="https://example.com" \
  -v $PWD:/work -w /work \
  grafana/k6 run ops/tests/k6/script.js
```

---
# 🎓 Closing — How to Use This Guide

You now have a single document covering:

- ✅ **Part 0** — DevOps philosophy, 7 pillars, complete tool ecosystem
- ✅ **Section 1** — Linux, Shell, Git, Env Mgmt, Docker, Compose, CI, GitHub Actions + ESLint + Prettier, AWS EC2, CD, Kubernetes intro
- ✅ **Section 2** — Docker Volumes / Networking / Registry deep dives + the combined MERN example, then full Jenkins (intro, architecture, Docker setup, admin/plugins, Web UI, Freestyle, full Pipeline + MERN Jenkinsfile)
- ✅ **Section 3** — Container-to-container, NGINX with the Vite ARG/ENV deep dive, Kubernetes Declarative (Pods/Deployments/Services/probes/rolling updates), ConfigMaps + Secrets, Ingress + cert-manager, Storage (PV/PVC/StatefulSet)
- ✅ **Section 4** — Tools Overview architecture, Sublyzer, full Terraform (lifecycle/state/blocks/import), full Ansible (bootstrap/deploy/observability playbooks), Caddy with ACME flow, Uptime Kuma, full Prometheus + Grafana with all exporters, k6 load testing
- ✅ **Bonus** — p50/p95/p99 deep dive, full alerting flow (server down → on-call), ArgoCD/GitOps, Helm, ELK/EFK + Loki, APM/tracing + OpenTelemetry, Datadog & Splunk, AWS networking deep, cloud comparison, security essentials, 70-question interview bank, master cheat-sheet
### Recommended reading order on first pass

Top to bottom. The sections genuinely depend on each other — Docker is needed for Compose; Compose is needed for Jenkins-with-socket-mount; Volumes/Networks are needed for Kubernetes; CI is needed for CD; Prometheus is needed for Alerting.

— _End of Guide_ —