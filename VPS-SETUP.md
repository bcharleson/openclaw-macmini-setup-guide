# OpenClaw VPS Setup Guide

Set up an OpenClaw agent on a cloud VPS (Ubuntu/Debian) with its own isolated user account.

---

## 📋 Before You Begin

**You will need:**
1. Root or sudo access to an Ubuntu/Debian VPS
2. An agent name (e.g., "EVE", "WALL-E", "DataBot")
3. A Slack workspace
4. An Anthropic API key

**Recommended VPS Specs:**
- **Minimum**: 2 CPU cores, 4GB RAM, 20GB SSD
- **Recommended**: 4 CPU cores, 8GB RAM, 40GB SSD
- **OS**: Ubuntu 22.04 LTS or Debian 12

**Supported Cloud Providers:**
- DigitalOcean
- Linode/Akamai Cloud
- AWS EC2
- Google Cloud Platform
- Vultr
- Hetzner

---

## ⚙️ Setup Variables

**Fill these out before starting. You'll use these throughout the guide:**

```bash
# Agent Configuration
AGENT_NAME="EVE"                    # Display name (e.g., "EVE", "WALL-E", "DataBot")
AGENT_USERNAME="eve"                # Lowercase username (e.g., "eve", "walle", "databot")
ADMIN_USERNAME="{your_admin_username}"   # Your admin username

# Slack Tokens (get these in Part 5)
SLACK_APP_TOKEN="xapp-YOUR-APP-TOKEN-HERE"
SLACK_BOT_TOKEN="xoxb-YOUR-BOT-TOKEN-HERE"

# Anthropic API Key
ANTHROPIC_API_KEY="sk-ant-YOUR-API-KEY-HERE"
```

# Please refer to the OpenClaw README.md on how to properly setup auth (such as OAuth).
# if you use OAuth, ensure you're on Claude Pro/Max or OpenAI ChatGPT Pro plans.

---

## Part 1: Initial Server Setup

### Update System

```bash
sudo apt update && sudo apt upgrade -y
```

### Install Essential Tools

```bash
sudo apt install -y curl git build-essential ufw fail2ban
```

### Configure Firewall

```bash
# Allow SSH (IMPORTANT: Do this first!)
sudo ufw allow OpenSSH

# Allow HTTP/HTTPS (if you plan to expose web UI)
sudo ufw allow 80/tcp
sudo ufw allow 443/tcp

# Enable firewall
sudo ufw --force enable

# Check status
sudo ufw status
```

### Set Up fail2ban (SSH Protection)

```bash
sudo systemctl enable fail2ban
sudo systemctl start fail2ban
```

---

## Part 2: Create User Account

Create a dedicated user account for your agent.

```bash
sudo useradd -m -s /bin/bash -c "$AGENT_NAME Agent" $AGENT_USERNAME
```

Set a password:

```bash
sudo passwd $AGENT_USERNAME
```

**Enter a password when prompted** (you'll need this if you ever need to manually log in as the agent).

### Grant Sudo Access (Optional)

If your agent needs sudo for certain operations:

```bash
sudo usermod -aG sudo $AGENT_USERNAME
```

---

## Part 3: Set Up SSH Key Authentication (Recommended)

For secure access to the agent account.

### Generate SSH Key (on your local machine)

```bash
ssh-keygen -t ed25519 -C "openclaw-$AGENT_USERNAME" -f ~/.ssh/openclaw-$AGENT_USERNAME
```

### Copy Public Key to Server

```bash
ssh-copy-id -i ~/.ssh/openclaw-$AGENT_USERNAME.pub $ADMIN_USERNAME@YOUR_VPS_IP
```

### Configure SSH for Agent User

On the server:

```bash
sudo mkdir -p /home/$AGENT_USERNAME/.ssh
sudo cp ~/.ssh/authorized_keys /home/$AGENT_USERNAME/.ssh/authorized_keys
sudo chown -R $AGENT_USERNAME:$AGENT_USERNAME /home/$AGENT_USERNAME/.ssh
sudo chmod 700 /home/$AGENT_USERNAME/.ssh
sudo chmod 600 /home/$AGENT_USERNAME/.ssh/authorized_keys
```

---

## Part 4: Install Node.js and Dependencies

Install Node.js via nvm for the agent user.

### Switch to Agent User

```bash
sudo su - $AGENT_USERNAME
```

### Install NVM

```bash
curl -o- https://raw.githubusercontent.com/nvm-sh/nvm/v0.40.1/install.sh | bash
```

### Load NVM

```bash
export NVM_DIR="$HOME/.nvm"
[ -s "$NVM_DIR/nvm.sh" ] && \. "$NVM_DIR/nvm.sh"
```

### Install Node.js 22

```bash
nvm install 22
nvm use 22
nvm alias default 22
```

### Install pnpm

```bash
npm install -g pnpm
```

### Exit Agent User

```bash
exit
```

---

## Part 5: Clone and Build OpenClaw

### Clone OpenClaw as Agent User

```bash
sudo -u $AGENT_USERNAME bash -c 'cd ~ && git clone https://github.com/openclaw/openclaw.git'
```

### Build OpenClaw

```bash
sudo -u $AGENT_USERNAME bash -c 'source ~/.nvm/nvm.sh && cd ~/openclaw && pnpm install'
```

```bash
sudo -u $AGENT_USERNAME bash -c 'source ~/.nvm/nvm.sh && cd ~/openclaw && pnpm ui:build'
```

```bash
sudo -u $AGENT_USERNAME bash -c 'source ~/.nvm/nvm.sh && cd ~/openclaw && pnpm build'
```

---

## Part 6: Create Slack App

Create a dedicated Slack app for your agent.

### Create App

1. Go to https://api.slack.com/apps
2. Click **"Create New App"** → **"From scratch"**
3. Name: Use your `$AGENT_NAME` (e.g., "EVE")
4. Select your workspace

### Enable Socket Mode

1. Settings → **Socket Mode** → Toggle **ON**
2. Click **"Generate Token and Scopes"**
3. Name: `socket-token`
4. Add scope: `connections:write`
5. Click **Generate**
6. **Copy the App Token** (`xapp-...`) - update `SLACK_APP_TOKEN` variable above

### Add Bot Scopes

Go to **OAuth & Permissions** → **Bot Token Scopes** → Add all:

- `app_mentions:read`
- `channels:history`
- `channels:read`
- `chat:write`
- `commands`
- `emoji:read`
- `files:read`
- `files:write`
- `groups:history`
- `groups:read`
- `groups:write`
- `im:history`
- `im:read`
- `im:write`
- `mpim:history`
- `mpim:read`
- `mpim:write`
- `pins:read`
- `pins:write`
- `reactions:read`
- `reactions:write`
- `users:read`

### Install to Workspace

1. Click **"Install to Workspace"**
2. Click **Allow**
3. **Copy the Bot Token** (`xoxb-...`) - update `SLACK_BOT_TOKEN` variable above

### Enable Events

1. Go to **Event Subscriptions** → Toggle **ON**
2. Under **Subscribe to bot events**, add:
   - `app_mention`
   - `message.channels`
   - `message.groups`
   - `message.im`
   - `message.mpim`
   - `reaction_added`
   - `reaction_removed`
3. Click **Save Changes**

### Enable App Home

1. Go to **App Home**
2. Toggle ON: **"Allow users to send Slash commands and messages from the messages tab"**

---

## Part 7: Configure OpenClaw

Create the OpenClaw configuration file with your Slack tokens.

### Create Configuration Directory

```bash
sudo -u $AGENT_USERNAME mkdir -p /home/$AGENT_USERNAME/.openclaw
```

### Create Configuration File

```bash
sudo -u $AGENT_USERNAME bash -c "cat > /home/$AGENT_USERNAME/.openclaw/openclaw.json << EOF
{
  \"gateway\": {
    \"mode\": \"local\",
    \"port\": 18789
  },
  \"agents\": {
    \"defaults\": {
      \"workspace\": \"~/openclaw\"
    },
    \"list\": [
      {
        \"id\": \"main\",
        \"identity\": {
          \"name\": \"$AGENT_NAME\"
        }
      }
    ]
  },
  \"channels\": {
    \"slack\": {
      \"enabled\": true,
      \"appToken\": \"$SLACK_APP_TOKEN\",
      \"botToken\": \"$SLACK_BOT_TOKEN\",
      \"dm\": {
        \"enabled\": true,
        \"policy\": \"open\",
        \"allowFrom\": [\"*\"]
      },
      \"groupPolicy\": \"open\"
    }
  }
}
EOF"
```

### Secure Configuration File

```bash
sudo chmod 600 /home/$AGENT_USERNAME/.openclaw/openclaw.json
```

### Verify Configuration

```bash
sudo -u $AGENT_USERNAME cat /home/$AGENT_USERNAME/.openclaw/openclaw.json
```

**Check that your tokens are correctly inserted.**

---

## Part 8: Set Up Anthropic API Key

Add the Anthropic API key to the agent's environment.

```bash
sudo -u $AGENT_USERNAME bash -c "echo 'export ANTHROPIC_API_KEY=\"$ANTHROPIC_API_KEY\"' >> ~/.bashrc"
```

### Verify

```bash
sudo -u $AGENT_USERNAME bash -c 'source ~/.bashrc && echo $ANTHROPIC_API_KEY'
```

---

## Part 9: Create systemd Service

Set up OpenClaw to run as a systemd service for automatic startup and management.

### Create Service File

```bash
sudo bash -c "cat > /etc/systemd/system/openclaw-$AGENT_USERNAME.service << 'EOFSERVICE'
[Unit]
Description=OpenClaw Gateway for $AGENT_NAME
After=network.target

[Service]
Type=simple
User=$AGENT_USERNAME
WorkingDirectory=/home/$AGENT_USERNAME/openclaw
Environment=\"PATH=/home/$AGENT_USERNAME/.nvm/versions/node/v22.13.1/bin:/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin\"
Environment=\"NVM_DIR=/home/$AGENT_USERNAME/.nvm\"
EnvironmentFile=/home/$AGENT_USERNAME/.openclaw/openclaw.env
ExecStart=/home/$AGENT_USERNAME/.nvm/versions/node/v22.13.1/bin/pnpm openclaw gateway --port 18789
Restart=always
RestartSec=10
StandardOutput=journal
StandardError=journal
SyslogIdentifier=openclaw-$AGENT_USERNAME

[Install]
WantedBy=multi-user.target
EOFSERVICE"
```

### Create Environment File

```bash
sudo -u $AGENT_USERNAME bash -c "cat > /home/$AGENT_USERNAME/.openclaw/openclaw.env << EOF
ANTHROPIC_API_KEY=$ANTHROPIC_API_KEY
NODE_ENV=production
EOF"
```

```bash
sudo chmod 600 /home/$AGENT_USERNAME/.openclaw/openclaw.env
```

### Reload systemd and Enable Service

```bash
sudo systemctl daemon-reload
sudo systemctl enable openclaw-$AGENT_USERNAME.service
```

---

## Part 10: Start Gateway

### Start the Service

```bash
sudo systemctl start openclaw-$AGENT_USERNAME.service
```

### Check Status

```bash
sudo systemctl status openclaw-$AGENT_USERNAME.service
```

### View Logs

```bash
# Follow logs in real-time
sudo journalctl -u openclaw-$AGENT_USERNAME.service -f

# View last 100 lines
sudo journalctl -u openclaw-$AGENT_USERNAME.service -n 100
```

---

## Part 11: Verify Setup

Check that everything is working.

### Check Gateway Status

```bash
sudo -u $AGENT_USERNAME bash -c 'source ~/.nvm/nvm.sh && cd ~/openclaw && pnpm openclaw status --all'
```

### Probe Slack Connection

```bash
sudo -u $AGENT_USERNAME bash -c 'source ~/.nvm/nvm.sh && cd ~/openclaw && pnpm openclaw channels status --probe'
```

---

## Part 12: Connect to Slack

Test your agent in Slack.

1. In Slack, go to any channel or create a test channel
2. Invite your agent: `/invite @$AGENT_NAME`
3. DM your agent to test!

---

## Maintenance

### Update OpenClaw

Create an update script:

```bash
sudo -u $AGENT_USERNAME bash -c "cat > ~/update-openclaw.sh << 'EOFUPDATE'
#!/bin/bash
set -e

echo \"Stopping gateway...\"
sudo systemctl stop openclaw-$AGENT_USERNAME.service

echo \"Backing up configuration...\"
cp -r ~/.openclaw ~/.openclaw.backup.\$(date +%Y%m%d-%H%M%S)

echo \"Fetching from upstream...\"
cd ~/openclaw
git fetch origin

BEHIND=\$(git rev-list HEAD..origin/main --count)
echo \"Commits behind upstream: \$BEHIND\"

if [ \"\$BEHIND\" -gt 0 ]; then
    echo \"Rebasing on upstream...\"
    git rebase origin/main

    echo \"Installing dependencies...\"
    source ~/.nvm/nvm.sh
    pnpm install

    echo \"Building UI...\"
    pnpm ui:build

    echo \"Building OpenClaw...\"
    pnpm build
else
    echo \"Already up to date!\"
fi

echo \"Starting gateway...\"
sudo systemctl start openclaw-$AGENT_USERNAME.service

echo \"✅ Update complete!\"
EOFUPDATE"
```

```bash
sudo chmod +x /home/$AGENT_USERNAME/update-openclaw.sh
```

### Run Update

```bash
sudo -u $AGENT_USERNAME /home/$AGENT_USERNAME/update-openclaw.sh
```

### Service Management Commands

```bash
# Start service
sudo systemctl start openclaw-$AGENT_USERNAME.service

# Stop service
sudo systemctl stop openclaw-$AGENT_USERNAME.service

# Restart service
sudo systemctl restart openclaw-$AGENT_USERNAME.service

# Check status
sudo systemctl status openclaw-$AGENT_USERNAME.service

# View logs
sudo journalctl -u openclaw-$AGENT_USERNAME.service -f
```

---

## Security Best Practices

### SSH Hardening

Edit `/etc/ssh/sshd_config`:

```bash
# Disable password authentication (use SSH keys only)
PasswordAuthentication no

# Disable root login
PermitRootLogin no

# Allow only specific users
AllowUsers $ADMIN_USERNAME $AGENT_USERNAME
```

Restart SSH:

```bash
sudo systemctl restart sshd
```

### Automatic Security Updates

```bash
sudo apt install -y unattended-upgrades
sudo dpkg-reconfigure --priority=low unattended-upgrades
```

### Firewall Review

```bash
sudo ufw status numbered
```

### Monitor System Resources

```bash
# Install htop
sudo apt install -y htop

# Check resource usage
htop

# Check disk usage
df -h

# Check memory
free -h
```

---

## Troubleshooting

### Service Won't Start

Check logs:
```bash
sudo journalctl -u openclaw-$AGENT_USERNAME.service -n 100
```

Common issues:
- Node path incorrect in service file
- Missing environment variables
- Port 18789 already in use

### Port Already in Use

Check what's using port 18789:
```bash
sudo lsof -i :18789
```

Kill the process:
```bash
sudo kill -9 <PID>
```

### Slack Not Connecting

- Verify tokens in config: `sudo -u $AGENT_USERNAME cat /home/$AGENT_USERNAME/.openclaw/openclaw.json`
- Check Socket Mode is ON in Slack app settings
- Check firewall isn't blocking outbound connections
- Run probe: `sudo -u $AGENT_USERNAME bash -c 'source ~/.nvm/nvm.sh && cd ~/openclaw && pnpm openclaw channels status --probe'`

### Permission Denied Errors

Re-run ownership commands:
```bash
sudo chown -R $AGENT_USERNAME:$AGENT_USERNAME /home/$AGENT_USERNAME/openclaw
sudo chown -R $AGENT_USERNAME:$AGENT_USERNAME /home/$AGENT_USERNAME/.openclaw
```

### Node/pnpm Not Found

Verify Node.js installation:
```bash
sudo -u $AGENT_USERNAME bash -c 'source ~/.nvm/nvm.sh && node --version'
```

Update PATH in service file if needed.

### Out of Memory

Upgrade VPS or add swap:
```bash
sudo fallocate -l 4G /swapfile
sudo chmod 600 /swapfile
sudo mkswap /swapfile
sudo swapon /swapfile
echo '/swapfile none swap sw 0 0' | sudo tee -a /etc/fstab
```

---

## Cloud Provider Specific Notes

### DigitalOcean

- Use Droplets with at least 2GB RAM
- Enable monitoring in control panel
- Consider enabling backups
- Default firewall: use DigitalOcean Cloud Firewall or ufw

### AWS EC2

- Use t3.medium or larger
- Configure Security Groups to allow SSH (22), HTTP (80), HTTPS (443)
- Use Elastic IP for static IP address
- Consider using AWS Systems Manager for secure access

### Google Cloud Platform

- Use e2-medium or larger
- Configure VPC firewall rules
- Use static external IP
- Enable Cloud Monitoring

### Linode/Akamai

- Use 4GB Shared CPU or larger
- Enable Cloud Firewall
- Consider using Linode Backup service
- Use Longview for monitoring

### Vultr

- Use 2GB instance or larger
- Configure firewall rules in control panel
- Enable automatic backups
- Use monitoring dashboard

### Hetzner

- Use CX21 or larger
- Configure firewall rules
- Enable Cloud Firewall if available
- Use monitoring tools

---

## Performance Optimization

### Enable Node.js Production Mode

Already set in service file with `NODE_ENV=production`.

### Increase File Descriptor Limits

Edit `/etc/security/limits.conf`:

```
$AGENT_USERNAME soft nofile 65536
$AGENT_USERNAME hard nofile 65536
```

### Monitor Performance

```bash
# CPU and memory
htop

# Disk I/O
iotop

# Network
iftop
```

---

## ✅ Checklist

- [ ] Set all variables at the top of this guide
- [ ] Updated system and installed essential tools (Part 1)
- [ ] Created user account (Part 2)
- [ ] Set up SSH key authentication (Part 3)
- [ ] Installed Node.js and dependencies (Part 4)
- [ ] Cloned and built OpenClaw (Part 5)
- [ ] Created Slack app and got tokens (Part 6)
- [ ] Configured OpenClaw with Slack tokens (Part 7)
- [ ] Set Anthropic API key (Part 8)
- [ ] Created systemd service (Part 9)
- [ ] Started gateway (Part 10)
- [ ] Verified setup and Slack connection (Part 11)
- [ ] Tested agent in Slack (Part 12)
- [ ] Configured firewall and security (Part 1)
- [ ] Set up automatic updates (Security Best Practices)

---

## Differences from Mac Mini Setup

| Feature | Mac Mini | VPS (Ubuntu/Debian) |
|---------|----------|---------------------|
| User creation | `dscl` | `useradd` |
| Service management | launchd | systemd |
| Package manager | Homebrew | apt |
| Firewall | macOS firewall | ufw |
| User shell | zsh | bash |
| Node.js path | Complex with nvm | Simpler with nvm |
| Automatic startup | launchd plist | systemd service |
| Log viewing | Log files | journalctl |

---

**Your agent is now running on a VPS! 🚀**

**Advantages of VPS Setup:**
- ✅ Always-on availability (no need to keep Mac Mini running)
- ✅ Accessible from anywhere
- ✅ Scalable resources (upgrade CPU/RAM as needed)
- ✅ Professional hosting with backups
- ✅ Better uptime guarantees
- ✅ Lower power costs

**Disadvantages:**
- ❌ Monthly hosting costs ($10-40/month typically)
- ❌ Requires basic Linux knowledge
- ❌ Less physical control
- ❌ Potential data transfer costs
