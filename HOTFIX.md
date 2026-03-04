# WakeLink Hotfix Process

> Emergency fixes for production-critical issues in WakeLink v1.x

## 🚨 When to Use This Process

Use the hotfix process for:
- **S1 Critical**: System down, security breach, data loss
- **S2 High**: Major feature broken, no workaround available

For S3/S4 issues, use the standard release cycle.

---

## 📋 Hotfix Workflow

```
┌─────────────────┐
│  Bug Reported   │
│  (S1/S2 only)   │
└────────┬────────┘
         │
         ▼
┌─────────────────┐
│  Triage & Verify│◄─── QA confirms reproduction
│  (< 1 hour)     │
└────────┬────────┘
         │
         ▼
┌─────────────────┐
│  Create Hotfix  │◄─── git checkout -b hotfix/v1.0.X main
│  Branch         │
└────────┬────────┘
         │
         ▼
┌─────────────────┐
│  Implement Fix  │◄─── Minimal change, single issue
│  + Unit Tests   │
└────────┬────────┘
         │
         ▼
┌─────────────────┐
│  Code Review    │◄─── Fast-track: 1 reviewer, async OK for S1
│  (< 2 hours)    │
└────────┬────────┘
         │
         ▼
┌─────────────────┐
│  Smoke Test     │◄─── Critical paths only
│  (< 30 min)     │
└────────┬────────┘
         │
         ▼
┌─────────────────┐
│  Deploy to Prod │◄─── docker-compose pull && up -d
│                 │
└────────┬────────┘
         │
         ▼
┌─────────────────┐
│  Monitor        │◄─── Watch metrics for 30 min
│  + Communicate  │
└────────┬────────┘
         │
         ▼
┌─────────────────┐
│  Merge & Tag    │◄─── main ← hotfix, tag v1.0.X
│                 │
└────────┬────────┘
         │
         ▼
┌─────────────────┐
│  Post-mortem    │◄─── Write incident report (S1/S2)
│  (within 48h)   │
└─────────────────┘
```

---

## 🔧 Step-by-Step Guide

### 1. Create Hotfix Branch

```bash
# Ensure main is up-to-date
git checkout main
git pull origin main

# Create hotfix branch
git checkout -b hotfix/issue-{NUMBER}-{short-desc}

# Example:
git checkout -b hotfix/issue-142-auth-bypass
```

### 2. Implement Fix

**Rules:**
- Fix ONLY the reported issue
- No refactoring or "while I'm here" changes
- Add regression test for the bug
- Update CHANGELOG.md

```bash
# Make changes
git add -A
git commit -m "fix: [S1] auth bypass in session validation

Fixes #142

- Added missing token expiry check
- Added regression test
"
```

### 3. Fast-Track Review

For S1 (Critical):
- Ping reviewer directly (Slack/Discord DM)
- Async approval acceptable: reviewer can approve via comment
- Single approval sufficient

For S2 (High):
- Standard PR, but expedited
- 2-hour SLA for review

### 4. Smoke Test Checklist

```markdown
- [ ] Server starts without errors
- [ ] Health endpoints respond (GET /health)
- [ ] Authentication works (login flow)
- [ ] Core feature works (wake command sends)
- [ ] No new errors in logs (30 sec observation)
```

### 5. Deploy

```bash
# SSH to production server
ssh deploy@wakelink-prod

# Pull and restart
cd /opt/wakelink
docker-compose -f docker-compose.prod.yml pull api
docker-compose -f docker-compose.prod.yml up -d api

# Verify
docker-compose logs -f api --tail=50
```

### 6. Monitor Post-Deploy

**Watch for 30 minutes:**
- Error rate in Grafana: should return to baseline
- Latency: no degradation
- No new exceptions in logs

**Metrics to check:**
- `http_requests_total{status="5xx"}` — should be near zero
- `http_request_duration_seconds{quantile="0.99"}` — under 500ms
- `wakelink_active_websocket_connections` — stable or growing

### 7. Merge and Tag

```bash
# Merge hotfix to main
git checkout main
git merge --no-ff hotfix/issue-{NUMBER}-{short-desc}

# Tag release
git tag -a v1.0.X -m "Hotfix: {short description}"
git push origin main --tags

# Cherry-pick to develop (if applicable)
git checkout develop
git cherry-pick -x <commit-hash>
git push origin develop

# Delete hotfix branch
git branch -d hotfix/issue-{NUMBER}-{short-desc}
git push origin --delete hotfix/issue-{NUMBER}-{short-desc}
```

---

## 🔄 Rollback Procedure

If the hotfix causes issues:

### Immediate Rollback (< 5 minutes)

```bash
# SSH to production
ssh deploy@wakelink-prod

# Rollback to previous image
cd /opt/wakelink
docker-compose -f docker-compose.prod.yml stop api
docker-compose -f docker-compose.prod.yml up -d api --no-recreate

# Or specify exact previous tag
docker-compose -f docker-compose.prod.yml pull api:v1.0.{X-1}
docker-compose -f docker-compose.prod.yml up -d api
```

### Full Rollback

```bash
# Revert the hotfix commit on main
git checkout main
git revert HEAD
git push origin main

# Re-deploy
# Follow normal deploy process
```

---

## 📢 Communication Templates

### Status Page Update (Incident Start)

```
🔴 INCIDENT: {Short description}

Status: Investigating
Impact: {Users affected / Features impacted}
Started: {UTC timestamp}

We are aware of the issue and actively working on a fix.
Updates will be posted here every 30 minutes.
```

### Status Page Update (Fix Deployed)

```
🟡 UPDATE: Fix deployed, monitoring

Status: Monitoring
Impact: Reduced — fix applied

A fix has been deployed. We are monitoring for stability.
Full resolution expected within {timeframe}.
```

### Status Page Update (Resolved)

```
🟢 RESOLVED: {Short description}

Status: Resolved
Duration: {X hours Y minutes}
Resolution: {Brief description of fix}

The issue has been resolved. All services operating normally.
A post-mortem will be published within 48 hours.
```

### Discord Announcement (Incident)

```
⚠️ **Service Incident**

We're currently experiencing issues with {affected feature}.
Our team is actively investigating.

**Impact:** {What users may notice}
**Status:** Investigating

We'll post updates here as we have them.
```

### Discord Announcement (Resolved)

```
✅ **Incident Resolved**

The issue with {affected feature} has been resolved.

**Duration:** {X hours}
**Root Cause:** {Brief explanation}
**Fix:** deployed v1.0.{X}

Thanks for your patience! 🙏
```

---

## 📝 Post-Mortem Template

Create file: `docs/postmortems/{DATE}-{incident-slug}.md`

```markdown
# Post-Mortem: {Incident Title}

**Date:** {YYYY-MM-DD}
**Duration:** {X hours Y minutes}
**Severity:** S1/S2
**Author:** {Name}

## Summary

{2-3 sentence summary of what happened}

## Timeline (UTC)

| Time | Event |
|------|-------|
| HH:MM | Issue reported by {source} |
| HH:MM | Team alerted, investigation started |
| HH:MM | Root cause identified |
| HH:MM | Fix developed and reviewed |
| HH:MM | Fix deployed to production |
| HH:MM | Issue confirmed resolved |

## Root Cause

{Technical explanation of what caused the issue}

## Impact

- **Users affected:** {number or percentage}
- **Features impacted:** {list}
- **Data loss:** {yes/no, details if yes}
- **Financial impact:** {if applicable}

## Resolution

{What was done to fix the issue}

## Lessons Learned

### What went well
- {Item 1}
- {Item 2}

### What could be improved
- {Item 1}
- {Item 2}

## Action Items

| Action | Owner | Due Date | Status |
|--------|-------|----------|--------|
| {Preventive measure 1} | {Name} | {Date} | ⬜ |
| {Preventive measure 2} | {Name} | {Date} | ⬜ |

## References

- GitHub Issue: #{number}
- PR: #{number}
- Related docs: {links}
```

---

## 📊 SLA Reference

| Severity | Response | Resolution | Escalation |
|----------|----------|------------|------------|
| S1 Critical | 15 min | 4 hours | Immediate: PM + all devs |
| S2 High | 1 hour | 24 hours | Daily standup priority |
| S3 Medium | 4 hours | 72 hours | Next sprint |
| S4 Low | 24 hours | Next release | Backlog |

---

## 🔐 Security Incidents

For security vulnerabilities:

1. **DO NOT** disclose publicly until fix is deployed
2. Create private security advisory on GitHub
3. Follow [SECURITY.md](SECURITY.md) disclosure process
4. Coordinate disclosure timeline with reporter

---

## 📞 Emergency Contacts

| Role | Contact | Availability |
|------|---------|--------------|
| On-call Dev | {name} | 24/7 via PagerDuty |
| Backup Dev | {name} | Business hours |
| DevOps | {name} | 24/7 via PagerDuty |
| PM | {name} | Business hours |

---

## ✅ Hotfix Checklist

Before deploying any hotfix:

```markdown
- [ ] Issue triaged and confirmed as S1/S2
- [ ] Hotfix branch created from main
- [ ] Fix implemented with minimal changes
- [ ] Unit/regression tests added
- [ ] CHANGELOG.md updated
- [ ] Code reviewed (1 reviewer minimum)
- [ ] Smoke tests passed
- [ ] Status page updated (if public-facing)
- [ ] Deployed to production
- [ ] Metrics monitored for 30 minutes
- [ ] Merged to main and tagged
- [ ] Cherry-picked to develop
- [ ] Post-mortem scheduled (S1/S2)
```

---

*Last updated: 2026-02-08*
*Process owner: Senior Developer*
