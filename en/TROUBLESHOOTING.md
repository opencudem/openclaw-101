# Troubleshooting

> Common issues and how to fix them

## Gateway Issues

### Gateway Won't Start

**Symptoms:** `openclaw gateway start` fails or hangs

**Solutions:**

1. Check port 3000 isn't in use:
   ```bash
   lsof -i :3000
   kill <PID>  # If needed
   ```

2. Check for configuration errors:
   ```bash
   openclaw config validate
   ```

3. Clear and restart:
   ```bash
   openclaw gateway stop
   rm ~/.openclaw/gateway.log
   openclaw gateway start
   ```

4. Check Node.js version:
   ```bash
   node --version  # Should be 18+
   ```

### Gateway Stops Unexpectedly

**Check:**

- Disk space: `df -h`
- Memory: `free -h`
- Logs: `openclaw gateway logs --tail 100`

**Fix:**

```bash
# Run with more verbose logging
OPENCLAW_LOG_LEVEL=debug openclaw gateway start
```

## Channel Issues

### Telegram Bot Not Responding

1. Verify token:
   ```bash
   curl https://api.telegram.org/bot<TOKEN>/getMe
   ```

2. Check if you messaged the right bot

3. Verify webhook isn't set:
   ```bash
   curl https://api.telegram.org/bot<TOKEN>/deleteWebhook
   ```

4. Remove and re-add:
   ```bash
   openclaw channel remove telegram
   openclaw channel add telegram --token <TOKEN>
   ```

### Discord Bot Offline

1. Check bot token in Discord Developer Portal
2. Enable "Message Content Intent" in Bot settings
3. Verify bot has permissions in the server
4. Re-invite with proper permissions:
   - Send Messages
   - Read Message History
   - View Channels

### WhatsApp QR Code Fails

1. Use a dedicated phone number (not primary)
2. Clear previous pairings on phone
3. Ensure phone has stable internet
4. Try `baileys` vs `whatsapp-web.js` alternatives

### Slack Bot Can't DM

1. Bot must be invited to channel first
2. Check bot scopes include: `chat:write`, `im:history`
3. Verify token starts with `xoxb-` (bot token, not user)

## Skill Issues

### Skill Not Found

```bash
# List installed skills
openclaw skill list

# Search for correct name
openclaw skill search <keyword>

# Check if globally available
npm list -g | grep openclaw-skill
```

### Skill Script Errors

1. Check file permissions:
   ```bash
   chmod +x ~/.openclaw/skills/<skill>/scripts/*.js
   ```

2. Verify dependencies installed:
   ```bash
   cd ~/.openclaw/skills/<skill>
   npm install
   ```

3. Check SKILL.md syntax

### Skill Not Updating

```bash
# Force reinstall
openclaw skill remove <name>
openclaw skill install <name>

# Or update via npm
npm update -g openclaw-skill-<name>
```

## Cron Issues

### Job Not Running

1. Check gateway is running
2. Verify cron syntax: https://crontab.guru
3. Check timezone setting
4. Review logs:
   ```bash
   openclaw gateway logs | grep cron
   ```

### Wrong Timezone

```bash
# Set in environment
export TZ=Asia/Ho_Chi_Minh

# Or in config
openclaw config set cron.timezone Asia/Ho_Chi_Minh
```

## API Issues

### 401 Unauthorized

- Check API token is valid
- Verify token hasn't expired
- Confirm correct scopes/permissions

### Rate Limiting

- Add delays between requests
- Implement caching
- Check API quotas

### Connection Timeout

- Check internet connection
- Verify firewall rules
- Test with curl:
  ```bash
  curl -v https://api.openclaw.ai/health
  ```

## Performance Issues

### Slow Response Times

1. Check API latency
2. Reduce context window (shorter conversations)
3. Disable unnecessary skills
4. Check system resources

### High Memory Usage

1. Limit conversation history
2. Archive old memory files
3. Remove unused skills
4. Restart gateway periodically

## Getting Help

### Debug Mode

```bash
# Enable verbose logging
export OPENCLAW_LOG_LEVEL=debug
openclaw gateway start
```

### Gather Information

When reporting issues, include:

1. OpenClaw version: `openclaw --version`
2. Node version: `node --version`
3. OS: `uname -a`
4. Relevant logs: `openclaw gateway logs --tail 100`
5. Config (remove secrets): `openclaw config list`

### Support Channels

- GitHub Issues: https://github.com/opencudem/openclaw-101/issues
- Discord: https://discord.com/invite/clawd
- Documentation: https://docs.openclaw.ai

---

Last updated: 2026-03-31
