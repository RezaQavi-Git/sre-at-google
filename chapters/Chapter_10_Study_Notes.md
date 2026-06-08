# SRE at Google - Chapter 10: Practical Alerting from Time-Series Data
## Comprehensive Study Notes

---

## 1. Introduction to Time-Series Alerting

### Time-Series Data Review
- **Definition** — Metrics collected over time
- **Examples** — CPU, latency, error rate, requests per second
- **Characteristics** — Regular intervals, historical values
- **Purpose** — Track system behavior over time

### Why Alerting on Time-Series is Complex
- **Variability** — Metrics naturally fluctuate
- **Baselines** — What's normal varies by time, day, load
- **False Positives** — Easy to alert on noise
- **False Negatives** — Easy to miss real problems
- **Trade-off** — Balance sensitivity vs. false positives

### Goals of Time-Series Alerting
1. **Detect Real Problems** — Alert when something is wrong
2. **Minimize False Positives** — Don't alert on normal variation
3. **Minimize False Negatives** — Don't miss real problems
4. **Enable Fast Response** — Alert before user-visible impact
5. **Reduce On-Call Burden** — Alert fatigue undermines effectiveness

---

## 2. Types of Time-Series Alerts

### Type 1: Threshold Alerts
- **Approach** — Alert when metric exceeds/falls below threshold
- **Example** — Alert if error_rate > 1%
- **Pros** — Simple to understand and implement
- **Cons** — Doesn't account for context or trends

#### Static vs. Dynamic Thresholds
```
Static Threshold:
- Alert if error_rate > 1% (always)
- Problem: 0.8% at 3am is fine, 1.2% during peak is bad

Dynamic Threshold:
- Alert if error_rate > (baseline + 50%)
- Accounts for time-of-day, day-of-week variation
```

#### Setting Threshold Levels
- **Too High** — Miss real problems
- **Too Low** — False positives, alert fatigue
- **Right Level** — 2-3 sigma above normal for that time
- **Calculation** — Requires understanding historical data

### Type 2: Rate-of-Change Alerts
- **Approach** — Alert when metric changes rapidly
- **Example** — Alert if latency doubles in 5 minutes
- **Pros** — Catches sudden problems
- **Cons** — Can miss gradual degradation

#### Rate-of-Change Calculation
```
Current value: 50ms
Previous value (5 min ago): 25ms
Change: 50ms - 25ms = 25ms (100% increase)
Threshold: Alert if > 50% change
Action: Alert triggered
```

### Type 3: Trend Alerts
- **Approach** — Alert if metric trending wrong direction
- **Example** — Alert if error rate increasing over 1 hour
- **Pros** — Catches gradual degradation
- **Cons** — Delayed notification

#### Trend Detection
```
Time    Error_Rate
1       0.5%
2       0.6%
3       0.7%
4       0.8%
5       0.9%

Trend: Linear increase
Duration: 5 minute window
Action: Alert if trend continues
```

### Type 4: Composite/Multi-Signal Alerts
- **Approach** — Multiple conditions must be true
- **Example** — Alert if (errors > 1%) AND (latency > 500ms)
- **Pros** — Reduces false positives
- **Cons** — More complex, harder to debug

#### Example Alert Logic
```
IF (error_rate > 1%) 
   AND (error_rate > 2x baseline)
   AND (errors > 100/sec)
   THEN alert with severity=high
```

---

## 3. Data Collection and Preparation

### Time-Series Measurement
- **Interval** — How often is metric collected? (1s, 1m, 5m, 1h?)
- **Resolution** — How detailed? (fine = expensive, coarse = miss detail)
- **Aggregation** — Sum, average, max, percentile?
- **Retention** — How long to keep data? (trade-off: detail vs. storage)

### Common Aggregations
```
Raw metric: Request latency (ms)
[10, 50, 20, 15, 25, 1000, 30, 22, 18, 12]

Average: 239ms (misleading!)
Median (P50): 23.5ms (better)
P95: 1000ms (outliers matter)
Max: 1000ms (worst case)
Count: 10 requests

Use: P95 or P99 for SLO, not average
```

### Data Quality Issues
- **Missing Data** — Gaps in collection
- **Outliers** — Extreme values skewing analysis
- **Noise** — Random variation
- **Seasonality** — Patterns that repeat (daily, weekly, yearly)

### Handling Data Quality
- **Smoothing** — Reduce noise via moving average
- **Outlier Detection** — Identify and handle extreme values
- **Interpolation** — Fill gaps in data
- **Normalization** — Account for seasonality

---

## 4. Alert Fatigue and False Positives

### The Alert Fatigue Problem
- **Definition** — Too many alerts, especially false positives
- **Consequence** — Engineers ignore alerts
- **Result** — Real problems go unnoticed
- **Root Cause** — Overly sensitive thresholds, poor alert design

### Types of False Positives
1. **Noise** — Alert on normal variation
2. **Transient** — Brief spike that resolves itself
3. **Unactionable** — Alert on something we can't fix
4. **Low Priority** — Alert on something that doesn't matter

### Reducing False Positives

#### Strategy 1: Avoid Alerting on Noise
```
Bad: Alert on any deviation
Good: Alert if > 2 sigma deviation
Better: Alert if > 2 sigma AND persists for 5 min
```

#### Strategy 2: Smart Thresholds
- **Context-Aware** — Different thresholds for different times
- **Baseline-Relative** — Alert on % change, not absolute
- **Adaptive** — Learn normal for each time period

#### Strategy 3: Alert Routing
- **Severity Levels** — Not all alerts are equal
- **On-Call vs. Ticket** — Urgent vs. background work
- **Escalation** — Route to expert if needed

#### Strategy 4: Alert Deduplication
- **Deduplicate** — Don't alert multiple times for same issue
- **Flapping** — Alert repeatedly on/off (normal behavior?)
- **Suppression** — Temporarily silence known issues

### Measuring False Positive Rate
```
Total alerts per week: 500
Alerts that required action: 150
False positive rate: 70%

Goal: < 50% false positive rate
```

---

## 5. Alert Design Best Practices

### Principle 1: Alert on Symptoms, Not Causes
- **Bad** — Alert on CPU > 80%
- **Better** — Alert on error_rate > 1%
- **Why** — High CPU might be fine; errors matter to users
- **Rule** — Alert on user-visible impact

### Principle 2: Actionable Alerts
- **Bad** — "Disk usage is 75%"
- **Better** — "Disk will be full in 2 hours at current rate"
- **Why** — Receiver knows what to do
- **Rule** — Alert means action is needed

### Principle 3: Clear Alert Messages
- **Bad** — "alert_12345_fired"
- **Better** — "API latency > 500ms (currently 750ms)"
- **Why** — Clear what's happening, what the threshold is
- **Include** — What, threshold, current value, when to escalate

### Principle 4: Appropriate Severity
- **Critical** — Immediate, user-visible impact
- **High** — Should respond soon, serious degradation
- **Medium** — Should respond within hours
- **Low** — Background info, not urgent
- **Rule** — Use severe sparingly

### Principle 5: Alert Grouping
- **Related Alerts** — Group similar alerts
- **Dependencies** — If A fails, B will fail too (suppress B)
- **Example** — Database down → app errors, suppress app error alert
- **Benefit** — Fewer alerts, clearer situation

---

## 6. Advanced Alerting Techniques

### Technique 1: Statistical Anomaly Detection
- **What** — Alert when value is statistically unusual
- **Math** — (current - mean) / stddev
- **Threshold** — Alert if > 2.5 sigma (99.4% confidence)
- **Benefit** — Adapts to changing baselines
- **Challenge** — Requires good historical data

#### Anomaly Detection Example
```
Baseline error rate: 0.5% (mean)
Standard deviation: 0.1%
Current error rate: 0.8%
Z-score: (0.8 - 0.5) / 0.1 = 3.0 sigma
Status: Anomalous (alert)
```

### Technique 2: Forecasting
- **What** — Predict future based on trend
- **Example** — "Disk will be full in 12 hours"
- **Benefit** — Proactive (fix before problem)
- **Challenge** — Accuracy depends on data quality

### Technique 3: Correlation Analysis
- **What** — Find metrics that move together
- **Example** — CPU up → latency up (expected)
- **Benefit** — Understand dependencies
- **Application** — Only alert if unexpected correlation

### Technique 4: Multi-Window Alerting
- **What** — Alert based on multiple time windows
- **Example** — Alert if (short-term is bad) AND (medium-term trending worse)
- **Benefit** — Combine different signals
- **Implementation** — Multiple window sizes, combine with AND/OR

---

## 7. Alert Delivery and Response

### Alert Routing
- **Who Gets Paged?** — On-call engineer
- **Who Gets Ticket?** — Support team (non-urgent)
- **Who Gets Email?** — Team lead (information)
- **Escalation** — Page another engineer if no response

### Alert Channels
- **Paging** — Immediate, urgent (PagerDuty, etc.)
- **Email** — Non-urgent notification
- **Slack/Chat** — Background awareness
- **Dashboard** — Always-on display
- **SMS** — When paging insufficient

### On-Call Response Requirements
- **Ack Time** — How fast to acknowledge? (minutes)
- **Action Time** — How fast to start fixing? (minutes)
- **Resolution Time** — How fast to fix? (hours)
- **Escalation** — Who to page if blocked?

### Alert Context & Runbooks
- **Dashboard Link** — To see graphs
- **Runbook Link** — To see fix procedures
- **Recent Changes** — What deployed recently?
- **Incident History** — Has this happened before?
- **Related Systems** — What else might be affected?

---

## 8. Tools and Technologies

### Time-Series Database Considerations
- **Data Retention** — How long to keep data?
- **Query Language** — Can you query what you need?
- **Alerting Integration** — Does it support alerting?
- **Cost** — Price at scale?

### Popular Alerting Systems
- **Prometheus** — Pull-based metrics with Alertmanager
- **Grafana** — Visualization with alerting
- **Datadog** — SaaS monitoring and alerting
- **New Relic** — APM with alerting
- **CloudWatch** — AWS managed alerting

### Alert Configuration
```
Alert Definition:
name: high_error_rate
condition: error_rate > 1% for 5 minutes
severity: critical
runbook: https://wiki/errors/high_error_rate
on_call: backend-team
escalation: backend-manager (15 min timeout)
```

---

## 9. Common Alerting Pitfalls

### Pitfall 1: Too Many Alerts
- **Problem** — Alert on everything
- **Consequence** — Alert fatigue, real issues missed
- **Solution** — Be selective, fewer is better

### Pitfall 2: Alerts Without Context
- **Problem** — Alert fires but unclear why
- **Consequence** — Wasted time investigating
- **Solution** — Include runbooks, context, graphs

### Pitfall 3: Unactionable Alerts
- **Problem** — Alert on something you can't fix
- **Consequence** — Frustration, ignored
- **Solution** — Only alert on actionable issues

### Pitfall 4: No Escalation
- **Problem** — Alert fires but no one gets notified
- **Consequence** — Problems go unnoticed
- **Solution** — Clear escalation procedures

### Pitfall 5: Static Thresholds
- **Problem** — Same threshold 24/7
- **Consequence** — False positives during high load, misses during low
- **Solution** — Context-aware, adaptive thresholds

### Pitfall 6: Poor Alert Quality
- **Problem** — Alert message unclear
- **Consequence** — Wasted time figuring out what's wrong
- **Solution** — Clear, actionable messages

---

## 10. Building Alert Dashboards

### Dashboard for Alert Creators
- **Current Alerts** — Active, firing now
- **Alert History** — Recent alerts, patterns
- **False Positive Rate** — % of alerts that were false
- **Response Time** — How fast people respond
- **Tuning Recommendations** — Which thresholds need adjustment

### Metrics to Track
```
Weekly Alert Metrics:
- Total alerts: 500
- High severity: 50 (10%)
- Critical: 10 (2%)
- False positive rate: 30%
- Avg response time: 5 minutes
- Avg resolution time: 20 minutes

Trend: False positive rate increasing → tune thresholds
```

---

## 11. Key Takeaways

### Alert Design Principles
1. **Alert on Symptoms** — User-visible impact
2. **Make Actionable** — Receiver knows what to do
3. **Minimize False Positives** — Credibility matters
4. **Appropriate Severity** — Match urgency
5. **Include Context** — Help with diagnosis

### Technical Approaches
1. **Threshold Alerts** — Simple, effective baseline
2. **Rate-of-Change** — Catch sudden problems
3. **Trend Analysis** — Catch gradual degradation
4. **Composite Alerts** — Multiple signals
5. **Anomaly Detection** — Statistical approach

### Operational Excellence
1. **Clear Routing** — Who gets paged?
2. **Fast Escalation** — Procedures if no response
3. **Runbooks Available** — How to fix
4. **Regular Review** — Tune alert quality
5. **Measure Everything** — Track alert effectiveness

### Alert Fatigue Prevention
1. **Be Selective** — Alert on critical issues
2. **Smart Thresholds** — Context-aware, adaptive
3. **Alert Deduplication** — One alert per issue
4. **Escalation Rules** — Clear routing
5. **Regular Tuning** — Reduce false positives

---

## 12. Discussion Questions & Study Points

- [ ] What are your top 5 most frequent alerts?
- [ ] How many are false positives?
- [ ] What alerts would you eliminate?
- [ ] What metrics should you be alerting on but aren't?
- [ ] How would you implement adaptive thresholds?
- [ ] What runbooks are missing for your alerts?
- [ ] How long does response to alerts take?
- [ ] How would you measure alert quality?

---

**Created**: 2026-06-08  
**Source**: Site Reliability Engineering: How Google Runs Production Systems - Chapter 10  
**Next**: Chapter 11 - Being On-Call  
**Related**: [[Chapter_6_Study_Notes.md]] - Monitoring Distributed Systems
