# OpenClaw Agent Setup Guide (Brandon Charleson's configuration)

Set up an OpenClaw agent on a dedicated Mac Mini with its own isolated user account.

---

## 📋 Before You Begin

**You will need:**
1. Admin access to a Mac Mini
2. An agent name (e.g., "EVE", "WALL-E", "DataBot")
3. A Slack workspace
4. An Anthropic API key

---

## ⚙️ Setup Variables

**Fill these out before starting. You'll use these throughout the guide:**

```bash
# Agent Configuration
AGENT_NAME="EVE"                    # Display name (e.g., "EVE", "WALL-E", "DataBot")
AGENT_USERNAME="eve"                # Lowercase username (e.g., "eve", "walle", "databot")
UNIQUE_ID="502"                     # User ID (502, 503, 504, etc. - must be unique)
ADMIN_USERNAME="{your_admin_username}"   # Your admin username

# Slack Tokens (get these in Part 4)
SLACK_APP_TOKEN="xapp-YOUR-APP-TOKEN-HERE"
SLACK_BOT_TOKEN="xoxb-YOUR-BOT-TOKEN-HERE"

# Anthropic API Key
ANTHROPIC_API_KEY="sk-ant-YOUR-API-KEY-HERE"
```

# Please refer to the OpenClaw README.md on how to properly setup auth (such as OAuth). 
# if you use OAuth, ensure you're on Claude Pro/Max or OpenAI ChatGPT Pro plans.

---

## Part 1: Create User Account

Create a dedicated user account for your agent.

```bash
sudo dscl . -create /Users/$AGENT_USERNAME
```

```bash
sudo dscl . -create /Users/$AGENT_USERNAME UserShell /bin/zsh
```

```bash
sudo dscl . -create /Users/$AGENT_USERNAME RealName "$AGENT_NAME"
```

```bash
sudo dscl . -create /Users/$AGENT_USERNAME UniqueID $UNIQUE_ID
```

```bash
sudo dscl . -create /Users/$AGENT_USERNAME PrimaryGroupID 20
```

```bash
sudo dscl . -create /Users/$AGENT_USERNAME NFSHomeDirectory /Users/$AGENT_USERNAME
```

```bash
sudo createhomedir -c -u $AGENT_USERNAME
```

```bash
sudo passwd $AGENT_USERNAME
```

**Enter a password when prompted** (you'll need this if you ever need to manually log in as the agent).

---

## Part 2: Set Up Agent Environment

Clone OpenClaw and set up the agent's workspace.

### Clone OpenClaw (if not already cloned)

```bash
cd ~/Developer
git clone https://github.com/openclaw/openclaw.git openclaw-agent
cd openclaw-agent
git remote remove origin
git remote add upstream https://github.com/openclaw/openclaw.git
```

### Copy to Agent's Home Directory

```bash
sudo cp -r ~/Developer/openclaw-agent /Users/$AGENT_USERNAME/openclaw
```

```bash
sudo chown -R $AGENT_USERNAME:staff /Users/$AGENT_USERNAME/openclaw
```

### Create Configuration Directory

```bash
sudo mkdir -p /Users/$AGENT_USERNAME/.openclaw
```

```bash
sudo chown -R $AGENT_USERNAME:staff /Users/$AGENT_USERNAME/.openclaw
```

### Create Developer Directory

```bash
sudo mkdir -p /Users/$AGENT_USERNAME/Developer
```

```bash
sudo chown -R $AGENT_USERNAME:staff /Users/$AGENT_USERNAME/Developer
```

---

## Part 3: Install Node.js and Build OpenClaw

Install Node.js via nvm and build OpenClaw for the agent.

### Install NVM

```bash
sudo -u $AGENT_USERNAME -i bash -c 'curl -o- https://raw.githubusercontent.com/nvm-sh/nvm/v0.40.1/install.sh | bash'
```

### Install Node.js 22

```bash
sudo -u $AGENT_USERNAME -i bash -c 'source ~/.nvm/nvm.sh && nvm install 22'
```

### Install pnpm

```bash
sudo -u $AGENT_USERNAME -i bash -c 'source ~/.nvm/nvm.sh && npm install -g pnpm'
```

### Install Dependencies

```bash
sudo -u $AGENT_USERNAME -i bash -c 'source ~/.nvm/nvm.sh && cd ~/openclaw && pnpm install'
```

### Build UI

```bash
sudo -u $AGENT_USERNAME -i bash -c 'source ~/.nvm/nvm.sh && cd ~/openclaw && pnpm ui:build'
```

### Build OpenClaw

```bash
sudo -u $AGENT_USERNAME -i bash -c 'source ~/.nvm/nvm.sh && cd ~/openclaw && pnpm build'
```

---

## Part 4: Create Slack App

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

## Part 5: Configure OpenClaw

Create the OpenClaw configuration file with your Slack tokens.

**Note:** The `"workspace": "~/openclaw"` setting refers to the OpenClaw installation directory. The agent will work on projects in `~/Developer/` by default.

```bash
sudo -u $AGENT_USERNAME -i bash -c "cat > ~/.openclaw/openclaw.json << 'EOF'
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

### Verify Configuration

```bash
sudo -u $AGENT_USERNAME cat /Users/$AGENT_USERNAME/.openclaw/openclaw.json
```

**Check that your tokens are correctly inserted.**

---

## Part 6: Set Up Anthropic API Key

Add the Anthropic API key to the agent's environment.

```bash
sudo -u $AGENT_USERNAME -i bash -c "echo 'export ANTHROPIC_API_KEY=\"$ANTHROPIC_API_KEY\"' >> ~/.zshrc"
```

### Verify

```bash
sudo -u $AGENT_USERNAME -i bash -c 'source ~/.zshrc && echo $ANTHROPIC_API_KEY'
```

---

## Part 7: Start Gateway

Start the OpenClaw gateway.

### Foreground Mode (for testing)

```bash
sudo -u $AGENT_USERNAME -i bash -c 'source ~/.nvm/nvm.sh && cd ~/openclaw && pnpm openclaw gateway --port 18789 --verbose'
```

**Press Ctrl+C to stop.**

### Background Mode (for production)

```bash
sudo -u $AGENT_USERNAME -i bash -c 'source ~/.nvm/nvm.sh && cd ~/openclaw && nohup pnpm openclaw gateway --port 18789 > /tmp/openclaw-gateway.log 2>&1 &'
```

**View logs:**
```bash
tail -f /tmp/openclaw-gateway.log
```

---

## Part 8: Verify Setup

Check that everything is working.

### Check Status

```bash
sudo -u $AGENT_USERNAME -i bash -c 'source ~/.nvm/nvm.sh && cd ~/openclaw && pnpm openclaw status --all'
```

### Probe Slack Connection

```bash
sudo -u $AGENT_USERNAME -i bash -c 'source ~/.nvm/nvm.sh && cd ~/openclaw && pnpm openclaw channels status --probe'
```

---

## Part 9: Connect to Slack

Test your agent in Slack.

1. In Slack, go to any channel or create a test channel
2. Invite your agent: `/invite @$AGENT_NAME`
3. DM your agent to test!

---

## Part 10: Admin Access (Optional)

Set up symlinks to easily access your agent's files from your admin account.

### Change Ownership of Work Directories

**Security Note:** We do NOT change ownership of `~/.openclaw` as it contains sensitive tokens.

```bash
sudo chown -R $ADMIN_USERNAME:staff /Users/$AGENT_USERNAME/Desktop
```

```bash
sudo chown -R $ADMIN_USERNAME:staff /Users/$AGENT_USERNAME/Documents
```

```bash
sudo chown -R $ADMIN_USERNAME:staff /Users/$AGENT_USERNAME/Downloads
```

```bash
sudo chown -R $ADMIN_USERNAME:staff /Users/$AGENT_USERNAME/openclaw
```

```bash
sudo chown -R $ADMIN_USERNAME:staff /Users/$AGENT_USERNAME/Developer
```

### Create Symlinks

```bash
ln -s /Users/$AGENT_USERNAME/Developer ~/Developer/${AGENT_USERNAME}-developer
```

```bash
ln -s /Users/$AGENT_USERNAME/Documents ~/Developer/${AGENT_USERNAME}-documents
```

```bash
ln -s /Users/$AGENT_USERNAME/Desktop ~/Developer/${AGENT_USERNAME}-desktop
```

```bash
ln -s /Users/$AGENT_USERNAME/Downloads ~/Developer/${AGENT_USERNAME}-downloads
```

```bash
ln -s /Users/$AGENT_USERNAME/.openclaw ~/Developer/${AGENT_USERNAME}-openclaw-config
```

```bash
ln -s /Users/$AGENT_USERNAME/openclaw ~/Developer/${AGENT_USERNAME}-openclaw
```

```bash
ln -s /Users/$AGENT_USERNAME/Public ~/Developer/${AGENT_USERNAME}-public
```

### Access Agent Files

```bash
# View agent's projects
ls ~/Developer/${AGENT_USERNAME}-developer/

# Work on agent's openclaw repo
cd ~/Developer/${AGENT_USERNAME}-openclaw

# View config (requires sudo due to security)
sudo -u $AGENT_USERNAME cat /Users/$AGENT_USERNAME/.openclaw/openclaw.json
```

---

## Maintenance

### Update OpenClaw

```bash
sudo -u $AGENT_USERNAME -i bash -c 'cd ~/openclaw && git fetch upstream && git merge upstream/main'
```

Then rebuild:

```bash
sudo -u $AGENT_USERNAME -i bash -c 'source ~/.nvm/nvm.sh && cd ~/openclaw && pnpm install && pnpm build'
```

### Restart Gateway

Stop existing gateway (if running in background):

```bash
pkill -f "openclaw gateway"
```

Start again:

```bash
sudo -u $AGENT_USERNAME -i bash -c 'source ~/.nvm/nvm.sh && cd ~/openclaw && nohup pnpm openclaw gateway --port 18789 > /tmp/openclaw-gateway.log 2>&1 &'
```

---

## Troubleshooting

### "User already exists"
Check existing user IDs:
```bash
dscl . -list /Users UniqueID | sort -n -k2
```
Use a different `UNIQUE_ID` (503, 504, 505, etc.).

### "Permission denied"
Re-run ownership commands:
```bash
sudo chown -R $AGENT_USERNAME:staff /Users/$AGENT_USERNAME/openclaw
```

### Slack not connecting
- Verify tokens in config: `sudo -u $AGENT_USERNAME cat /Users/$AGENT_USERNAME/.openclaw/openclaw.json`
- Check Socket Mode is ON in Slack app settings
- Run: `sudo -u $AGENT_USERNAME -i bash -c 'source ~/.nvm/nvm.sh && cd ~/openclaw && pnpm openclaw channels status --probe'`

### Node/pnpm not found
Ensure nvm is sourced: `source ~/.nvm/nvm.sh`

### View gateway logs
```bash
tail -f /tmp/openclaw-gateway.log
```

---

## ✅ Checklist

- [ ] Set all variables at the top of this guide
- [ ] Created user account (Part 1)
- [ ] Set up environment (Part 2)
- [ ] Installed Node.js and built OpenClaw (Part 3)
- [ ] Created Slack app and got tokens (Part 4)
- [ ] Configured OpenClaw with Slack tokens (Part 5)
- [ ] Set Anthropic API key (Part 6)
- [ ] Started gateway (Part 7)
- [ ] Verified setup and Slack connection (Part 8)
- [ ] Tested agent in Slack (Part 9)
- [ ] (Optional) Set up admin access (Part 10)

---

**Your agent is now ready! 🚀**
