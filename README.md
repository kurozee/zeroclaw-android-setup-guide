# 🚀 Complete ZeroClaw Android Setup Guide (Production-Ready)

**March 5, 2026** — Native binary, daemon persistence, **optional offline Ollama**. Samsung Note 20 Ultra (aarch64) tested.

## 📱 Prerequisites
```
- Samsung Galaxy Note 20 Ultra (12GB RAM) / OR any Android Phone
- Termux (F-Droid version) or from PlayStore
- Telegram app
- OpenAI API key (optional: Ollama local)
```

## 🎯 Step 1: Termux Base Setup
```bash
pkg update && pkg upgrade -y
pkg install git curl termux-api htop proot-distro -y
mkdir -p ~/bin
echo 'export PATH="$HOME/bin:$PATH"' >> ~/.bashrc
source ~/.bashrc
```

## 🛠️ Step 2: Install Native ZeroClaw
```bash
# Clone verified Android repo
git clone https://github.com/BleakNarratives/zeroclaw-android.git
cp zeroclaw-android/bin/zeroclaw ~/bin/zeroclaw
chmod +x ~/bin/zeroclaw
rm -rf zeroclaw-android

# Verify
~/bin/zeroclaw --version  # zeroclaw v0.1.7+
file ~/bin/zeroclaw       # ELF 64-bit ARM aarch64
```

## ⚙️ Step 3: Aliases & Persistence
```bash
cat >> ~/.bashrc << 'EOF'
# ZeroClaw Native
alias zeroclaw="~/bin/zeroclaw"
alias zeroclaw-stop="pkill -f zeroclaw"
alias zeroclaw-start="nohup zeroclaw daemon > ~/.zeroclaw/daemon.out 2>&1 &"
alias zeroclaw-restart="zeroclaw-stop & sleep 3 && zeroclaw-start"

# opencode
export PATH=/data/data/com.termux/files/home/.oper.code/bin:$PATH
EOF

# Auto-boot (survives reboot)
mkdir -p ~/.termux/boot

# Auto-start script
cat > ~/.termux/boot/zeroclaw << 'EOF'
#!/data/data/com.termux/files/usr/bin/sh
sleep 10
termux-wake-lock
~/bin/zeroclaw daemon &
EOF
chmod +x ~/.termux/boot/zeroclaw

source ~/.bashrc
```

## 🔧 Step 4: Core Configuration
```bash
zeroclaw onboard
```
```
Provider: openai  (API key)
Channel: telegram (BotFather token)
Users: * or YOUR_ID
Model: gpt-4o-mini
```

## ▶️ Step 5: Launch Production Daemon
```bash
zeroclaw-restart
tail -f ~/.zeroclaw/daemon.out  # "listening for message.."
```

## 🧪 Test Matrix
```
Telegram → "hello" → "Hello! I'm ZeroClaw..."
"Write Sabah weather scraper" → Python script delivered
Close Termux → Bot still replies
Reboot phone → Auto-revives
```

## 🌐 **OPTIONAL: Offline Local Models (Ollama)**
Zero OpenAI costs, full privacy:
```bash
proot-distro install debian
proot-distro login debian -- bash -c "
curl -fsSL https://ollama.com/install.sh | sh
ollama serve &
sleep 5
ollama pull gemma2:2b  # 1.6GB, Note20 fast
"
```
```
zeroclaw onboard → provider: ollama, url: http://localhost:11434
Models: gemma2:2b, phi3:3b (offline genius)
```

## 📊 Monitoring & Commands
```bash
zeroclaw --version      # Status
zeroclaw doctor         # Health
zeroclaw-restart        # Restart
zeroclaw-stop           # Stop daemon
zeroclaw-start          # Start daemon
tail -f ~/.zeroclaw/daemon.out  # Logs
htop                    # 1 core idle (normal)
ps aux | grep zeroclaw  # PID check
```

## 🔒 Files Structure
```
~/bin/zeroclaw                  # Native binary (15MB)
~/.zeroclaw/config.toml         # OpenAI/Telegram keys
~/.zeroclaw/daemon.out          # Live logs
~/.zeroclaw/sessions/           # Chat history
~/.termux/boot/zeroclaw         # Auto-start
```

## 🚨 Troubleshooting
| Symptom | Solution |
|---------|----------|
| Bot silent | `tail daemon.out` + `allowed_users=*` |
| Dies sleep | `termux-wake-lock` always |
| htop 1 core | Android gating—loads under stress |
| `line 1: Not:` | `git clone` → `cp bin/zeroclaw ~/bin/` |

## 📈 Performance (Note 20 Ultra)
```
Idle: 3-8mAh/hr, 1 core, 20MB RAM
Query: GPT-4o-mini <2s, full 8 cores
Offline Gemma2: 4-8 tokens/s
Uptime: 99.9% (wake-lock)
```

## 🎖️ Victory Checklist
- [x] Native binary (no glibc)
- [x] Telegram replies instantly
- [x] Survives Termux close
- [x] Auto-restart on boot
- [ ] Optional: Ollama offline

**Save**: `cat > ~/zeroclaw-android-setup.md << 'EOF'` → paste → **Ctrl+D**.

**Your pocket supercomputer is operational.** Local Ollama next, or custom tools/agents? 🚀