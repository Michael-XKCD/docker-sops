# docker-sops

A lightweight shell tool that combines **Docker Compose**, **Git sparse worktrees**, and **Mozilla SOPS** for clean, secure stack management.

It’s built for setups where:
- Each stack (like `librechat`, `nextcloud`, `mealie`) lives in its own subfolder of a shared `docker-compose` repo.
- Encrypted `.env` files live in a parallel `docker-compose-env` repo.
- You want to **deploy**, **update**, and **commit** stacks individually without juggling `.env` files in GUIs like Portainer.

---

## 🧩 Features

- 🔐 **SOPS decryption** on the fly — no plaintext `.env` left on disk  
- 🧱 **Sparse worktrees** — only the stack you need is checked out  
- 🧵 **Flat working folders** — everything appears under `/mnt/user/Docker/<stack>/`  
- 🪶 **Symlink mirroring** — edit and commit directly from that flat folder  
- 🚀 **Simple commands** — `docker-sops up`, `docker-sops logs`, `docker-sops edit-env`, etc.  
- 🛠️ **Bare repo model** — stores upstream Git repos once in `/mnt/user/Docker/.repos`

---

## 📦 Installation

### 1. Clone the repo
```bash
git clone https://github.com/Michael-XKCD/docker-sops.git
cd docker-sops

2. Install the binary

Copy or symlink it to your $PATH:

sudo cp docker-sops /usr/local/bin/
sudo chmod +x /usr/local/bin/docker-sops

3. Prepare directories

sudo mkdir -p /mnt/user/Docker
sudo chown "$USER":"$USER" /mnt/user/Docker

4. Verify dependencies

You’ll need:
	•	Docker with the Compose plugin
	•	Git 2.30+
	•	Mozilla SOPS
	•	GPG or age key for decryption

Test it:

docker-sops help


⸻

🔐 Installing SOPS

You’ll need SOPS on both your server and development machine to encrypt/decrypt .env files.

macOS (Homebrew)

brew install sops

Debian / Ubuntu

sudo apt update
sudo apt install -y sops

RHEL / CentOS / Fedora

sudo dnf install -y sops

Arch Linux

sudo pacman -S sops

Manual install (universal)

wget https://github.com/mozilla/sops/releases/latest/download/sops-$(uname -s | tr '[:upper:]' '[:lower:]')-$(uname -m)
sudo mv sops-* /usr/local/bin/sops
sudo chmod +x /usr/local/bin/sops

Test:

sops --version


⸻

🔐 Getting Started with SOPS

1. Generate a key

Option 1: GPG

gpg --full-generate-key

Option 2: age

mkdir -p ~/.config/sops/age
age-keygen -o ~/.config/sops/age/keys.txt


⸻

2. Configure SOPS

Create a file called .sops.yaml at the root of your docker-compose-env repo.

Example (GPG)

# .sops.yaml
creation_rules:
  - path_regex: '.*\.env\.sops$'
    encrypted_regex: '^(?!#).*'
    key_groups:
      - pgp:
          - FINGERPRINT_OF_YOUR_GPG_KEY

To find your key fingerprint:

gpg --list-keys

Example (age)

# .sops.yaml
creation_rules:
  - path_regex: '.*\.env\.sops$'
    encrypted_regex: '^(?!#).*'
    key_groups:
      - age:
          - age1examplekeyxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx


⸻

3. Encrypt an env file

Create and encrypt your stack’s .env:

cd docker-compose-env/librechat
sops --encrypt --in-place librechat.env.sops

4. Test decryption

sops -d librechat.env.sops | head


⸻

🚀 First Use

1. Initialize a stack

This links both repos, sets up sparse worktrees, and creates symlinks:

docker-sops init librechat \
  --compose-url https://github.com/<you>/docker-compose \
  --env-url https://github.com/<you>/docker-compose-env

2. Deploy

cd /mnt/user/Docker/librechat
docker-sops up


⸻

🧰 Common Commands

Task	Command
View logs	docker-sops logs
Stop stack	docker-sops down
Pull new images	docker-sops pull
Commit compose changes	docker-sops git commit -am "update"
Edit secrets	docker-sops edit-env
Commit secrets	docker-sops git-env commit -am "update env"
Push all	docker-sops full-push
Update all stacks	docker-sops full-pull

You can run all these commands without specifying the stack name if you’re inside /mnt/user/Docker/<stack>.

⸻

🧭 Directory Layout

/mnt/user/Docker/
├── .repos/
│   ├── docker-compose.git
│   └── docker-compose-env.git
├── .worktrees/
│   ├── compose-librechat/
│   └── env-librechat/
├── librechat/
│   ├── docker-compose.yml -> symlink into worktree
│   ├── ...
│   └── .docker-sops-links
└── librechat.env.sops -> symlink into env worktree


⸻

⚙️ Notes
	•	Each stack is isolated — docker-sops only decrypts and composes what you call.
	•	Temporary decrypted env files are securely deleted after each run.
	•	Public-safe — no plaintext secrets or key material ever logged or stored.
	•	Works great alongside Dozzle for log viewing and watch docker ps for quick stack monitoring.
