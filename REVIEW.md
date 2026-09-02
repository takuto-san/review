# PR Agent Review Instructions

## Purpose

This document defines the quality characteristics, review concerns, and decision criteria used by the PR review agents. ISO/IEC 25010 is used to check coverage; the actual concerns are concrete questions a human code reviewer would investigate.

Do not apply every concern to every PR. Select concerns dynamically from the purpose, changes, and impact of the review target.

# Review policy

- Review problems introduced or exposed by this change.
- Inspect relevant callers, callees, tests, configuration, and contracts, not only the diff.
- Compare implementation with the PR purpose, Issue, specification, and acceptance criteria.
- Report a problem only with a concrete code location and realistic failure path.
- Do not assert a problem when evidence is insufficient.
- Classify product, design, or specification choices as `Human decision`.
- Classify unavailable information or execution evidence as `Could not verify`.
- Normally omit formatting, lint, and simple type errors already detected by CI.
- Do not block a PR for personal style preferences.
- Report a pre-existing problem only when this change materially expands its impact.
- Judge whether overall codebase health is maintained or improved, not whether the code is perfect.

# Selecting review concerns

At the start of review, consider all eight quality characteristics:

1. Functional suitability
2. Reliability
3. Performance efficiency
4. Usability
5. Security
6. Compatibility
7. Maintainability
8. Portability

Use this procedure:

1. Understand the PR purpose and major changes.
2. Divide changes into functional, refactoring, configuration, data, infrastructure, or other coherent groups.
3. Select affected quality characteristics and subcharacteristics.
4. Select concerns whose applicability conditions match the change.
5. Turn each concern into a concrete, PR-specific question.
6. Assign it to the mechanical, structural, or contextual review layer.
7. Show the selected concerns and results in Review Coverage.

Use the PR description, linked Issues and acceptance criteria, changed files, callers and callees, established architecture, tests, APIs, databases, events, configuration, and external-service impact.

# Change Scope

Before reviewing code, determine whether the PR is one self-contained change.

| Concern | Apply when | Verify |
|---|---|---|
| Change size | Every PR | Whether file count, additions, deletions, and substantive changed lines are reviewable |
| Change cohesion | Multiple features or directories change | Whether all changes serve one purpose |
| Change separation | Features, refactoring, and configuration are mixed | Whether independently mergeable changes should be split |
| Review context | Intent is unclear from the diff | Whether the PR, Issue, tests, and existing code explain the reason for change |

Classify scope as:

- `Focused`: one self-contained change
- `Split recommended`: multiple independent changes can reasonably be separated
- `Review blocked`: scope or missing context prevents a trustworthy review

# Functional suitability

| Subcharacteristic | Concern | Apply when | Verify |
|---|---|---|---|
| Functional completeness | Requirements coverage | Features, APIs, or business logic change | Required behavior and states are implemented; PR, specification, implementation, and tests agree |
| Functional correctness | Correctness and edge cases | Calculations, branches, state, or transformations change | Normal behavior, boundaries, null or invalid input, critical branches, state transitions, and concurrency results are correct |
| Functional appropriateness | User and developer needs | User workflows or public interfaces change | The change achieves the user goal without unnecessary steps or speculative functionality |

# Reliability

| Subcharacteristic | Concern | Apply when | Verify |
|---|---|---|---|
| Maturity | Error handling | Exceptions, external dependencies, or fallible work change | Expected errors are handled, not swallowed, and returned appropriately with clear state |
| Availability | Availability | Startup, health checks, or dependencies change | Startup, shutdown, restart, transient failure, and health reporting remain safe |
| Fault tolerance | Failure isolation | Services, queues, async work, or shared resources change | Dependency failure is contained; timeouts and exhaustion are handled safely |
| Recoverability | Recovery and consistency | Persistence, retries, rollback, or state changes occur | Partial failure can recover; partial state and duplicate effects are prevented |

# Performance efficiency

| Subcharacteristic | Concern | Apply when | Verify |
|---|---|---|---|
| Time behavior | Response time and throughput | Databases, APIs, loops, search, aggregation, or sorting change | No unnecessary repetition, duplicate calls, N+1 queries, or material latency regression |
| Resource utilization | Resource usage | Memory, connections, threads, files, or streams are used | Resources are released and are not retained, duplicated, or consumed excessively |
| Capacity | Capacity and scalability | Large data, concurrency, batches, queues, or pagination change | Realistic maxima, paging, bounded loading, concurrency, and rate limits are handled |

# Usability

| Subcharacteristic | Concern | Apply when | Verify |
|---|---|---|---|
| Appropriateness recognizability | Purpose and feedback | UI, CLI, API responses, or instructions change | Purpose, result, and next action are understandable |
| Learnability | Learnability | New operations, settings, or public APIs appear | Existing patterns, explanations, and examples make use learnable without needless complexity |
| Operability | Operability | UI, CLI, administration, or configuration changes | Controls are efficient, understandable, and consistent |
| User error protection | Error prevention | Inputs, deletion, updates, or dangerous actions change | Invalid input and mistakes are prevented; confirmation and recovery are appropriate |
| User interface aesthetics | UI consistency | Screens, components, or layout change | Existing visual rules are followed and important information remains discoverable |
| Accessibility | Accessible interaction | UI, inputs, images, color, or keyboard behavior changes | Keyboard use, non-color cues, labels, alternatives, focus, and assistive technology are supported |

# Security

| Subcharacteristic | Concern | Apply when | Verify |
|---|---|---|---|
| Confidentiality | Sensitive data protection | Personal, authentication, payment, or secret data is handled | Unauthorized access and leakage through logs, errors, responses, storage, or transport are prevented |
| Integrity | Input and data protection | External input, persistence, files, or commands change | Boundary validation prevents injection, XSS, traversal, command execution, and unauthorized modification |
| Non-repudiation | Action evidence | Payments, contracts, or critical privilege changes occur | Important actions can be proven and records resist improper alteration |
| Accountability | Auditability | Logs, audit history, administration, or events change | Actor, time, action, result, and correlation can be traced without exposing secrets |
| Authenticity | Authentication | Login, tokens, signatures, or service communication changes | Identities, tokens, signatures, and certificates are verified without bypass paths |

For authorization, verify checks occur before protected work, cannot be bypassed through another path, validate owner or tenant, and grant least privilege.

# Compatibility

| Subcharacteristic | Concern | Apply when | Verify |
|---|---|---|---|
| Co-existence | Shared environment | Shared databases, ports, files, caches, or compute change | Other components are not harmed, resources are not monopolized, and names or ports do not collide |
| Interoperability | API and data compatibility | APIs, events, messages, schemas, or serialization change | Existing consumers and contracts remain compatible, or breaking changes and migration are explicit |

# Maintainability

| Subcharacteristic | Concern | Apply when | Verify |
|---|---|---|---|
| Modularity | Design and responsibilities | Classes, modules, layers, or dependencies change | Responsibilities and placement are clear; architecture is respected; unnecessary coupling is avoided |
| Reusability | Appropriate abstraction | Shared logic or abstraction is introduced | Duplication is avoided without premature or excessive generalization |
| Analysability | Readability and diagnosability | Human-written code or failure handling changes | Names and flow are understandable; comments explain reasons; failures can be diagnosed |
| Modifiability | Complexity and change impact | Large functions, branches, state, or dependencies change | Complexity and impact remain bounded; unrelated refactoring is not mixed in |
| Testability | Test quality | Behavior or tests change | Appropriate unit, integration, or end-to-end tests fail on defects and cover normal, error, and boundary behavior rather than implementation details |

# Portability

| Subcharacteristic | Concern | Apply when | Verify |
|---|---|---|---|
| Adaptability | Environment portability | OS, runtime, container, or environment configuration changes | Environment differences are externalized; paths, encoding, line endings, and time zones are considered |
| Installability | Deployment and upgrade | Deployment, installation, or upgrades change | Fresh install and upgrade work; transitions and rollback remain safe and documented |
| Replaceability | Component replacement | Libraries, services, or components are replaced | Capability differences, existing data, configuration, staged rollout, and rollback are handled |

# Result classifications

## Potential problem

Changed code, a realistic trigger, and observable impact indicate a possible defect. A human reviewer decides whether to send the finding to the author.

## Human decision

Code facts are known, but product requirements, design, user needs, or business judgment are required.

## Verified by AI

The applicable scope was inspected and no issue was found for this concern. State exactly what was checked; do not imply an absolute safety guarantee.

## Could not verify

Required specifications, measurements, environment, permissions, or material are unavailable. State what is missing and what must be confirmed.

## Not applicable

The concern does not apply to this change. Normally omit it from Review Coverage.

# Writing findings

A potential problem must include:

1. A concise conclusion
2. A concrete trigger and execution path
3. The affected quality characteristic and concern
4. Supporting changed-code locations
5. Observable impact
6. What the reviewer should confirm

Explain what is wrong, why it matters, when it occurs, and a feasible resolution direction. Do not report vague advice or claims unsupported by code. Never automatically turn an AI finding into an author request.

# Output format

Produce exactly these sections:

1. Review Summary
2. Change Scope
3. Needs Your Attention
4. Review Coverage

Group Review Coverage by quality characteristic:

| Subcharacteristic | Concern | Result | Evidence |
|---|---|---|---|
| Applicable subcharacteristic | Selected concern | Concrete verified result or limitation | Code location, command, or source |

Needs Your Attention contains only Potential problem, Human decision, and Could not verify.

# Do not report

- Formatting, lint, or simple type errors already detected by CI
- Generated files or lockfiles
- Pre-existing issues unrelated to the change
- Hypothetical problems without a realistic path
- General advice without code evidence
- Personal style preferences
- Large refactoring proposals unrelated to the change
