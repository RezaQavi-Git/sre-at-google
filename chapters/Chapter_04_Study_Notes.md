# SRE at Google - Chapter 4: Service Level Objectives
## Comprehensive Study Notes

---

## 1. Introduction to Service Level Objectives

### The Purpose of SLOs
- **Bridge Gap** — Between user expectations and system design
- **Enable Decisions** — Guide trade-offs between features and reliability
- **Align Teams** — Create shared understanding across product and ops
- **Measure Progress** — Track and report on reliability
- **Drive Culture** — Emphasize importance of reliability

### SLO Characteristics
- **User-Centric** — Defined from user perspective
- **Measurable** — Based on objective metrics
- **Achievable** — Realistic to maintain
- **Stable** — Don't change frequently
- **Meaningful** — Actually reflect user experience

---

## 2. Service Level Indicators (SLIs)

### Defining SLIs

#### What Makes a Good SLI?
1. **Reflects User Experience** — Metric that matters to users
2. **Observable** — Can be measured from outside the system
3. **Can Be Aggregated** — Can combine across multiple sources
4. **Not Too Fine-Grained** — Practical to collect and store
5. **Valid** — Actually measures what we care about

#### Bad SLIs (Anti-Patterns)
- Internal metrics (CPU, memory) — Don't reflect user experience
- Metrics you can't measure — Not implementable
- Metrics that are always perfect — Not useful
- Metrics that don't correlate to user experience — Misleading

### Common SLI Categories

#### Availability
- **Definition** — Service is accessible and responsive
- **Measurement** — % of requests that succeed
- **Implementation** — HTTP status code (2xx/3xx = success)
- **Example SLI** — "99.95% of requests return 2xx or 3xx status"

#### Latency
- **Definition** — Service responds in timely manner
- **Measurement** — Request response time
- **Implementation** — Measure end-to-end time
- **Example SLI** — "95% of requests complete in < 100ms"

#### Durability
- **Definition** — Written data persists correctly
- **Measurement** — Data integrity, no data loss
- **Implementation** — Verify written data can be retrieved
- **Example SLI** — "99.999% of writes successfully persist"

#### Correctness
- **Definition** — System returns correct results
- **Measurement** — Accuracy of outputs
- **Implementation** — Compare against expected results
- **Example SLI** — "99.9% of search results are relevant"

### Multi-Dimensional SLIs

#### Combining Indicators
- **Availability & Latency** — Request must both succeed AND be fast
- **Weighted Combinations** — Different importance for different metrics
- **User Segments** — Different SLIs for different user types

#### Example Multi-Dimensional SLI
```
"95% of requests from authenticated users complete
successfully within 100ms, AND requests remain available
99.95% of the time"
```

---

## 3. Choosing and Defining SLOs

### Step 1: Understand User Expectations
- **User Research** — What latency is acceptable?
- **Competitor Analysis** — What do others offer?
- **Use Cases** — How will users use service?
- **Context** — Mobile vs desktop, real-time vs batch?

### Step 2: Determine Business Requirements
- **User Impact** — How many users affected by downtime?
- **Revenue Impact** — Cost of service disruption?
- **Strategic Importance** — How critical is service?
- **Market Positioning** — Premium vs budget service?

### Step 3: Assess Technical Feasibility
- **Current Performance** — What can we actually achieve?
- **Infrastructure** — What's required to hit targets?
- **Cost** — What's the budget?
- **Timeline** — How long to implement?

### Step 4: Set SLO Targets
- **Conservative Start** — Better to exceed than miss
- **Achievable Goals** — Should be reachable 95%+ of time
- **Buffer Above SLA** — Leave margin for error
- **Document Rationale** — Why this target?

### Step 5: Implement Measurement
- **Collection** — How will SLI be measured?
- **Instrumentation** — What code changes needed?
- **Alerting** — How will deviations be detected?
- **Dashboards** — How will progress be tracked?

### Example SLO Definition

```
Service: User Search
SLI #1: Availability
- Definition: HTTP request returns 2xx/3xx status
- Measurement: Count successful requests / total requests
- Target: 99.95%
- Rationale: 43 minutes/month acceptable for search feature

SLI #2: Latency (P95)
- Definition: Request completes within 100ms
- Measurement: 95th percentile of request latency
- Target: 95% of requests < 100ms
- Rationale: Users expect sub-100ms interactive response

SLI #3: Durability
- Definition: Saved searches persist reliably
- Measurement: Verify data retrieval after write
- Target: 99.99%
- Rationale: Data loss is critical issue
```

---

## 4. SLOs vs SLIs vs SLAs

### Detailed Comparison

#### Service Level Indicator (SLI)
- **What** — Measurable metric
- **Owner** — SREs/Product team set measurement
- **Focus** — Technical implementation
- **Example** — "Request latency measured at 95th percentile"

#### Service Level Objective (SLO)
- **What** — Target for SLI
- **Owner** — SREs and product team collaborate
- **Focus** — Operational goal
- **Example** — "95% of requests < 100ms"

#### Service Level Agreement (SLA)
- **What** — Contract commitment
- **Owner** — Legal/business teams establish
- **Focus** — Customer expectations
- **Example** — "99.9% uptime guaranteed, $X credit if not met"

### Relationship & Hierarchy

```
SLA (99.0%)  ← Customer commitment
    ↓
SLO (99.5%)  ← Internal target (provides buffer)
    ↓
SLI          ← Measurement mechanism
```

### Why This Structure?
- **Buffer Protection** — SLO > SLA prevents SLA breach
- **Operational Headroom** — Team has target above contract
- **Grace Under Pressure** — Not always hitting limit
- **Professional Standard** — Common practice

---

## 5. Error Budget Impact on SLO Setting

### Relationship to Chapter 3
- **Error Budget** — Derived from SLO
- **SLO Choice** — Determines error budget
- **Budget Implications** — Different SLOs = different budgets

### Error Budget Examples

#### High SLO (99.99%)
```
Monthly downtime budget: 4.3 minutes
Implications:
- Very little room for error
- Conservative change approach
- High infrastructure cost
- Frequent deployments risky
```

#### Medium SLO (99.9%)
```
Monthly downtime budget: 43 minutes
Implications:
- Comfortable margin
- Can deploy frequently
- Reasonable cost
- Enables innovation
```

#### Lower SLO (99%)
```
Monthly downtime budget: 7+ hours
Implications:
- Significant allowance
- Can take major risks
- Lower cost
- Good for experimental services
```

### Choosing SLO Based on Budget Tolerance
1. **Determine Acceptable Risk** — How much downtime is OK?
2. **Set SLO Accordingly** — SLO matches risk tolerance
3. **Verify Achievability** — Can we meet this SLO?
4. **Implement Measurement** — Instrument system to track SLI
5. **Monitor & Adjust** — Track performance, update if needed

---

## 6. SLO Anti-Patterns & Pitfalls

### Anti-Pattern 1: SLO Too High
- **Problem** — Set 99.99% but achieve 99.8%
- **Consequence** — Always in breach, team demoralized
- **Solution** — Set realistic targets based on actual capability

### Anti-Pattern 2: SLO Too Low
- **Problem** — Set 95% when easily achieving 99.5%
- **Consequence** — False sense of success, misses improvements
- **Solution** — Match SLO to business requirements, not minimum capability

### Anti-Pattern 3: No SLO
- **Problem** — No defined target for reliability
- **Consequence** — No alignment, no way to prioritize
- **Solution** — Define explicit SLO even if conservative

### Anti-Pattern 4: Changing SLO Frequently
- **Problem** — Adjust targets every month
- **Consequence** — No stability, hard to track progress
- **Solution** — Set SLO quarterly/annually, stick with it

### Anti-Pattern 5: SLO Too Granular
- **Problem** — Different SLOs for every endpoint/feature
- **Consequence** — Complex, hard to manage
- **Solution** — Few SLOs per service (availability, latency, durability)

### Anti-Pattern 6: SLO Not Based on User Experience
- **Problem** — Measure infrastructure metrics (CPU, disk)
- **Consequence** — Metrics don't reflect what users care about
- **Solution** — Base SLO on observable user-facing metrics

---

## 7. Multi-Tier SLOs

### Why Multiple Tiers?
- **Different User Segments** — Different needs
- **Different Service Components** — Some more critical
- **Realistic Granularity** — Track what matters

### Example: E-Commerce Platform

#### Tier 1: Checkout (Most Critical)
- **SLO** — 99.99% availability, < 500ms latency
- **Reasoning** — Direct revenue impact
- **Budget** — Very small error budget

#### Tier 2: Search & Browse
- **SLO** — 99.9% availability, < 1s latency
- **Reasoning** — Important but not revenue critical
- **Budget** — Moderate error budget

#### Tier 3: Recommendations
- **SLO** — 99% availability, < 2s latency
- **Reasoning** — Nice to have, not critical
- **Budget** — Large error budget

### Composite SLO
- **Overall Service SLO** — Conservative of all components
- **Implementation** — If checkout fails, overall SLO fails
- **Rationale** — Service only as good as weakest link

---

## 8. SLO Documentation & Communication

### What Should Be Documented?

#### SLO Document Contents
1. **Overview**
   - Service name and description
   - Owner team
   - Last updated date

2. **SLOs**
   - Each SLO statement
   - Target percentage
   - Measurement method
   - Rationale

3. **SLIs**
   - Definition of each indicator
   - How it's measured
   - Data collection method
   - Tools/dashboards

4. **Related SLAs**
   - Customer-facing commitments
   - Penalties/SLA credits
   - Exclusions/exceptions

5. **Error Budget**
   - How much budget available
   - How budget is tracked
   - Escalation procedures

6. **Review Schedule**
   - When SLOs are reviewed
   - Process for updating
   - Stakeholder approval

### Communicating SLOs
- **To Users** — Set expectations
- **To Teams** — Align on priorities
- **To Customers** — SLA commitments
- **To Executives** — Risk and investment needs

---

## 9. Measuring & Monitoring SLOs

### Measurement Methods

#### Option 1: Client-Side Measurement
- **Approach** — Measure from user device/client
- **Advantages** — True user experience
- **Disadvantages** — Hard to distinguish service vs network issues
- **When to Use** — Mobile apps, frontend services

#### Option 2: Server-Side Measurement
- **Approach** — Measure at service boundary
- **Advantages** — Complete visibility, easy to implement
- **Disadvantages** — Doesn't account for network issues
- **When to Use** — Most backend services

#### Option 3: Synthetic Monitoring
- **Approach** — Automated tests that simulate users
- **Advantages** — Consistent measurement, can probe specific flows
- **Disadvantages** — May not represent real usage
- **When to Use** — Critical paths, uptime monitoring

#### Option 4: Hybrid Approach
- **Approach** — Combine multiple methods
- **Advantages** — More complete picture
- **Implementation** — Use multiple signals, weight accordingly
- **Best Practice** — Recommended for critical services

### SLO Dashboards
- **Real-Time Tracking** — Current budget status
- **Historical Trends** — Progress over time
- **Budget Forecast** — Projected status end-of-period
- **Incident Markers** — When incidents occurred
- **Alerts** — Warning when approaching depletion

---

## 10. Key Takeaways

### SLO Fundamentals
1. **SLOs bridge user expectations and system design**
2. **Based on SLIs (measurable, user-centric metrics)**
3. **Set conservatively (should exceed 95% of time)**
4. **Derived from business requirements**
5. **Enable data-driven operational decisions**

### Implementation Insights
1. **Start simple** — Few SLOs per service
2. **Base on user experience** — Not infrastructure metrics
3. **Make achievable** — Set targets you can realistically meet
4. **Document thoroughly** — Everyone should understand
5. **Review regularly** — Update as business needs change

### Operational Benefits
1. **Alignment** — Dev and ops teams share goals
2. **Transparency** — Clear expectations and trade-offs
3. **Prioritization** — Error budgets guide decisions
4. **Learning** — Track performance over time
5. **Communication** — Common language with stakeholders

---

## 11. Discussion Questions & Study Points

- [ ] What are 3 SLIs for your primary service?
- [ ] Should they have different SLOs? Why?
- [ ] How would you measure these SLIs?
- [ ] What SLO would be achievable? Ambitious?
- [ ] How would different SLOs affect error budget?
- [ ] How would you communicate SLOs to users?
- [ ] What SLO would justify 99.99% infrastructure investment?
- [ ] How would you detect SLO violations?

---

**Created**: 2026-06-08  
**Source**: Site Reliability Engineering: How Google Runs Production Systems - Chapter 4  
**Next**: Chapter 5 - Eliminating Toil  
**Related**: [[Chapter_3_Study_Notes.md]] - Error Budgets and Risk
