[🇬🇧 English](TEMPLATE.md) | [🇷🇺 Русский](TEMPLATE_RU.md)

# Post-Mortem: Template Example

**Date:** YYYY-MM-DD
**Duration:** X hours Y minutes
**Severity:** S1/S2
**Author:** [Your Name]

## Summary

Brief 2-3 sentence summary of what happened, what was affected, and how it was resolved.

## Timeline (UTC)

| Time | Event |
|------|-------|
| HH:MM | Issue reported by [source] |
| HH:MM | Team alerted, investigation started |
| HH:MM | Root cause identified |
| HH:MM | Fix developed and reviewed |
| HH:MM | Fix deployed to production |
| HH:MM | Issue confirmed resolved |

## Root Cause

Technical explanation of what caused the issue. Include code references if applicable.

## Impact

- **Users affected:** [number or percentage]
- **Features impacted:** [list affected features]
- **Data loss:** No / Yes - [details]
- **Financial impact:** N/A or [details]

## Resolution

Description of what was done to fix the issue. Include PR/commit references.

## Lessons Learned

### What went well
- Alerting system notified team within X minutes
- Fix was straightforward once root cause identified
- Good team communication during incident

### What could be improved
- [Item that could have prevented or reduced impact]
- [Item that could improve response time]

## Action Items

| Action | Owner | Due Date | Status |
|--------|-------|----------|--------|
| Add monitoring for [specific metric] | [Name] | [Date] | ⬜ |
| Add test coverage for [scenario] | [Name] | [Date] | ⬜ |
| Update runbook for [process] | [Name] | [Date] | ⬜ |

## References

- GitHub Issue: #XXX
- Pull Request: #XXX
- Slack thread: [link]
- Related documentation: [links]

---

*This template is part of the WakeLink incident response process.*
*See [HOTFIX.md](/HOTFIX.md) for the full hotfix workflow.*
