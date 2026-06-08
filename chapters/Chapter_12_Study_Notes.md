# SRE at Google - Chapter 12: Effective Troubleshooting
## Comprehensive Study Notes

---

## 1. Troubleshooting Overview

### What is Troubleshooting?
- **Definition** — Process of finding and fixing problems
- **Goal** — Restore service as quickly as possible
- **Scope** — From user report to root cause fix
- **Complexity** — Simple to extremely complex
- **Skills** — Blend of technical knowledge and problem-solving

### Troubleshooting vs. Debugging
- **Troubleshooting** — User-facing symptoms, find root cause
- **Debugging** — Code-level investigation, step through code
- **Relationship** — Troubleshooting often leads to debugging
- **Timeline** — Troubleshooting is urgent, debugging is methodical

### Why Troubleshooting is Hard
- **Incomplete Information** — Users report symptoms, not causes
- **Distributed Systems** — Problem could be anywhere
- **Unknown Unknowns** — Don't know what to look for
- **Time Pressure** — Users impacted, need quick fix
- **Stress** — High-stakes, urgent situations
- **Complexity** — Modern systems are very complex

---

## 2. The Troubleshooting Process

### Phase 1: Symptom Identification

#### What Users Report
- **Behavior** — "Page is loading slowly"
- **Frequency** — "Happens sometimes"
- **Scope** — "Only on mobile" or "Everyone"
- **Timeline** — "Started at 3pm"
- **Comparison** — "Different from yesterday"

#### Clarifying Questions
- **When?** — First noticed? Still happening?
- **Where?** — One user? All users? Some regions?
- **What?** — Exact error message? Partial failure?
- **How Often?** — Every time? Intermittent?
- **What Changed?** — What was different before?

#### Symptom vs. Root Cause
```
User Report (Symptom): "Site is slow"
Possible Root Causes:
- Database queries slow
- Network latency
- Service overloaded
- Bad code change
- Infrastructure issue
- Upstream dependency slow
```

### Phase 2: Information Gathering

#### Data Sources
1. **Metrics** — Graphs of system performance
2. **Logs** — Detailed event records
3. **Traces** — Request flow across services
4. **Alerts** — What triggered?
5. **Status Page** — Other known issues?
6. **Recent Changes** — Deployments, config changes
7. **Dependency Status** — Are upstream services OK?

#### Key Questions to Answer
- **Is this a user-facing issue?** (impact)
- **Is this widespread or isolated?** (scope)
- **Is this a service issue or dependency issue?** (location)
- **What changed recently?** (correlation)
- **What's the pattern?** (timing, regularity)

#### Investigation Tools
- **Dashboard** — Overview of metrics
- **Logs** — Service logs, application logs
- **Monitoring UI** — Historical graphs
- **SSH** — Access to systems
- **Databases** — Query live data
- **Trace UI** — Distributed tracing
- **Version Control** — See recent changes

### Phase 3: Hypothesis Formation

#### Scientific Method
1. **Observation** — Symptoms and data
2. **Hypothesis** — Theory about cause
3. **Prediction** — "If cause is X, then we'd see Y"
4. **Test** — Check if prediction is true
5. **Conclusion** — Hypothesis confirmed or rejected

#### Example Troubleshooting Flow
```
Symptom: API returning 500 errors

Observation:
- Error rate spiked at 3pm
- New deployment at 2:55pm
- Errors only on /api/v2/users endpoint
- Database is at 40% CPU (normal is 20%)

Hypothesis 1: New code has bug causing errors
- Test: Rollback deployment
- Result: Errors stop
- Conclusion: Confirmed (code issue)

Root Cause Investigation:
- Look at code change
- Added N+1 database query
- Query runs for every request
- This caused database load

Fix: Revert change, optimize query, redeploy
```

### Phase 4: Testing Hypothesis

#### How to Test Hypotheses
- **Observation** — Look at data (logs, metrics, traces)
- **Isolation** — Change one thing at a time
- **Reversion** — Rollback to known-good state
- **Simulation** — Reproduce in test environment
- **Reasoning** — Does data support hypothesis?

#### What NOT to Do
- **Don't Guess** — Test, don't assume
- **Don't Panic** — Stay methodical
- **Don't Make It Worse** — Be careful with changes
- **Don't Assume** — Verify all assumptions
- **Don't Rush** — Slow is smooth, smooth is fast

### Phase 5: Solution Implementation

#### Quick Fix vs. Permanent Fix
```
Quick Fix: Stop the bleeding
- Rollback bad change
- Increase capacity
- Disable feature
- Failover to backup
- Time: Minutes

Permanent Fix: Address root cause
- Fix bug in code
- Optimize query
- Improve monitoring
- Update architecture
- Time: Hours to days
```

#### Fix Verification
- **Does It Work?** — Is symptom gone?
- **Did You Break Anything?** — Check related functionality
- **Is It Stable?** — Sustained or temporary fix?
- **Can You Monitor It?** — Know if it regresses?

---

## 3. Tools and Techniques for Troubleshooting

### Monitoring & Metrics
- **Baseline** — What's normal?
- **Deviations** — What changed?
- **Correlation** — What moved together?
- **Timeline** — When did it change?

#### Using Metrics for Diagnosis
```
Symptom: Slow API response times
Look at:
- Request volume (is traffic up?)
- Database query time (are queries slow?)
- CPU usage (is system overloaded?)
- Error rate (are some requests failing?)
- Upstream latency (is dependency slow?)

Pattern Analysis:
- Correlates with traffic spike → overload
- Correlates with deployment → code issue
- Correlates with data increase → performance regression
- Correlates with time of day → predictable (caching issue?)
```

### Log Analysis
- **Search** — Find relevant log entries
- **Correlation** — Group related logs
- **Timeline** — When did events occur?
- **Error Stack** — Understand what went wrong

#### Log-Based Diagnosis
```
Error in logs: "Database connection timeout"
Investigation:
1. How many? (1 error or many?)
2. When? (exactly when user reported issue?)
3. Who? (all users or specific subset?)
4. Which DB? (main DB or cache?)
5. Why? (pool exhausted or query slow?)

Follow-up logs:
- Look for queries that were running
- Check application logs for connection leaks
- Review recent code changes
- Look for anomalies in request patterns
```

### Distributed Tracing
- **Trace Requests** — See path through system
- **Identify Bottlenecks** — Where time is spent
- **Find Failures** — Which service is failing?
- **Correlate Events** — Timeline across services

#### Using Traces
```
User reports: "Search is slow"

Trace for slow request:
Frontend (50ms) → API (100ms) → Service A (30ms) → Service B (3000ms)

Finding: Service B is slow
Next step: Investigate Service B

In Service B trace:
- Service B → Database (2900ms)
- Database query: SELECT ... (very complex, unoptimized)

Root cause: Slow database query
Fix: Add index, optimize query
```

### Testing in Production
- **Hypothesis Testing** — Test your theory safely
- **Canary Changes** — Roll out to small portion first
- **A/B Testing** — Compare versions
- **Load Testing** — Reproduce in controlled way

---

## 4. Troubleshooting Strategies

### Strategy 1: Divide and Conquer
- **Approach** — Eliminate possibilities systematically
- **Method** — Split problem in half, test each
- **Benefit** — Exponentially faster than linear search
- **Example** —
  ```
  Is issue in frontend or backend?
  → Test backend directly (API call)
  → Issue in backend? Yes
  
  Which backend service?
  → Test Service A directly
  → Issue in Service A? No
  → Test Service B directly
  → Issue in Service B? Yes
  
  Which part of Service B?
  → Test without cache
  → Issue without cache? Yes
  → Root cause likely in query logic
  ```

### Strategy 2: Start with the Most Likely
- **Approach** — Based on experience, try most probable causes first
- **Prioritization** — Order by likelihood
- **Benefit** — Find answer faster
- **Example** —
  ```
  Slow API endpoint:
  1. Most likely: N+1 query (common bug) → Test
  2. Second: Unindexed query → Test
  3. Third: Upstream dependency slow → Test
  4. Fourth: Memory leak → Test
  5. Least likely: CPU/IO bottleneck → Test
  ```

### Strategy 3: Follow the Trail
- **Approach** — Let data guide investigation
- **Pattern** — Look for correlated events
- **Trail** — Follow chain of causality
- **Example** —
  ```
  High latency detected
  ↓
  CPU usage spiked at same time
  ↓
  Specific process using CPU
  ↓
  Recent change to that process
  ↓
  Code change introduced bug
  ↓
  Root cause identified
  ```

### Strategy 4: Check Assumptions
- **Approach** — Verify "obvious" things
- **Reality** — What we assume is often wrong
- **Method** — Explicitly verify assumptions
- **Example** —
  ```
  Assumption: "The database is running"
  Check: Actually verify database is up
  
  Assumption: "The config is correct"
  Check: Read the actual config, don't assume
  
  Assumption: "The code is what we think"
  Check: Verify actual code deployed
  ```

---

## 5. Common Troubleshooting Mistakes

### Mistake 1: Not Reading Error Messages
- **Problem** — Error message says what's wrong, but ignored
- **Example** — "Connection refused" means connection failure, not application bug
- **Solution** — Read error messages carefully

### Mistake 2: Fixing the Wrong Thing
- **Problem** — Fix a symptom, not root cause
- **Example** — Increase cache size instead of fixing bad query
- **Solution** — Find root cause before fixing

### Mistake 3: Over-Confident Diagnosis
- **Problem** — Assume you know what's wrong without testing
- **Example** — "Must be X" without verifying
- **Solution** — Test hypothesis before declaring diagnosis

### Mistake 4: Not Documenting
- **Problem** — Figure out problem, don't write it down
- **Consequence** — Next time it happens, same investigation
- **Solution** — Create runbook, add monitoring

### Mistake 5: Tunnel Vision
- **Problem** — Convinced of one theory, miss other data
- **Example** — Ignore metric that contradicts your theory
- **Solution** — Stay open to data, be willing to revise theory

### Mistake 6: Not Involving Others
- **Problem** — Try to solve alone, stuck
- **Solution** — Ask for help, fresh perspective
- **Benefit** — Often solves faster

---

## 6. Troubleshooting in Distributed Systems

### Complexity Multiplier
- **Single System** — 10 possible failure points
- **Distributed System** — 10^10 possible failure points
- **Scale** — Network, multiple machines, data consistency
- **Uncertainty** — Partial failures, cascading issues

### Network Issues
- **Detection** — Slow requests, timeouts
- **Diagnosis** — Latency measurements, packet loss
- **Root Cause** — Network congestion, bad hardware
- **Testing** — Network diagnostics tools

### Cascading Failures
- **What** — One failure causes another
- **Example** — Database down → App errors → Load on other service → That service fails
- **Detection** — Multiple services degraded at same time
- **Solution** — Implement bulkheads, circuit breakers, graceful degradation

### Data Consistency Issues
- **What** — Data differs across replicas
- **Example** — User sees different data on different regions
- **Detection** — Compare data across instances
- **Root Cause** — Replication lag, split brain, corruption

---

## 7. Troubleshooting Under Pressure

### Staying Calm
- **Acknowledge Pressure** — Accept it's stressful
- **Breathe** — Literally, take deep breaths
- **Step Back** — Sometimes you need a moment
- **Communicate** — Let team know what you're doing

### Methodical Approach
- **Slow Down** — Rushing leads to mistakes
- **Document** — Write down what you're doing
- **Verify** — Don't assume, test
- **Ask for Help** — When stuck, escalate
- **Think Before Acting** — Could this make it worse?

### Time Management
- **Quick Wins** — Try obvious fixes first
- **Escalation Point** — "If not fixed by X minutes, escalate"
- **Parallel Paths** — Work on root cause while having fallback
- **Know When to Give Up** — If approach not working, try different one

---

## 8. Learning from Troubleshooting

### Creating Better Runbooks
- **Document** — Write down what you did
- **Include** — Symptoms, diagnosis steps, resolution
- **Test** — Verify runbook works
- **Share** — Make available to team

### Adding Monitoring
- **Found Issue?** — Add monitor to catch earlier next time
- **Slow Diagnosis?** — Add data collection to ease diagnosis
- **Repeated Issue?** — Prevent via automated check

### Improving Observability
- **Logging** — Are there sufficient logs?
- **Metrics** — Are you measuring the right things?
- **Tracing** — Can you see request flow?
- **Testing** — Can you reproduce?

### Preventing Recurrence
- **Root Cause** — Fix the underlying issue
- **Broader Impact** — Are other services vulnerable?
- **Code Review** — Would review have caught this?
- **Testing** — Add test case to prevent regression

---

## 9. Key Troubleshooting Skills

### Technical Skills
1. **System Knowledge** — Understanding architecture
2. **Tool Proficiency** — Using monitoring, logging, debugging tools
3. **Query Skills** — Reading logs, running queries
4. **Network Understanding** — How systems communicate
5. **Database Knowledge** — Diagnosis and optimization

### Problem-Solving Skills
1. **Pattern Recognition** — Seeing patterns in data
2. **Hypothesis Formation** — Developing theories
3. **Logical Reasoning** — Testing systematically
4. **Creativity** — Thinking outside the box
5. **Persistence** — Not giving up easily

### Interpersonal Skills
1. **Communication** — Explaining situation clearly
2. **Collaboration** — Working with team
3. **Escalation** — Knowing when to ask for help
4. **Empathy** — Understanding user impact
5. **Calm Demeanor** — Not panicking under pressure

---

## 10. Key Takeaways

### Troubleshooting Process
1. **Identify Symptoms** — What are users experiencing?
2. **Gather Information** — Metrics, logs, traces
3. **Form Hypothesis** — Theory about cause
4. **Test Hypothesis** — Verify theory
5. **Implement Solution** — Fix problem
6. **Learn** — Prevent recurrence

### Effective Strategies
1. **Divide and Conquer** — Eliminate possibilities systematically
2. **Most Likely First** — Prioritize by probability
3. **Follow the Trail** — Let data guide investigation
4. **Check Assumptions** — Verify what you assume
5. **Stay Methodical** — Don't rush, test carefully

### Best Practices
1. **Read Error Messages** — They often tell you what's wrong
2. **Test Don't Guess** — Verify before declaring cause
3. **Document** — Create runbooks, share learning
4. **Improve Observability** — Better data enables faster diagnosis
5. **Know When to Escalate** — Ask for help, fresh perspectives

### Under Pressure
1. **Stay Calm** — Panic leads to mistakes
2. **Go Methodical** — Step by step, not random
3. **Communicate** — Keep team informed
4. **Know Limits** — Escalate if needed
5. **Take Breaks** — You need mental clarity

---

## 11. Discussion Questions & Study Points

- [ ] What's the slowest incident you've investigated?
- [ ] What made it hard to diagnose?
- [ ] How could diagnosis have been faster?
- [ ] What tools would have helped?
- [ ] What should have been better documented?
- [ ] How would you measure troubleshooting effectiveness?
- [ ] What's your most common incident?
- [ ] How could you prevent it?

---

**Created**: 2026-06-08  
**Source**: Site Reliability Engineering: How Google Runs Production Systems - Chapter 12  
**Completion**: Full book coverage (Chapters 1-12)  
**Related**: [[Chapter_11_Study_Notes.md]] - Being On-Call, [[Chapter_6_Study_Notes.md]] - Monitoring
