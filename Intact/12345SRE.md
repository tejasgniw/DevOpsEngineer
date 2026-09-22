# DevOps vs SRE

DevOps and SRE are complementary frameworks that aim to accelerate software delivery while maintaining high product reliability. DevOps focuses on the end-to-end application lifecycle, utilizing a collaborative, multidisciplinary approach to bridge the gap between development and operations through automation and continuous delivery. In contrast, SRE (Site Reliability Engineering) focuses specifically on operational resilience, emphasizing the stability, scalability, and performance of the production environment

# SLI, SLO, SLA, Error budget

- Service Level Indicator (SLI): A quantitative measure of the actual performance of a service, such as uptime or response time. Its significance is providing the concrete data needed to track compliance against internal and external goals.

- Service Level Objective (SLO): An internal target for a specific metric over a set time window, such as achieving 99.9% uptime over 30 days. Its significance is aligning teams around shared benchmarks to resolve issues before they impact the customer experience.

- Service Level Agreement (SLA): A formal commitment between a provider and a customer that outlines expected service levels and the consequences, like financial penalties, for failing to meet them. Its significance is establishing accountability and defining the legal or business relationship regarding service quality.

- Error Budget: The allowable room for error or downtime (e.g., 4 minutes of downtime for a 99.99% SLO) before a service objective is missed. Its significance is empowering teams to balance innovation and reliability by clarifying how much risk they can take during experimentation.

# Incident Response Lifecycle

```
Preparation
    |
Detection and Analysis
    |
Containment, Eradication, and Recovery
    |
Post-Event Activity
```

## Preparation

The Preparation phase covers the work an organization does to get ready for incident response, including establishing the right tools and resources and training the team. This phase includes work done to prevent incidents from happening. e.g. having monitoring tools or alerts in place

## Detection and Analysis

Accurately detecting and assessing incidents is often the most difficult part of incident response for many organizations. e.g. checking cluster to see which resources are affected and how big the incident is.

## Containment, Eradication, and Recovery

This phase focuses on keeping the incident impact as small as possible and mitigating service disruptions.

e.g. Cordoning nodes for containment, temporarily stopping HPA for eradication, and fixing the issue and restoring services for recovery.

## Post-Event Activity

Learning and improving after an incident is one of the most important parts of incident response and the most often ignored. In this phase the incident and incident response efforts are analyzed. The goals here are to limit the chances of the incident happening again and to identify ways of improving future incident response activity.

e.g. post-incident review, updating playbooks, building required tools and testing fixes.

# “You’re an DevOps, why are you considering a SRE role?”

Throughout my career in the startup project, I learned how to build infra, CI/CD pipelines, infrastructure as code, cloud platforms. Most importantly learned how to keep those infra alive during the incidents, outages, SLAs, and real user impact.

DevOps taught me how to ship.
SRE taught me how to sleep after shipping.

Over time, the titles mattered less than the responsibility. Whether it’s called DevOps or SRE, the real work is the same:

- Automating the manual tasks
- Designing infra that don’t fall over under pressure
- Learning from failures instead of hiding them
- Making engineers and non-engineers trust the platform

So when I interview for a SRE role, I’m not stepping away from DevOps,
I’m bringing DevOps into reliability.

And when I work as an SRE, I’m not just firefighting,
I’m building better systems so fires don’t happen again.

Different titles. Same mindset. Same goal.