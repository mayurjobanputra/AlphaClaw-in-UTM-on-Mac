# Running AlphaClaw in a UTM Virtual Machine on macOS

> **Why a VM?** Running untrusted or semi-trusted software inside a full virtual machine gives you the strongest isolation from your host Mac. Nothing inside the VM can access your Mac's files, credentials, or network unless you explicitly allow it.

---

## Prerequisites

- macOS with Apple Silicon (M1/M2/M3/M4) or Intel
- An Ubuntu Server 24.04 ISO (we'll download this below)

---

## Step 1 — Install UTM

UTM is a free, open-source virtual machine app for macOS. Pick one method:

**Option A — Download directly:**

1. Go to [mac.getutm.app](https://mac.getutm.app)
2. Click **Download**
3. Open the downloaded `.dmg` and drag UTM to your Applications folder

**Option B — Homebrew:**

```bash
brew install --cask utm
```

**Option C — Mac App Store:**

UTM is also available on the [Mac App Store](https://apps.apple.com/us/app/utm-virtual-machines/id1538878817) (paid — supports the developer, but functionally identical to the free version).

Once installed, open UTM from your Applications folder to confirm it launches.

---

## Step 2 — Download Ubuntu Server ISO

- **Apple Silicon Macs (M1/M2/M3/M4):** Download the ARM64 (aarch64) ISO from [ubuntu.com/download/server/arm](https://ubuntu.com/download/server/arm)
- **Intel Macs:** Download the standard amd64 ISO from [ubuntu.com/download/server](https://ubuntu.com/download/server)

Save the `.iso` file somewhere you can find it (e.g. your Downloads folder).

---

## Step 3 — Create the VM in UTM

1. Open UTM and click **Create a New Virtual Machine**
2. Select **Virtualize** (not Emulate — this gives near-native speed on Apple Silicon)
3. Choose **Linux**
4. Browse to your Ubuntu Server ISO
5. Configure resources:
   - **RAM**: 8 GB minimum (AlphaClaw recommends 8 GB)
   - **CPU**: 4 cores
   - **Disk**: 30 GB minimum
6. Finish and boot the VM

---

## Step 4 — Install Ubuntu Server

1. Follow the Ubuntu installer prompts (defaults are fine for most screens)
2. Enable **OpenSSH server** when prompted — this lets you SSH in from your Mac's terminal, which is much nicer than the UTM console window
3. Complete installation and reboot
4. Remove the ISO from the VM's CD drive in UTM settings so it boots from disk

---

## Step 5 — SSH into the VM (optional but recommended)

From the UTM console, find the VM's IP:

```bash
ip addr show
```

Then from your Mac terminal:

```bash
ssh your-username@<vm-ip-address>
```

---

## Step 6 — Install Node.js 22+

```bash
sudo apt update && sudo apt upgrade -y
curl -fsSL https://deb.nodesource.com/setup_22.x | sudo -E bash -
sudo apt install -y nodejs git curl
node --version  # should show v22.x.x or higher
```

---

## Step 7 — Install OpenClaw

AlphaClaw wraps OpenClaw and manages it as a child process. Install OpenClaw globally first:

```bash
sudo npm install -g openclaw@latest
openclaw --version
```

---

## Step 8 — Install and Run AlphaClaw

```bash
mkdir ~/alphaclaw && cd ~/alphaclaw
npm install @chrysb/alphaclaw
npx alphaclaw start
```

AlphaClaw will start on port 3000 by default.

---

## Step 9 — Access the Setup UI from your Mac

From your Mac browser, navigate to:

```
http://<vm-ip-address>:3000
```

You'll be prompted for the setup password. Set it via environment variable before starting:

```bash
SETUP_PASSWORD=your-strong-password npx alphaclaw start
```

---

## Alternative: Docker Inside the VM

If you prefer to match the project's recommended setup exactly:

```bash
# Install Docker inside the VM
sudo apt install -y docker.io
sudo systemctl enable --now docker
sudo usermod -aG docker $USER
# Log out and back in for group change to take effect

# Run AlphaClaw via Docker
docker run -d \
  -p 3000:3000 \
  -e SETUP_PASSWORD=your-strong-password \
  -v alphaclaw-data:/data \
  --name alphaclaw \
  chrysb/alphaclaw
```

---

## Security Reminders

- **Do not share your host Mac's filesystem** with the VM (no shared folders)
- **Review the source code** before providing any API keys, tokens, or credentials
- **Use throwaway/test credentials** until you've verified the project is legitimate
- The VM is your security boundary — if anything goes wrong, you can delete it entirely with zero impact on your Mac
- There are **active phishing reports** targeting OpenClaw-related repos. Verify this project through official OpenClaw channels before trusting it with real credentials

---

## Useful UTM Tips

- **Snapshots**: Take a VM snapshot before running AlphaClaw so you can roll back instantly
- **Network isolation**: In UTM VM settings, you can switch networking to "Host Only" to prevent the VM from reaching the internet (useful after initial setup)
- **Clipboard sharing**: Keep it disabled for untrusted software

---

## FAQ

### What is OpenClaw?

[OpenClaw](https://github.com/anthropics/openclaw) is an open-source AI agent framework that runs locally on your machine. It connects to messaging platforms (WhatsApp, Telegram, Discord, Slack) and can automate tasks, answer questions, and manage workflows on your behalf. It uses LLM providers like Anthropic, OpenAI, and others under the hood. Think of it as a self-hosted AI assistant you control.

### What is AlphaClaw?

[AlphaClaw](https://github.com/chrysb/alphaclaw) is a third-party wrapper around OpenClaw that adds a browser-based setup UI, a self-healing watchdog, Git-backed rollback, and simplified integrations. It's designed to make deploying and managing OpenClaw easier — no CLI required after initial setup. AlphaClaw is not an official OpenClaw project.

### What is UTM?

[UTM](https://mac.getutm.app) is a free, open-source virtual machine application for macOS. It lets you run full operating systems (Linux, Windows, etc.) inside an isolated sandbox on your Mac. On Apple Silicon, it uses Apple's native Virtualization framework for near-native performance. It's the macOS equivalent of VirtualBox or VMware, but built specifically for Mac.

### Why run AlphaClaw in a VM instead of directly on my Mac?

Isolation. A VM is a completely separate computer running inside your Mac. If the software does anything unexpected — accesses files it shouldn't, phones home, or behaves maliciously — it's contained within the VM. You can delete the VM and everything inside it disappears. Your Mac stays untouched.

### Why not just use Docker on my Mac?

Docker on macOS does run inside a lightweight Linux VM (via Docker Desktop), so there's some isolation. But Docker is designed for convenience, not security boundaries. It shares your host network, can mount host volumes, and isn't hardened against a malicious container escaping. A full VM in UTM gives you a much stronger security boundary.

### Do I need to install OpenClaw separately?

Likely yes. AlphaClaw manages OpenClaw as a child process but the README doesn't confirm it bundles OpenClaw as a dependency. Step 7 in this guide installs OpenClaw globally before AlphaClaw. If AlphaClaw's npm install does pull it in automatically, the global install won't conflict — it just means you'll have it available either way.

### Is AlphaClaw safe to use?

We can't confirm that. There are [active phishing reports](https://www.techedubyte.com/github-phishing-scam-openclaw-wallet-drain-report/) targeting OpenClaw-related repositories on GitHub. AlphaClaw asks for API keys, GitHub tokens, and bot tokens during setup. Run it in a VM, use throwaway credentials, and review the source code before trusting it with anything real. Verify through official OpenClaw channels whether this project is endorsed.

### Can I run this on an Intel Mac?

Yes. UTM supports both Apple Silicon and Intel Macs. For Intel, use the standard amd64 Ubuntu ISO instead of the ARM64 one, and select **Emulate** instead of **Virtualize** when creating the VM (or use Virtualize if UTM offers it for your Intel chip — it depends on the macOS version).

### How much disk space / RAM do I need?

The VM needs about 8 GB of RAM and 30 GB of disk space. Your Mac should have at least 16 GB of total RAM to comfortably run the VM alongside macOS.
