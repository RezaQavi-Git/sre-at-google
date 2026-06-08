# SRE at Google - Chapter 2: How Google Runs Production Systems
## Comprehensive Study Notes

---

## 1. Google's SRE Organization Structure

### Team Composition
- **SRE Teams** — Own reliability of specific services/systems
- **Product Teams (Developers)** — Build features and maintain code
- **Shared Responsibility Model** — SREs don't own products; they ensure reliability

### Key Roles
- **SRE Manager** — Leads team, manages capacity, hiring, and strategy
- **Senior SREs** — Technical mentors, architecture decisions, on-call lead
- **SREs** — Day-to-day operations, automation, incident response
- **Associate/Junior SREs** — Learning, shadowing, gradual responsibility increase

### Organizational Philosophy
- SREs are **engineers**, not traditional ops staff
- Expected to understand code and system architecture
- Career path for SREs (not a stepping stone to dev roles)
- Mutual respect between SRE and product teams

---

## 2. SRE Responsibilities

### Primary Responsibilities
- **Availability & Reliability** — Own SLOs for assigned services
- **Latency & Performance** — Monitor and improve response times
- **Disaster Recovery** — Plan for and test failure scenarios
- **Capacity Planning** — Ensure systems handle expected load
- **Change Management** — Control deployments and configuration changes

### Day-to-Day Activities
- **Monitoring & Alerting**
  - Real-time system health monitoring
  - Alert response and investigation
  - Root cause analysis
- **Incident Response**
  - On-call rotations (24/7 coverage)
  - Rapid incident triage and mitigation
  - Communication during outages
- **Automation & Tooling**
  - Build tools to reduce toil
  - Automate repetitive tasks
  - Improve operational efficiency
- **Documentation & Knowledge**
  - Document systems and procedures
  - Maintain runbooks for common issues
  - Knowledge transfer to team

### Strategic Activities
- **System Design & Architecture**
  - Advise on reliability in design phase
  - Scale systems proactively
  - Plan for growth
- **Testing & Resilience**
  - Chaos engineering (intentional failures)
  - Disaster recovery drills
  - Load testing and capacity validation
- **Process Improvement**
  - Identify toil and engineer solutions
  - Improve incident response processes
  - Optimize deployment procedures

---

## 3. On-Call & Incident Response

### On-Call Rotations
- **Purpose** — Ensure 24/7 coverage for production issues
- **Rotation Model**:
  - Primary on-call (primary responder)
  - Secondary on-call (escalation, backup)
  - Scheduled rotation (typically weekly)
- **Coverage** — Global teams provide around-the-clock coverage

### On-Call Expectations
- **Responsiveness** — Answer alerts quickly (minutes)
- **Presence** — Available to handle incidents
- **Knowledge** — Know the systems and runbooks
- **Judgment** — Triage severity and escalate appropriately

### Incident Response Process
1. **Alert Triggered** — Monitoring detects problem
2. **Page On-Call** — Alert sent to on-call engineer
3. **Diagnosis** — Determine root cause
4. **Mitigation** — Quick fix to restore service
5. **Resolution** — Permanent fix or workaround
6. **Post-Incident** — Review and document lessons

### Severity Levels
- **Sev 1** — Complete service outage, affecting many users
  - Immediate escalation
  - All hands on deck
  - CEO-level notification
- **Sev 2** — Significant degradation, affecting important functionality
  - Rapid response required
  - May page multiple people
- **Sev 3** — Minor issues, not affecting core functionality
  - Standard on-call response
  - Can often wait for business hours

---

## 4. Incident Management & Blameless Post-Mortems

### Incident Management Philosophy
- **Blameless Culture** — Focus on systems and processes, not blame individuals
- **Learning Orientation** — Every incident is a learning opportunity
- **Documented Decisions** — Record what happened and why

### Post-Incident Review (PIR) Components
- **Timeline** — When did things happen?
- **Impact** — How many users affected? For how long?
- **Root Cause** — What actually went wrong?
- **Trigger** — What event caused the problem?
- **Resolution** — How was the issue fixed?
- **Action Items** — What will prevent recurrence?

### Blameless Post-Mortem Principles
1. **No Punishment** — Goal is learning, not blame
2. **Assume Good Intent** — People make good decisions with available info
3. **Focus on Systems** — Problem is usually process or tooling, not people
4. **Document Everything** — For institutional knowledge
5. **Share Widely** — Other teams learn from incidents

### Action Items Categories
- **Prevent Recurrence** — Prevent same issue happening again
- **Improve Detection** — Catch similar issues faster
- **Improve Response** — Make incident response easier
- **Process Improvements** — Update procedures/runbooks

---

## 5. Production Readiness Reviews (PRRs)

### What is a PRR?
- **Definition** — Formal review ensuring new service/feature is production-ready
- **Timing** — Before launch to production
- **Purpose** — Catch reliability issues before they affect users

### PRR Checklist Topics
- **Monitoring & Observability**
  - Are metrics being collected?
  - Are alerts configured?
  - Is logging sufficient?
  - Can we detect problems quickly?
- **Scalability**
  - Can system handle expected load?
  - Capacity plan for growth?
  - Load testing results?
- **Security**
  - Authentication/authorization in place?
  - Data protection implemented?
  - Vulnerability scanning done?
- **Disaster Recovery**
  - Backup strategy?
  - Recovery plan documented?
  - MTTR (Mean Time To Recover) targets?
- **Runbooks & Documentation**
  - Operational procedures documented?
  - Common failure modes documented?
  - Troubleshooting guides written?
- **Deployment & Rollback**
  - Automated deployment process?
  - Canary deployment capability?
  - Quick rollback possible?
- **Dependencies**
  - External service dependencies identified?
  - Graceful degradation if dependency fails?
  - Timeout configurations correct?

### PRR Value
- **Prevents firefighting** — Catches issues early
- **Shared responsibility** — Dev and SRE discuss reliability upfront
- **Institutional knowledge** — Standardizes best practices
- **Reduces outages** — Better prepared systems = fewer problems

---

## 6. Disaster Recovery & Resilience Testing

### Types of Failures Considered
- **Data Center Outages** — Entire DC goes down
- **Network Failures** — Partial connectivity loss
- **Cascading Failures** — One failure causes others
- **Resource Exhaustion** — CPU, memory, disk, network limits
- **Configuration Errors** — Bad deployment or config change
- **Dependency Failures** — Services your system depends on fail

### Disaster Recovery Planning
- **RTO (Recovery Time Objective)** — How long is downtime acceptable?
- **RPO (Recovery Point Objective)** — How much data loss is acceptable?
- **Backup Strategy** — How are backups created and tested?
- **Failover Mechanisms** — Automatic vs manual failover?
- **Documentation** — Recovery procedures written and accessible?

### Resilience Testing
- **Chaos Engineering** — Intentionally inject failures
  - Bring down instances
  - Introduce network latency
  - Simulate dependency failures
  - Cause resource exhaustion
- **Disaster Drills** — Simulate major incidents
  - Test response procedures
  - Verify documentation is accurate
  - Train team on escalation
  - Find gaps before real disasters
- **Game Days** — Collaborative exercises
  - Product and SRE teams participate
  - Simulate realistic failure scenarios
  - Learning-focused, not pass/fail

---

## 7. Capacity Planning

### Why Capacity Planning Matters
- **Prevents Overload** — System fails under high load
- **Cost Optimization** — Avoid over-provisioning
- **Growth Planning** — Ensure systems scale with demand
- **Performance** — Maintain SLOs under peak load

### Capacity Planning Process
1. **Collect Metrics**
   - Historical usage trends
   - Seasonal patterns
   - Growth rate
   - Peak loads
2. **Forecast Growth**
   - Project future demand
   - Account for new features
   - Consider market growth
3. **Calculate Requirements**
   - Resources needed to meet demand
   - Buffer for headroom (usually 20-30%)
   - Account for failover capacity
4. **Plan Procurement**
   - Order hardware/cloud resources
   - Schedule deployment
   - Test new capacity
5. **Monitor & Adjust**
   - Track actual vs. predicted growth
   - Adjust models based on reality
   - Regular review cycles

### Headroom & Utilization
- **Headroom** — Spare capacity above expected peak load
- **Typical Target** — 20-30% headroom above peak
- **Too Little** — No buffer for unexpected spikes
- **Too Much** — Wasted resources and money
- **Balance** — Right amount of headroom for SLO and cost

---

## 8. Configuration Management & Change Control

### Challenge
- **Configuration is Critical** — Wrong configs cause outages
- **Scale Problem** — Managing configs across thousands of machines
- **Consistency** — Ensuring all machines have correct config

### Configuration Management Best Practices
- **Version Control** — All configs in version control (like code)
- **Review Process** — Changes reviewed before deployment
- **Testing** — Configs tested in staging before production
- **Rollback Capability** — Quick revert if config is bad
- **Validation** — Configs validated for correctness
- **Automation** — Deployment automated, not manual

### Change Control Process
1. **Author** — Write config change
2. **Review** — Peer/SRE reviews change
3. **Test** — Test in staging environment
4. **Validate** — Ensure config is correct
5. **Deploy** — Roll out to production
6. **Monitor** — Watch for issues
7. **Rollback** — Quick revert if needed

---

## 9. Google's SRE Growth & Scale

### Evolution at Google
- Started as small team solving operational problems
- Grew as Google's services scaled
- Became formal discipline with defined practices
- Now applies across all Google services

### Key Lessons from Google's Experience
- **Automation Scales** — Manual processes don't
- **Measurement Enables Management** — Can't improve what you don't measure
- **Culture Matters** — Blameless environment enables learning
- **Early Investment Pays Off** — Reliability built in from start, not added later
- **Balance is Key** — Reliability and velocity are complementary, not competing

### Applicability Beyond Google
- Principles apply to smaller organizations
- Can be adapted to different scales
- Not about copying Google exactly
- About adopting the mindset and practices

---

## 10. Relationship Between SREs & Product Teams

### The Partnership Model
- **Not Adversarial** — Both want service to succeed
- **Shared Goals** — Aligned on SLOs and velocity
- **Complementary Skills**:
  - Product teams excel at features
  - SREs excel at reliability
- **Clear Boundaries**:
  - Product team owns code quality and features
  - SRE team owns production reliability
  - Both care about meeting SLOs

### Collaboration Points
- **Design Reviews** — SRE input on architecture
- **Production Readiness** — PRR before launch
- **Incident Response** — Team up during major incidents
- **Performance Issues** — Investigate and resolve together
- **Capacity Planning** — Forecast growth and plan resources

### Avoiding Common Pitfalls
- **SREs as "Gatekeepers"** — Should advise, not block
- **Developers Ignoring Reliability** — SLO enforcement prevents this
- **SREs Ignoring Business Needs** — Error budgets help balance needs
- **Lack of Communication** — Regular touchpoints essential

---

## 11. Key Takeaways

### Organizational Insights
1. **SREs are engineers** — Not traditional ops staff
2. **Shared responsibility** — Dev and SRE both care about reliability
3. **Error budgets align incentives** — Both teams work toward same goals
4. **Culture of learning** — Blameless post-mortems enable improvement
5. **Proactive not reactive** — PRR and testing prevent problems

### Operational Practices
1. **On-call is part of SRE role** — But rotation prevents burnout
2. **Incident response is structured** — Clear severity levels and process
3. **Post-mortems are learning events** — Not punishment
4. **Disaster drills are normal** — Testing increases confidence
5. **Automation reduces toil** — Frees time for engineering work

### Strategic Insights
1. **Capacity planning is essential** — Prevents crises
2. **Configuration management is critical** — Configs are code
3. **Production readiness is preventive** — Catch issues early
4. **Measurement enables decisions** — Data-driven operations
5. **Scale requires engineering solutions** — Not just more people

---

## 12. Discussion Questions & Study Points

- [ ] What are the different severity levels for incidents in your organization?
- [ ] How would you structure a PRR checklist for your service?
- [ ] What prevents "blameless" culture in incident reviews?
- [ ] How do you balance on-call burden with team well-being?
- [ ] What would a capacity plan look like for your service?
- [ ] How would chaos engineering improve your system?
- [ ] What's the relationship between SRE and product teams in your org?
- [ ] How do you prevent configuration management disasters?

---

**Created**: 2026-06-08  
**Source**: Site Reliability Engineering: How Google Runs Production Systems - Chapter 2  
**Next**: Review Chapter 3 - Embracing Risk
**Related**: [[Chapter_1_Study_Notes.md]] - Introduction & SRE Fundamentals
