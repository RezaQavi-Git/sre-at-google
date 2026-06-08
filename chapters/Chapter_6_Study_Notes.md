# SRE at Google - Chapter 6: Monitoring Distributed Systems
## Comprehensive Study Notes

---

## 1. Monitoring Overview

### Purpose of Monitoring
- **Measure System Health** — Is the system working?
- **Detect Problems** — Identify issues quickly
- **Understand Behavior** — How is the system performing?
- **Debug Issues** — What went wrong and why?
- **Verify SLOs** — Are we meeting targets?
- **Plan Capacity** — What resources are needed?
- **Enable Automation** — Detect issues for auto-response

### Monitoring vs. Observability
- **Monitoring** — Collecting and alerting on metrics
- **Observability** — Ability to understand system state from external outputs
- **Relationship** — Monitoring is part of observability
- **Observability Components**:
  - Metrics (quantitative measurements)
  - Logs (event records)
  - Traces (request flow across systems)

### Why Monitoring is Critical for SRE
- **Enables SLO Verification** — Measure if we're meeting targets
- **Drives Alerting** — Know when to respond
- **Provides Context** — Understand incident impact
- **Prevents Surprises** — Proactive vs. reactive
- **Justifies Investment** — Data for decision-making

---

## 2. Types of Metrics

### Metric Categories

#### USE Method (for system resources)
- **U (Utilization)** — How much of resource is being used?
- **S (Saturation)** — How full is the resource?
- **E (Errors)** — How many errors are occurring?

#### RED Method (for services)
- **R (Rate)** — Requests per second
- **E (Errors)** — Error rate
- **D (Duration)** — Latency/response time

#### Four Golden Signals (Google's approach)
1. **Latency** — How long requests take
2. **Traffic** — How many requests
3. **Errors** — How many failed requests
4. **Saturation** — How full is system

### Metric Types

#### Counter
- **Definition** — Value that only increases
- **Example** — Total requests served, errors, bytes transmitted
- **Use Case** — Track cumulative activity
- **Pros** — Simple, no data loss
- **Cons** — Must calculate rate manually

#### Gauge
- **Definition** — Value that can go up or down
- **Example** — CPU usage, memory, queue depth
- **Use Case** — Measure current state
- **Pros** — Direct measurement
- **Cons** — Can miss spikes between checks

#### Histogram/Distribution
- **Definition** — Distribution of values in ranges
- **Example** — Request latency (50% < 10ms, 95% < 100ms)
- **Use Case** — Understand distribution, percentiles
- **Pros** — Shows full picture
- **Cons** — More complex to implement

### Percentiles vs. Averages
```
Request latencies:
10ms, 20ms, 25ms, 30ms, 10000ms

Average: 2017ms (misleading!)
Median (P50): 25ms (better)
P95: 10000ms (important for user experience)
P99: 10000ms (outliers matter)

Lesson: Use percentiles, not averages
```

---

## 3. Collecting Metrics

### Whitebox vs. Blackbox Monitoring

#### Whitebox Monitoring (Instrumentation)
- **What** — Monitor internal system behavior
- **How** — Instrument code to collect metrics
- **Examples**:
  - Request latency at service boundary
  - Database query duration
  - Cache hit rate
  - Error rates by type
- **Pros** — Complete visibility, understand internals
- **Cons** — Requires code changes, performance impact

#### Blackbox Monitoring (Synthetic Tests)
- **What** — Monitor from outside, like a user
- **How** — External tests probe service
- **Examples**:
  - HTTP health checks
  - Synthetic transactions
  - Uptime monitoring
  - Performance tests
- **Pros** — True user perspective, no code changes
- **Cons** — Limited visibility, can't distinguish root causes

#### Hybrid Approach
- **Best Practice** — Use both together
- **Whitebox** — Understand what's happening
- **Blackbox** — Verify user experience
- **Together** — Complete picture

### Metrics Collection Methods

#### Pull-Based Collection
- **Approach** — Central system pulls metrics from services
- **Example** — Prometheus scraping `/metrics` endpoint
- **Pros** — Simple, easy to scale
- **Cons** — Service must expose metrics

#### Push-Based Collection
- **Approach** — Services push metrics to central system
- **Example** — StatsD, Graphite
- **Pros** — Works for batch jobs, short-lived processes
- **Cons** — More complex, potential data loss

#### Log-Based Metrics
- **Approach** — Extract metrics from logs
- **Example** — Parse access logs for latency distribution
- **Pros** — No code instrumentation needed
- **Cons** — Delayed, possible data loss

---

## 4. Storing & Querying Metrics

### Time Series Databases
- **Purpose** — Store metrics over time
- **Characteristics**:
  - High write throughput (millions per second)
  - Efficient compression (same value repeated)
  - Time-based queries
  - Retention policies (keep recent, discard old)

### Popular Time Series DBs
- **Prometheus** — Pull-based, local storage, good for Kubernetes
- **InfluxDB** — High-cardinality metrics, queryable
- **Graphite** — Push-based, simple, good compatibility
- **CloudWatch** — AWS managed, expensive at scale
- **Datadog** — SaaS monitoring, comprehensive

### Query Language Considerations
- **PromQL** — Prometheus, powerful time series queries
- **InfluxQL** — InfluxDB, SQL-like syntax
- **Custom APIs** — Some systems provide APIs
- **Important** — Should enable quick investigation

---

## 5. Alerting

### Purpose of Alerts
- **Notify on Problems** — Tell someone when action is needed
- **Enable Response** — Can't respond to problems you don't know about
- **Drive Action** — Alert should mean "do something now"

### Alert Design Principles

#### Principle 1: Alerting on Symptoms, Not Causes
- **Bad** — Alert when CPU is 90%
- **Better** — Alert when requests are slow
- **Why** — CPU high might be fine; slow requests matter to users
- **Relationship** — CPU high is symptom, slowness is cause

#### Principle 2: Actionable Alerts
- **Bad** — Alert on condition you can't respond to
- **Good** — Alert only when human action is needed
- **Example** — Don't alert on CPU usage unless it causes problems
- **Implication** — Receiver must know what to do

#### Principle 3: Alert Severity Levels
- **Critical** — Immediate action required, user impact
- **High** — Urgent, needs response soon
- **Medium** — Should respond within hours
- **Low** — Informational, can respond when convenient
- **Practice** — Most alerts should be Medium or Low

### Alert Fatigue
- **Problem** — Too many alerts leads to ignoring them
- **Consequence** — Real problems get missed
- **Solution** — Be selective, alert only on important issues
- **Target** — 1 alert should mean 1 real problem

### Alerting Strategies

#### Strategy 1: Threshold-Based Alerts
- **What** — Alert when metric crosses threshold
- **Example** — Alert if error rate > 1%
- **Pros** — Simple, straightforward
- **Cons** — May not account for context

#### Strategy 2: Rate-of-Change Alerts
- **What** — Alert when metric changes rapidly
- **Example** — Alert if latency jumps 2x
- **Pros** — Catches problems quickly
- **Cons** — Needs tuning to avoid false positives

#### Strategy 3: Trend-Based Alerts
- **What** — Alert if metric trending wrong direction
- **Example** — Alert if error rate increasing over time
- **Pros** — Catches gradual degradation
- **Cons** — Delayed notification

#### Strategy 4: Composite Alerts
- **What** — Multiple conditions must be true
- **Example** — Alert if (errors > 1%) AND (latency > 500ms)
- **Pros** — Reduces false positives
- **Cons** — More complex

---

## 6. Dashboards & Visualization

### Purpose of Dashboards
- **Quick Status Check** — "Is everything OK?"
- **Investigation** — "What's happening right now?"
- **Trend Tracking** — "Is this getting better or worse?"
- **Communication** — "Here's system status for stakeholders"

### Dashboard Design Principles

#### Principle 1: Audience-Specific Dashboards
- **Executive Dashboard** — High-level metrics, business impact
- **SRE Operational Dashboard** — Detailed metrics, system health
- **Developer Dashboard** — Component-specific, what to debug
- **Customer Dashboard** — User-visible status, transparency

#### Principle 2: Golden Signals
- **Show** — Latency, traffic, errors, saturation
- **Why** — These directly indicate user experience
- **Layout** — Large, easy to read numbers

#### Principle 3: Actionable Information
- **Show** — Metrics that enable action
- **Hide** — Internal metrics that don't drive decisions
- **Organize** — Group related metrics

#### Principle 4: Context and History
- **Current** — What's happening now
- **Historical** — Trend over time
- **Baseline** — What's normal for this service
- **Alerts** — When problems occurred

### Dashboard Pitfalls
- **Too Much Information** — Overwhelms viewer
- **Not Updated** — Shows stale data, undermines trust
- **Non-Actionable** — Metrics don't enable response
- **No History** — Can't understand trends
- **Wrong Level of Detail** — Either too high or too low

---

## 7. Monitoring for SLOs

### Connecting Metrics to SLOs
- **SLI** — Measurable metric (like 95% < 100ms latency)
- **Monitoring** — Collect metric and track against SLO
- **Alerting** — Alert if SLO at risk of breach

### SLO-Based Alerting

#### Alert Before Breach
```
Monthly SLO: 99.9% (43 minutes error budget)
Days elapsed: 15 (halfway through month)
Budget consumed: 25 minutes already

Alert threshold: If consuming more than 50% of budget 
at halfway point, adjust risk tolerance

Action: Slow deployments, focus on stability
```

#### Alert on Violation
```
Monthly SLO: 99.9%
Current performance: 99.8% 
Status: In violation

Alert: Trigger incident response, focus on root cause fix
```

### Monitoring Practices for SLOs
1. **Track Against SLO** — Not just raw metric
2. **Measure Error Budget** — How much left?
3. **Alert On Risk** — When approaching breach
4. **Report Status** — Regular SLO reports to team
5. **Post-Incident** — How did incident impact SLO?

---

## 8. Common Monitoring Pitfalls

### Pitfall 1: Measuring Wrong Thing
- **Example** — Monitor CPU instead of latency
- **Problem** — Doesn't reflect user experience
- **Solution** — Monitor SLI, not infrastructure metrics

### Pitfall 2: No Baseline
- **Example** — Alerting when CPU > 80%, but normal is 75%
- **Problem** — Alert fatigue, wrong thresholds
- **Solution** — Establish baseline for each metric

### Pitfall 3: Overly Sensitive Alerts
- **Example** — Alert on every small deviation
- **Problem** — Alert fatigue, ignored
- **Solution** — Set thresholds that indicate real problems

### Pitfall 4: Insufficient Detail
- **Example** — Only monitoring service availability, not latency
- **Problem** — Can't understand what's wrong
- **Solution** — Monitor multiple aspects (latency, errors, etc.)

### Pitfall 5: No Correlation Data
- **Example** — Alert fires but can't see cause
- **Problem** — Hard to debug, slow response
- **Solution** — Correlate metrics, add context

### Pitfall 6: Monitoring Too Fine-Grained
- **Example** — Alert on every function call latency
- **Problem** — Excessive data, noise
- **Solution** — Focus on user-facing metrics

---

## 9. Observability Beyond Metrics

### Logs
- **Purpose** — Record of events and errors
- **Use Case** — Debugging specific problems
- **Challenge** — Volume (billions per day)
- **Best Practice** — Structured logs with consistent format

### Traces/Distributed Tracing
- **Purpose** — Track request flow across services
- **Use Case** — Understand latency, find bottlenecks
- **Implementation** — Add request ID to all logs/spans
- **Tools** — Jaeger, Zipkin, DataDog APM

### Profiling
- **Purpose** — Understand CPU and memory usage
- **Use Case** — Performance optimization
- **Types** — CPU profiling, memory profiling, flamegraphs

### Building Observability
- **Metrics** — What's happening (quantitative)
- **Logs** — Why it's happening (qualitative)
- **Traces** — How requests flow (path-based)
- **Together** — Complete understanding of system

---

## 10. Key Takeaways

### Monitoring Fundamentals
1. **Essential for SRE** — Can't operate without monitoring
2. **Measure what users care about** — Not infrastructure metrics
3. **Use multiple approaches** — Whitebox and blackbox
4. **Collect relevant metrics** — Use golden signals
5. **Store efficiently** — Time series databases

### Alerting Excellence
1. **Alert on symptoms** — User-facing problems
2. **Keep alerts actionable** — Receiver must know what to do
3. **Fight alert fatigue** — Be selective
4. **Appropriate severity** — Match impact level
5. **Update thresholds** — Based on baselines and reality

### Dashboard & Visualization
1. **Audience-specific** — Different users need different views
2. **Show golden signals** — Latency, traffic, errors, saturation
3. **Include history** — Trends matter
4. **Keep updated** — Stale data erodes trust
5. **Enable investigation** — Support root cause analysis

### SLO Monitoring
1. **Monitor SLI directly** — Not proxy metrics
2. **Track error budget** — How much left?
3. **Alert before breach** — Catch problems early
4. **Report regularly** — Team visibility
5. **Continuous improvement** — Refine based on data

---

## 11. Discussion Questions & Study Points

- [ ] What are the golden signals for your service?
- [ ] How would you instrument your code for monitoring?
- [ ] What's a good alert vs. bad alert for your service?
- [ ] How many alerts is too many per day?
- [ ] How would you design a dashboard for your service?
- [ ] How would you measure SLO compliance?
- [ ] What logs would help debug critical issues?
- [ ] How would you correlate metrics across services?

---

**Created**: 2026-06-08  
**Source**: Site Reliability Engineering: How Google Runs Production Systems - Chapter 6  
**Next**: Chapter 7 - The Evolution of Automation at Google  
**Related**: [[Chapter_3_Study_Notes.md]] and [[Chapter_4_Study_Notes.md]] - SLOs and Error Budgets
