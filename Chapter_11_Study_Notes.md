# SRE at Google - Chapter 11: Being On-Call
## Comprehensive Study Notes

---

## 1. Understanding On-Call

### What is On-Call?
- **Definition** — Availability to respond to incidents outside normal hours
- **Purpose** — 24/7 coverage for production issues
- **Responsibility** — Diagnose and fix urgent problems
- **Requirement** — Most SRE roles require on-call participation
- **Reality** — Part of SRE responsibility, not optional

### The On-Call Commitment
- **Availability** — Must be reachable during rotation
- **Response Time** — Alert page → response in minutes
- **Presence** — Full focus on incident during active response
- **Expertise** — Deep knowledge of systems
- **Willingness** — Accept disruption to personal schedule

### Types of On-Call

#### Primary On-Call
- **Role** — First responder to alerts
- **Responsibility** — Triage and initial response
- **Escalation** — Page secondary if needed

#### Secondary On-Call
- **Role** — Backup for primary
- **Responsibility** — Handle escalations, complex issues
- **Standby** — Less active, awaiting escalation

#### Follow-The-Sun On-Call
- **Approach** — Rotate on-call across time zones
- **Benefit** — On-call during work hours for each region
- **Coordination** — Handoff procedures between regions

---

## 2. On-Call Rotations

### Rotation Design
- **Team Size** — Usually 3-10 SREs
- **Rotation Length** — Weekly typical (varies)
- **Handoff** — Clear transition between primary/secondary
- **Coverage** — 24/7/365 with adequate backup
- **Fairness** — Equal distribution of on-call burden

#### Rotation Schedule Example
```
Week 1:  Alice (primary), Bob (secondary)
Week 2:  Bob (primary), Carol (secondary)
Week 3:  Carol (primary), Dave (secondary)
Week 4:  Dave (primary), Alice (secondary)
...repeats every 4 weeks
```

### Rotation Constraints
- **Maximum Frequency** — Not too often (burnout)
- **Minimum Notice** — Know rotation schedule in advance
- **Trading** — Allow swaps with approval
- **Coverage** — Never leave gaps in coverage
- **Fairness** — No one always gets bad shifts

### On-Call Hours
- **Full Coverage** — 24/7 coverage standard
- **Regional** — Follow-the-sun for distributed teams
- **Business Hours** — Lighter rotation during business hours
- **Escalation Hours** — Some teams escalate after-hours

---

## 3. Preparing for On-Call

### Knowledge Requirements
- **System Architecture** — How does system work?
- **Common Failures** — What typically goes wrong?
- **Runbooks** — Procedures for common issues
- **Escalation** — Who to page for different issues?
- **Dependencies** — What systems depend on this?

### Preparation Checklist
- [ ] Read system documentation
- [ ] Review past incidents
- [ ] Understand runbooks
- [ ] Know escalation procedures
- [ ] Test access to systems
- [ ] Know how to page people
- [ ] Understand monitoring and alerting
- [ ] Review team communication channels

### Getting Ready: The Day Before
- **Sync** — Chat with previous on-call
- **Review** — Look at recent alerts/incidents
- **Verify** — Test access to tools
- **Notify** — Tell team you're primary next
- **Prepare** — Have systems accessible

### Setup & Tools
- **Phone/Pager** — Charged, with you always
- **Laptop** — Accessible, fully updated
- **VPN** — Tested and ready
- **Alerts** — Properly configured
- **Chat/Comm** — Available and monitored
- **Backup Plan** — What if phone dies?

---

## 4. During On-Call

### Daily Responsibilities
- **Monitoring** — Watch systems throughout rotation
- **Alerts** — Respond to any alerts
- **Logs** — Monitor for anomalies
- **Communication** — Keep team informed
- **Escalation** — Know when to escalate

### Alert Response Cycle

#### Phase 1: Alert Received
- **Notification** — Pager goes off
- **Response** — Answer within minutes
- **Acknowledgment** — Acknowledge in alerting system
- **Triage** — Determine severity and urgency

#### Phase 2: Investigation
- **Dashboard** — Check relevant graphs
- **Logs** — Review system logs
- **Related Systems** — Check dependencies
- **Communication** — Update team on status
- **Hypothesis** — Form theory about cause

#### Phase 3: Mitigation
- **Quick Fix** — If obvious problem, fix it
- **Workaround** — Temporary fix if needed
- **Escalation** — Page help if needed
- **Documentation** — Track what you're doing

#### Phase 4: Resolution
- **Root Cause** — Understand what happened
- **Permanent Fix** — Implement lasting solution
- **Verify** — Confirm system is healthy
- **Handoff** — Note for future on-call
- **Incident Report** — Document for learning

### Decision Points
```
Alert fires: Is this real? 
→ False positive? → Silence alert, fix alert
→ Real problem? → Continue investigation

Problem identified: How bad is it?
→ Minor degradation? → Document, fix later
→ Major outage? → Emergency response

Can you fix it?
→ Yes, obvious fix? → Fix immediately
→ Yes, but complex? → Page expert
→ No idea? → Page expert immediately
```

---

## 5. Incident Communication

### Communicating During Incidents
- **Status Updates** — Regular updates to team
- **ETA** — When will it be fixed?
- **Impact** — How many users affected?
- **Actions** — What are you doing?
- **Escalation** — What help do you need?

### Communication Channels
- **Slack/Chat** — Team coordination
- **Conference Call** — For complex incidents
- **Status Page** — External user communication
- **Email** — Post-incident summary
- **Docs** — Incident notes for reference

### What Not to Do
- **Don't Panic** — Stay calm, think clearly
- **Don't Guess** — Verify before acting
- **Don't Make It Worse** — Be careful with fixes
- **Don't Isolate** — Keep team informed
- **Don't Forget** — Document for learning

### Difficult Conversations
- **Bad News** — "Service is down"
- **Unknown Fix** — "I don't know how to fix this yet"
- **Long Duration** — "This is taking longer than expected"
- **Escalation** — "I need expert help"
- **Severity** — "This is worse than we thought"

---

## 6. On-Call Fatigue and Burnout

### The On-Call Burden
- **Mental Load** — Always potentially available
- **Disruptions** — Sleep interrupted by alerts
- **Stress** — Pressure to fix problems quickly
- **Time Commitment** — Can't plan personal time
- **Accumulation** — Effects compound over time

### Signs of On-Call Burnout
- **Resentment** — Angry about on-call duty
- **Cynicism** — Skeptical about alerts
- **Reduced Effectiveness** — Slower response, more mistakes
- **Health Issues** — Sleep deprivation, stress
- **Departure** — Leaving the job

### Protecting on-call Health

#### Strategy 1: Reasonable Rotation Frequency
- **Recommendation** — 1 week per month maximum
- **Scheduling** — Spread across team
- **Notice** — Know schedule in advance
- **Swaps** — Allow trading with consent

#### Strategy 2: Alert Quality
- **Reduce False Positives** — So alerts mean something
- **Reduce Total Alerts** — Only alert on important things
- **Smart Thresholds** — Don't alert during sleep on minor issues
- **Escalation** — Page primary, not secondary, for low-severity

#### Strategy 3: Incident Severity
- **P1/Critical** — Immediate response required
- **P2/High** — Respond within 30 minutes
- **P3/Medium** — Respond during business hours
- **P4/Low** — Can wait for scheduled work
- **Non-Critical** — Don't page on-call

#### Strategy 4: Support & Recovery
- **Flexible Hours** — If woken at 2am, can work later
- **Days Off** — Get real time off after intense incidents
- **Debriefing** — Talk about stress after incident
- **Compensation** — Time off for on-call time
- **Mental Health** — Employee assistance programs

---

## 7. On-Call Runbooks

### Runbook Purpose for On-Call
- **Reduce Cognitive Load** — Don't have to remember procedures
- **Ensure Consistency** — Everyone follows same steps
- **Enable Learning** — New people can follow procedures
- **Speed Response** — Don't have to figure out what to do
- **Reduce Mistakes** — Documented steps are less error-prone

### Effective Runbook Components
1. **Alert Name** — What's this runbook for?
2. **Detection** — How do you know if you have this issue?
3. **Severity Assessment** — How bad is it?
4. **Quick Fix** — Is there an obvious quick fix?
5. **Investigation Steps** — How to understand the problem?
6. **Resolution Steps** — How to fix it?
7. **Escalation** — When to page help?
8. **Post-Incident** — What to do after fixing?
9. **Related Issues** — What else might be related?
10. **Contact Info** — Who to escalate to?

### Example Runbook

```
Runbook: Database Connection Pool Exhausted

Detection:
- Alert: "db_connection_pool_exhausted"
- Symptoms: 
  - High latency on all queries
  - New connections failing
  - Error: "connection pool timeout"

Severity:
- P1: If affecting production traffic
- P2: If affecting some users
- P3: If only test systems

Quick Fix (2 minutes to try):
1. Increase pool size (temporary config change)
   - ssh db-server
   - vi /etc/db/config.yaml (DB_POOL_SIZE=100 → 150)
   - systemctl restart db
2. Monitor for recovery (check error rate)

If not fixed in 5 minutes, continue:

Investigation (what's happening?):
1. Check active connections:
   - psql → SELECT count(*) FROM pg_stat_activity
   - Look for long-running queries
2. Check query logs:
   - tail -f /var/log/db.log | grep SLOW
3. Check application logs:
   - Find which app is creating connections
   - Look for connection leaks

Resolution Options:

Option A: Kill long-running queries
- Identify: SELECT * FROM pg_stat_statements ORDER BY mean_time DESC
- Kill: SELECT pg_terminate_backend(pid) FROM pg_stat_activity WHERE ...
- Verify: Connections should drop

Option B: Scale database capacity
- Provision new DB replica
- Update app config to use replica
- Move some traffic to replica
- Verify: Load is distributed

Option C: Scale application
- Reduce connections per app instance
- Restart apps in rolling fashion
- Verify: Connections are balanced

Escalation (when to page):
- If: Can't identify long-running queries AND pool > 95% → Page DBA
- If: Application connection leak detected AND can't fix → Page App Team Lead
- If: Incident > 30 minutes AND not fixed → Page VP Engineering

Post-Incident:
1. Don't close incident until done with these
2. Identify root cause (why did pool exhaust?)
3. Create ticket for permanent fix
4. Discuss with team in standup
5. Update this runbook if procedures changed
```

---

## 8. Learning from Incidents

### Post-Incident Review
- **Timing** — Within 48 hours of incident
- **Attendance** — Responders and interested parties
- **Goal** — Learn, not blame
- **Culture** — Blameless, focused on systems

### Post-Incident Components
1. **Timeline** — What happened and when?
2. **Impact** — How many users? For how long?
3. **Root Cause** — Why did it happen?
4. **Detection** — How was it found?
5. **Response** — What was done to fix?
6. **Prevention** — What prevents recurrence?
7. **Detection** — How can we catch it faster?
8. **Action Items** — Who does what by when?

### Action Item Categories
- **Prevent Recurrence** — Fix root cause
- **Improve Detection** — Catch faster next time
- **Improve Response** — Make fix easier
- **Improve Documentation** — Update runbooks
- **Improve Monitoring** — Add alerts/graphs

### Using Learning to Improve On-Call
- **Update Runbooks** — Based on incident
- **Add Alerts** — For early detection
- **Add Monitoring** — To understand problems
- **Train Team** — Share lessons learned
- **Automate Response** — If repeatedly fixed manually

---

## 9. On-Call Metrics & Improvement

### Metrics to Track
- **Alert Volume** — How many alerts per week?
- **Response Time** — How long to acknowledge?
- **Resolution Time** — How long to fix?
- **False Positive Rate** — % of non-issues
- **Escalation Rate** — How many escalated?
- **Incident Duration** — How long outages last

### Analyzing Metrics
```
Weekly On-Call Metrics:
- Alerts: 150 (up from 120 last week)
- Avg response time: 8 minutes (up from 5)
- Avg resolution: 45 minutes (consistent)
- False positives: 40% (down from 50%)
- Escalations: 5/week

Analysis:
- Alert volume increasing → investigate why
- Response time increasing → team fatigue?
- False positive rate improving → good alert tuning
- On-call burden: moderate
```

### Improvement Priorities
1. **Reduce Alert Volume** — Especially false positives
2. **Reduce Response Time** — Better alerting, runbooks
3. **Reduce Resolution Time** — Automation, better diagnostics
4. **Reduce Escalations** — Better on-call training
5. **Improve Satisfaction** — Surveys, feedback

---

## 10. Key Takeaways

### On-Call Expectations
1. **Availability** — Must be reachable
2. **Response** — Answer in minutes
3. **Expertise** — Know your systems
4. **Responsibility** — Fix problems
5. **Judgment** — Know when to escalate

### Preparation is Critical
1. **Know System** — Architecture and common problems
2. **Know Runbooks** — Procedures for common issues
3. **Know Tools** — Access and use monitoring
4. **Know Team** — Escalation and expertise
5. **Know Yourself** — Your limits and needs

### Incident Response
1. **Triage** — Understand severity
2. **Investigate** — Find root cause
3. **Mitigate** — Quick fix to reduce impact
4. **Resolve** — Permanent fix
5. **Learn** — Post-incident review

### Protecting Team Health
1. **Reasonable Frequency** — Not too often
2. **Alert Quality** — Meaningful alerts only
3. **Support** — Flexible hours, recovery time
4. **Learning** — Continuous improvement
5. **Culture** — Blameless, supportive

---

## 11. Discussion Questions & Study Points

- [ ] What's your current on-call rotation schedule?
- [ ] How many alerts per week do you get?
- [ ] What are your most frequent incidents?
- [ ] Which runbooks exist? Which are missing?
- [ ] How long is average incident resolution?
- [ ] What causes escalations?
- [ ] How do you measure on-call effectiveness?
- [ ] What would improve on-call experience?

---

**Created**: 2026-06-08  
**Source**: Site Reliability Engineering: How Google Runs Production Systems - Chapter 11  
**Next**: Chapter 12 - Effective Troubleshooting  
**Related**: [[Chapter_2_Study_Notes.md]] - SRE Responsibilities and On-Call, [[Chapter_6_Study_Notes.md]] - Monitoring
