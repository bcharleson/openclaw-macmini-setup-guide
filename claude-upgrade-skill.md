# OpenClaw Agent Upgrade - Claude Skill

## Description

Guide users through upgrading their OpenClaw agent installation while preserving configuration and ensuring smooth restarts. This skill helps troubleshoot upgrade issues and verify successful updates.

## When to Use This Skill

Invoke this skill when users ask about:
- Updating OpenClaw to the latest version
- Syncing with upstream openclaw repository
- "My agent isn't working after an update"
- Rebuilding OpenClaw after changes
- Upgrading while keeping configuration
- Gateway restart issues

## Prerequisites

Before upgrading, the user should have:
- An existing OpenClaw agent installation (following AGENT-SETUP.md)
- The update script installed at `/Users/{agent_username}/update-openclaw.sh`
- Admin/sudo access to the Mac Mini

## Instructions

### 1. Pre-Upgrade Check

Ask the user:
- Which agent are they upgrading? (Get the agent username, e.g., "walle")
- Is the gateway currently running?
- Do they have any unsaved work in the agent's workspace?

### 2. Verify Update Script Exists

Check if the update script is installed:

```bash
ls -la /Users/{agent_username}/update-openclaw.sh
```

If not found, create it using this content:

```bash
#!/bin/bash
set -e

AGENT_USER="{agent_username}"
OPENCLAW_DIR="/Users/$AGENT_USER/openclaw"
CONFIG_DIR="/Users/$AGENT_USER/.openclaw"
LOG_FILE="/Users/$AGENT_USER/openclaw-update.log"

echo "=== OpenClaw Update Script ===" | tee -a "$LOG_FILE"
echo "Started at: $(date)" | tee -a "$LOG_FILE"

run_as_agent() {
    sudo -u "$AGENT_USER" bash -c "$1"
}

echo "Stopping gateway..." | tee -a "$LOG_FILE"
pkill -f "openclaw gateway" || echo "Gateway not running" | tee -a "$LOG_FILE"
sleep 2

echo "Backing up configuration..." | tee -a "$LOG_FILE"
run_as_agent "cp -r $CONFIG_DIR ${CONFIG_DIR}.backup.$(date +%Y%m%d-%H%M%S)" 2>&1 | tee -a "$LOG_FILE"

echo "Fetching from upstream..." | tee -a "$LOG_FILE"
run_as_agent "cd $OPENCLAW_DIR && git fetch origin" 2>&1 | tee -a "$LOG_FILE"

BEHIND=$(run_as_agent "cd $OPENCLAW_DIR && git rev-list HEAD..origin/main --count")
echo "Commits behind upstream: $BEHIND" | tee -a "$LOG_FILE"

if [ "$BEHIND" -gt 0 ]; then
    echo "Rebasing on upstream..." | tee -a "$LOG_FILE"
    run_as_agent "cd $OPENCLAW_DIR && git rebase origin/main" 2>&1 | tee -a "$LOG_FILE"

    echo "Installing dependencies..." | tee -a "$LOG_FILE"
    run_as_agent "source ~/.nvm/nvm.sh && cd $OPENCLAW_DIR && pnpm install" 2>&1 | tee -a "$LOG_FILE"

    echo "Building UI..." | tee -a "$LOG_FILE"
    run_as_agent "source ~/.nvm/nvm.sh && cd $OPENCLAW_DIR && pnpm ui:build" 2>&1 | tee -a "$LOG_FILE"

    echo "Building OpenClaw..." | tee -a "$LOG_FILE"
    run_as_agent "source ~/.nvm/nvm.sh && cd $OPENCLAW_DIR && pnpm build" 2>&1 | tee -a "$LOG_FILE"
else
    echo "Already up to date!" | tee -a "$LOG_FILE"
fi

echo "Configuration preserved at: $CONFIG_DIR" | tee -a "$LOG_FILE"

echo "Starting gateway in background..." | tee -a "$LOG_FILE"
run_as_agent "source ~/.nvm/nvm.sh && cd $OPENCLAW_DIR && nohup pnpm openclaw gateway --port 18789 > /Users/$AGENT_USER/openclaw-gateway.log 2>&1 &" 2>&1 | tee -a "$LOG_FILE"
sleep 3

if pgrep -f "openclaw gateway" > /dev/null; then
    echo "✅ Gateway started successfully!" | tee -a "$LOG_FILE"
else
    echo "⚠️  Gateway may not have started. Check logs: /Users/$AGENT_USER/openclaw-gateway.log" | tee -a "$LOG_FILE"
fi

echo "Completed at: $(date)" | tee -a "$LOG_FILE"
echo "=== Update Complete ===" | tee -a "$LOG_FILE"
```

Then install it:
```bash
sudo chown {agent_username}:staff /Users/{agent_username}/update-openclaw.sh
sudo chmod +x /Users/{agent_username}/update-openclaw.sh
```

### 3. Run the Update

Execute the update script:

```bash
sudo /Users/{agent_username}/update-openclaw.sh
```

Watch the output for:
- ✅ "Commits behind upstream: X" - shows how many updates
- ✅ "Already up to date!" - no updates needed
- ✅ "Gateway started successfully!" - restart worked
- ⚠️  Any errors during rebase, build, or restart

### 4. Monitor the Update

The script logs everything to `/Users/{agent_username}/openclaw-update.log`

If the update is taking a while, you can monitor progress:

```bash
tail -f /Users/{agent_username}/openclaw-update.log
```

### 5. Verify the Update

After the update completes, verify:

**Check gateway is running:**
```bash
ps aux | grep "openclaw gateway" | grep -v grep
```

**Check status:**
```bash
sudo -u {agent_username} -i bash -c 'source ~/.nvm/nvm.sh && cd ~/openclaw && pnpm openclaw status --all'
```

**Test Slack connection:**
```bash
sudo -u {agent_username} -i bash -c 'source ~/.nvm/nvm.sh && cd ~/openclaw && pnpm openclaw channels status --probe'
```

**View gateway logs:**
```bash
tail -n 50 /Users/{agent_username}/openclaw-gateway.log
```

### 6. Test the Agent

Have the user:
1. Send a DM to the agent in Slack
2. Verify the agent responds
3. Check that the agent can access tools and run commands

## Common Issues and Solutions

### Issue: "Permission denied" on .git directory

**Symptom:** Git operations fail during update

**Solution:**
```bash
sudo chown -R {agent_username}:staff /Users/{agent_username}/openclaw/.git
```

### Issue: Rebase conflicts

**Symptom:** Update stops with "CONFLICT" messages

**Solution:**
```bash
# View conflicts
sudo -u {agent_username} bash -c 'cd ~/openclaw && git status'

# Option 1: Abort and manual fix
sudo -u {agent_username} bash -c 'cd ~/openclaw && git rebase --abort'

# Option 2: Resolve conflicts
# Edit conflicted files manually, then:
sudo -u {agent_username} bash -c 'cd ~/openclaw && git add . && git rebase --continue'
```

### Issue: Gateway won't start after update

**Symptom:** Update completes but "Gateway may not have started" warning

**Solution:**
```bash
# Check logs for errors
tail -n 100 /Users/{agent_username}/openclaw-gateway.log

# Common fixes:
# 1. Port already in use
sudo lsof -i :18789
pkill -f "openclaw gateway"

# 2. Build failed - rebuild manually
sudo -u {agent_username} -i bash -c 'source ~/.nvm/nvm.sh && cd ~/openclaw && pnpm build'

# 3. Start gateway manually
sudo -u {agent_username} -i bash -c 'source ~/.nvm/nvm.sh && cd ~/openclaw && pnpm openclaw gateway --port 18789 --verbose'
```

### Issue: "Command not found: pnpm"

**Symptom:** Build commands fail with pnpm not found

**Solution:**
```bash
# Reinstall pnpm
sudo -u {agent_username} -i bash -c 'source ~/.nvm/nvm.sh && npm install -g pnpm'
```

### Issue: Configuration lost after update

**Symptom:** Agent can't connect to Slack or has wrong settings

**Solution:**
The update script automatically backs up config. Restore if needed:

```bash
# List backups
ls -la /Users/{agent_username}/.openclaw.backup.*

# Restore latest backup
sudo -u {agent_username} cp -r /Users/{agent_username}/.openclaw.backup.YYYYMMDD-HHMMSS /Users/{agent_username}/.openclaw
```

## Update Frequency Recommendations

- **Weekly**: Check for upstream updates
- **Monthly**: Review and apply security patches
- **As needed**: When new features are released
- **Emergency**: When critical bugs are fixed

## Manual Update (Alternative Method)

If the script doesn't work, manual update:

```bash
AGENT_USERNAME="{agent_username}"

# Stop gateway
pkill -f "openclaw gateway"

# Backup config
sudo -u $AGENT_USERNAME cp -r /Users/$AGENT_USERNAME/.openclaw /Users/$AGENT_USERNAME/.openclaw.backup

# Update repo
sudo -u $AGENT_USERNAME bash -c 'cd ~/openclaw && git fetch origin && git rebase origin/main'

# Rebuild
sudo -u $AGENT_USERNAME -i bash -c 'source ~/.nvm/nvm.sh && cd ~/openclaw && pnpm install && pnpm ui:build && pnpm build'

# Restart
sudo -u $AGENT_USERNAME -i bash -c 'source ~/.nvm/nvm.sh && cd ~/openclaw && nohup pnpm openclaw gateway --port 18789 > ~/openclaw-gateway.log 2>&1 &'
```

## Post-Update Checklist

- [ ] Gateway is running (check with `ps aux | grep openclaw`)
- [ ] Slack connection works (DM the agent)
- [ ] Agent responds to commands
- [ ] No errors in gateway logs
- [ ] Config preserved (check ~/.openclaw/openclaw.json)
- [ ] Update log shows success

## Example Usage

**User:** "How do I update Wall-E to the latest OpenClaw version?"

**Agent:** "I'll help you update Wall-E! Let's make sure everything goes smoothly.

First, let me verify the update script is installed:

```bash
ls -la /Users/walle/update-openclaw.sh
```

If it exists, run the update:

```bash
sudo /Users/walle/update-openclaw.sh
```

This will:
1. Stop the gateway
2. Backup your configuration
3. Fetch and rebase upstream changes
4. Rebuild OpenClaw
5. Restart the gateway

The process takes about 2-5 minutes. Watch for the ✅ 'Gateway started successfully!' message.

After it completes, test Wall-E by sending a DM in Slack. Let me know if you see any errors!"

## Files

- `AGENT-SETUP.md`: Initial setup guide
- `claude-skill.md`: Setup assistance skill
- `claude-upgrade-skill.md`: This upgrade skill

## Repository

https://github.com/bcharleson/openclaw-macmini-setup-guide
