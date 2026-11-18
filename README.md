# Building confidence in AI Model
Ensuring AI models are trained on dataset that accurately reflects real RAN and/or Core network conditions.
How the generated datasets are realistic and trustworthy for ingestion to AI Models or LLMs so that they can behave in real time as we expect for outcome?

Live networks don't always provide enough examples of unusual events, such as interference, 
unexpected traffic patterns, handover failures or network outages, 
making it difficult to properly train and test AI models.

## Simulation perspective
How to combine synthetic scenarios with real network data, allowing teams/designers to generate __realistic training datasets__
and safely reproduce varrying network conditions.

## Pre-deployment validation playbook for AI in the RAN

Confidence is evidence that a specific model version behaves acceptably for a defined use case, on a defined network, under normal and degraded conditions. A high model score is not a release decision. Before a model controls live traffic, establish independently that it makes useful decisions, protects service, and has a credible business case. Record untested conditions and keep the baseline policy in control there.

### 1. Define the operating envelope and pass criteria

- Document the decision, inputs, outputs, timing deadline, allowed actions, control authority, dependencies, and fallback. Separate **observe** (read-only), **recommend** (human approval), and **act** (automated control); require stronger evidence as authority and potential impact increase.
- State the supported bands, bandwidths, duplex modes, radio configurations, cell types, UE capabilities, vendors, software versions, traffic and mobility ranges, and service classes. Define how unsupported, stale, or unknown conditions are detected and handled.
- Compare against the current operator policy with identical guardrails and inputs where possible, and against a simple heuristic or statistical model. Set a minimum worthwhile improvement, non-inferiority margins for protected service KPIs, hard safety limits, test duration, sample size, and stop rules before tuning.
- Map every critical requirement to a scenario, metric, threshold, evidence artifact, and owner. Treat absent evidence in a critical scenario as a coverage gap, not a pass.

| Placement or use case | Examples of behavior to validate |
| --- | --- |
| PHY: beam, CSI, or link adaptation | BLER, beam failures and recovery, spectral efficiency, and deadline misses under changing channel conditions; retain a conventional control path. |
| MAC: scheduling or resource allocation | Cell-edge and per-service throughput, HARQ behavior, queue delay, fairness, and deadline misses under bursty loads and mixed QoS. |
| RLC / PDCP, where applicable | Buffer and discard behavior, retransmissions, packet loss and ordering, and tail latency across transport and mobility conditions. |
| RRC / mobility | Handover success, interruption time, radio link failures, and ping-pong rates across speeds, cell boundaries, and neighbor combinations. |
| RIC / SON: optimization or energy saving | Neighbor-cell impact, oscillation and convergence, coverage, wake-up delay, and energy per delivered unit of traffic alongside absolute energy use. |

### 2. Establish that the data and simulator represent the network

- Inventory and version the telemetry, labels, feature definitions, configurations, and collection windows. Verify units, time alignment, counter resets and denominators, missingness, sampling bias, and label quality. Check that each feature would be available by the real decision deadline and that training and serving compute it the same way.
- Split data before fitting, simulation calibration decisions, or augmentation. Use time-forward holdouts and hold out entire sites or correlated cell clusters, devices, incidents, or vendor configurations as applicable. Keep the final test set locked. Deduplicate before splitting and prevent leakage from future KPIs, overlapping windows, or synthetic variants of test traces.
- Use approved, minimized telemetry; pseudonymize sensitive identifiers, restrict access and retention, and avoid subscriber payloads unless specifically authorized and necessary. Document collection gaps and what cannot be inferred.
- Calibrate a system/link simulator or digital twin against approved real traces. Compare joint distributions, time dynamics, handover and interference patterns, and KPI tails on separate real traces, not just averages on calibration sites. Record where fidelity is insufficient; report results on real-only, synthetic-only, and blended sets separately. Do not treat synthetic samples as independent evidence of real-world uplift.
- Build a coverage matrix spanning busy and quiet hours, urban/rural and indoor/outdoor sites, cell edge, UE mixes, mobility, traffic profiles, and seasonal or special-event peaks. Add rare and combined cases in the lab: congestion with handovers, neighbor outages with load shifts, interference or coverage degradation, telemetry delay/loss, and equipment or software changes. Use systematic scenario sweeps, boundary tests, and multiple random seeds; separate plausible scenarios from out-of-envelope stress tests.

### 3. Validate offline performance, uncertainty, and control behavior

- Match metrics to the decision: precision/recall at the action threshold for alerts, forecast error and interval coverage for predictions, and service KPIs plus constraint violations for control. Compare at the same operating point or action budget. Report denominators, tail percentiles, and results by site, band, device, service class, mobility, and load, including the worst supported slices.
- Quantify uncertainty in two different ways. For predictive confidence, use reliability plots and Brier score for probabilities or empirical coverage for prediction intervals; check calibration by slice and under shift. For KPI uplift, report confidence intervals that account for correlated cells and time windows (for example, cluster or block bootstrap). Define a minimum sample size per critical slice; do not treat zero observed failures as proof of zero risk.
- Evaluate abstention on missing, stale, noisy, and out-of-distribution inputs. Plot error versus coverage, measure missed detections and false alarms for drift checks where labeled shifts exist, and verify that fallback preserves service KPIs. Calibrate thresholds on validation data, not the locked test set. Check that model decisions change sensibly when relevant inputs change and remain stable under benign measurement noise.
- Replay representative and incident traces with recorded event order and timing, using the release model and the baseline side by side. Replay verifies observation and decision behavior, not how the network would respond to a different action. For policies evaluated from logged actions, use off-policy estimates only where action coverage and logging assumptions hold; mark unsupported actions as unevaluable.
- In closed-loop simulation, let actions change future traffic, radio state, and neighbor load. Check feedback stability, recovery time, action limits, and interactions with existing SON/RIC controllers; include conflicting policies, repeat decisions, and delayed measurements. Where feasible, verify hard invariants on the action guardrails and complement average-case tests with worst-case scenario search.
- If an LLM is used for operator recommendations, test factual grounding against approved runbooks and telemetry, stale or conflicting evidence, abstention, structured output validation, and human approval. Keep free-form output out of direct RAN control paths.

### 4. Prove integration and recovery on the target stack

- Progress from software-in-the-loop to hardware-in-the-loop and a production-like staging RAN. Use UE and traffic emulators, channel emulators or authorized shielded RF testing, and representative RU/DU/CU, edge hardware, and controller versions. Verify applicable 3GPP and O-RAN interface behavior for the deployed releases, including contracts, timing, partial upgrades, and multi-vendor interoperability.
- Run end-to-end peak, burst, and soak tests on the packaged model, including quantized or accelerated builds. Measure full telemetry-to-action latency and jitter (including tail latency and deadline-miss rate), synchronization, CPU/GPU/memory, power, and any impact on other workloads. Check cold start, restart, and upgrade behavior, not just average inference speed.
- In an isolated testbed, exercise missing or reordered telemetry, clock drift, failed dependencies, overloaded inference, and disconnected controllers. Verify input/schema validation, model provenance and integrity, least-privilege access, and auditable model decisions without logging sensitive payloads.
- Keep operator-owned action bounds, rate limits, expiry, and conflict arbitration outside the model. Rehearse manual override, automatic fallback, and rollback to the known-good policy, including removal of persistent actions and reconciliation of controller state. Measure time to detect, stop, and restore service; verify the fallback still works when the model or monitoring is unavailable.

### 5. Separate pre-traffic evidence from production exposure

| Stage | What it proves and what it does not |
| --- | --- |
| Offline, simulation, and isolated staging | No production exposure. Validate envelope coverage, stability, integration, and recovery. Passing permits an operator review; it does not authorize control of live traffic. |
| Read-only shadowing, only after operator approval | Evaluate on mirrored live telemetry with no actuation permissions and isolated or bounded resources. Check live feature parity, freshness, latency, guardrail rejects, abstentions, and disagreements with the baseline. Shadowing cannot measure the causal effect of actions never applied, and is not required if live data access is disallowed. |
| Limited live canary, only after pre-deployment sign-off | This is **production deployment**, not a pre-traffic test. Limit to approved sites, actions, and exposure windows; exclude protected services or sites as required. Use independent abort monitoring, an on-call owner, and rehearsed rollback. Expand only following explicit review. |

For a live comparison, randomize by independent cell clusters when feasible to account for mobility and interference spillover. Otherwise use matched controls or a switchback with washout periods where carryover can be controlled. Account for load, seasonality, concurrent changes, and delayed outcomes. Monitor leading signals (telemetry freshness, inference deadlines, fallback rates) and outcomes (service KPIs and neighbor impact). Stop on a guardrail breach or loss of monitoring; inconclusive results are not grounds for expansion.

### 6. Tie the release decision to service protection and ROI

| Gate | Required evidence |
| --- | --- |
| Service protection | Operator-defined hard limits and non-inferiority margins met for availability, drops, mobility, latency, protected services, and cell-edge users as applicable. Compare each critical slice with the baseline, not just the network average. |
| Model utility | The predeclared minimum gain on the primary KPI is supported by uncertainty bounds appropriate to the stage; known failure modes, abstention behavior, and exclusions are documented. |
| Operational readiness | Target-platform budgets met; alerts, overrides, and rollback rehearsed; versions of model, data, feature pipeline, runtime, configuration, and test evidence recorded. |
| Economic case | Approved cost and benefit assumptions, conservative/base/upside estimates, sensitivity to adoption and traffic, and a plan to measure realized returns against the baseline. |

- Translate KPI changes into measurable value: kWh saved times the applicable tariff, attributable capacity deferral where finance agrees it is realizable, or incident handling hours avoided times a fully loaded rate. Include compute, power, licensing, telemetry, integration, testing, monitoring, on-call support, retraining, and rollout costs. Avoid double counting and never trade a hard service guardrail for savings.
- For a stated horizon, `ROI = (attributable benefits - incremental costs) / incremental costs` when incremental costs are positive. Report expected payback and key assumptions. Label offline and simulation estimates as forecasts; update them with controlled live evidence before claiming realized ROI.
- Keep one release record with the operating envelope, baseline, test matrix, results and confidence intervals, open risks, abort thresholds, rollback evidence, and named RAN operations, model, privacy/security, and business approvers. Re-run affected gates when model, features, firmware, interfaces, control policies, or deployment scope changes.

Reference points: [3GPP TR 38.843 (AI/ML for the NR air interface study)](https://portal.3gpp.org/desktopmodules/Specifications/SpecificationDetails.aspx?specificationId=3983), [O-RAN specifications](https://www.o-ran.org/specifications), and the [NIST AI Risk Management Framework](https://www.nist.gov/itl/ai-risk-management-framework). Use the versions applicable to the operator stack; these references do not replace operator acceptance criteria.

### 7. Validate adaptation across network and climatic conditions

Classify **network health** independently of **weather**. A clear day can have severe congestion, and a storm may leave service healthy. Weather is context, not a command to change the network. Define healthy, degraded, critical, and recovering states per site, band, and service using operator thresholds for radio quality (for example, SINR and BLER), load, handover failures, latency/loss, and transport, power, and equipment alarms.

| Network state | Scenarios to test | Expected behavior |
| --- | --- | --- |
| Healthy | Normal and peak demand; stationary and high-mobility UEs; good radio quality under both clear and adverse weather | Stay on the baseline unless a bounded action has a demonstrated benefit. Avoid churn and false interventions, including during false weather alerts. |
| Degraded | Weak coverage, interference, congestion, partial backhaul loss, or a neighbor outage, with and without adverse weather | Distinguish radio, load, and transport symptoms before acting. Test a bounded action appropriate to the cause (for example, steering only to a neighbor with spare capacity). Measure service and neighbor KPIs against the baseline. |
| Critical or unknown | Site or power outage, correlated failures, unsupported conditions, or stale/conflicting network telemetry | Stop discretionary actions, keep operator-approved resilience mechanisms in control, and alert operations. Verify emergency and priority-service protections, as applicable; do not assume the model can restore unavailable infrastructure. |
| Recovering | Storm clears but backhaul remains impaired; repeated link flaps; traffic returns in a surge | Require independent evidence of stable service before resuming automation. Test entry/exit thresholds, dwell times, cooldowns, gradual re-entry, and repeat degradation so the model does not oscillate. |

| Climatic scenario | What to reproduce in replay, simulation, or isolated staging |
| --- | --- |
| Clear, dry, or seasonally typical | Establish a matched baseline by site, load, and time of day. Include failures unrelated to weather so the model does not learn that clear means healthy. |
| Heavy rain, hail, or thunderstorms | Test path-specific rain fade on susceptible higher-frequency access and microwave backhaul links, and separate it from storm-related power or transport failures. Check frequency, path length, polarization where relevant, and measured link margin; do not apply the same attenuation to every band or path. |
| Snow, freezing rain, or icing | Reproduce accumulation or ice on susceptible antennas or radomes, degraded links, and delayed recovery even after precipitation ends. |
| Fog, high humidity, or coastal conditions | Test measured local effects such as condensation or relevant propagation changes; account for frequency and path rather than assuming fog degrades every link. |
| Heat waves, cold snaps, or prolonged extreme temperatures | Emulate equipment thermal alarms or derating, reduced edge-compute headroom, and impaired backup power where observed. Recheck inference deadlines and fallback with the reduced resources. |
| High winds, flooding, or other locally relevant hazards | Model antenna alignment changes where plausible and correlated outages of neighboring cells, shared power, and backhaul. Test traffic displacement and verify that adaptation does not overload surviving sites. |
| Slow seasonal changes and compound events | Test wet/dry foliage, seasonal demand, and changed outdoor/indoor traffic patterns where relevant. Include combinations such as rain fade plus peak load, heat plus equipment derating, and a storm followed by unstable recovery. |

- **Build a coverage matrix.** Cross health state and weather severity with region, season, band, site type, backhaul type, traffic, and mobility. Prioritize combinations by likelihood and service impact; mark untested combinations explicitly. Hold out entire events and affected regions where feasible, not random samples from the same storm. Use real incidents to calibrate simulations and test conditions from onset through recovery.
- **Validate weather data as optional context.** Align observations and forecasts to the affected site or link path and use only values available at decision time. Test false alarms, missed events, stale or missing feeds, coarse spatial coverage, and contradictory readings. Compare the same model with and without weather inputs against the operator baseline. If weather data is unreliable but network telemetry is healthy, use a validated telemetry-only mode or the baseline; never relax safety limits.
- **Keep actions proportional to the evidence.** A forecast may prompt extra monitoring or operator review. Require network evidence for automated service-affecting actions; for example, suspend discretionary cell sleep when surrounding capacity is impaired, or shift traffic only if coverage, transport, and neighbor headroom are verified. Test action bounds, expiration, and automatic rollback if service worsens.
- **Gate release by condition, not by average.** Measure false interventions in healthy weather-affected sites, time to detect degradation and restore service, tail latency, drops, priority-service performance, neighbor impact, and rollback success. Require service guardrails in each supported critical slice. Keep unvalidated conditions on the baseline or in advisory mode. Estimate weather-related benefits using local event frequency and avoided service disruption where measurable, net of added telemetry, compute, and operational costs; claim savings only with attributable evidence.

For propagation modeling, see [ITU-R P.838 (rain specific attenuation)](https://www.itu.int/rec/R-REC-P.838/en) and [ITU-R P.530 (terrestrial line-of-sight links)](https://www.itu.int/rec/R-REC-P.530/en). Check applicability to the actual band, path, and equipment, and calibrate against operator measurements.

### 8. Do-not-skip checks before autonomous network control

**Verification** asks whether the packaged model and controller obey their specifications. **Validation** asks whether their decisions protect service and improve the intended KPI in the supported network conditions.

| Do not skip | Simulate or verify | Required evidence |
| --- | --- | --- |
| Define the operating envelope and baseline | Run the current operator policy and candidate on matched traffic, topology, and mobility. Include an unsupported configuration. | Set the minimum useful gain, protected KPI limits, and decision deadline before testing. Unsupported conditions stay on the baseline. |
| Validate telemetry and uncertainty | Replay held-out site or incident traces. Inject missing, stale, and out-of-order telemetry and unfamiliar load. | Confirm feature parity with production and verify abstention or fallback before unsafe actuation. |
| Verify action guardrails and timing | Submit out-of-bounds, duplicate, or expired actions; add inference delays and conflicting controller requests. | Independent guardrails reject invalid actions, and late decisions cannot change live state. Confirm end-to-end timing on the target stack. |
| Validate closed-loop stability and service impact | Combine a traffic surge with a neighbor or backhaul fault. Let the model's actions change subsequent network state. | No sustained oscillation or cascading overload. Compare the intended gain, tail latency, drops, and protected service KPIs against the baseline, including the worst supported slices. |
| Rehearse fallback and rollback | Disable the model or monitoring during an active change. Restore the known-good policy, then repeat the fault during recovery. | Demonstrate an independent stop and rollback path, reconcile outstanding actions, and restore service within the agreed recovery time. |
| Require reproducible evidence and operator sign-off | Repeat paired runs across multiple seeds and record model, simulator, scenario, and guardrail versions. Rehearse on isolated staging. | No live actuation if a hard limit is breached, critical evidence is missing, or a required gain is inconclusive. Keep untested conditions on the baseline. |

#### Recommended simulator: ns-3

Start with [ns-3 (GitHub mirror)](https://github.com/nsnam/ns-3-dev-git) for repeatable, packet-level closed-loop tests. Add [5G-LENA](https://5g-lena.cttc.es/) for 5G NR radio behavior, and pin a compatible pair of versions. Connect the packaged model and its action guardrails to a simulated telemetry and control loop; ns-3 supplies the network behavior, not the model or a confidence score.

**First experiment: two-cell load steering.** Compare the current operator policy and the AI policy in separate runs with matched traffic, UE mobility, and random seeds. Run steady traffic, add a surge in one cell, reduce the neighboring cell's backhaul capacity, then restore it. Repeat with delayed telemetry and a model timeout. Record overload, packet loss, p95 latency, action reversals, and time to fall back and recover. Judge each run against the predeclared limits. Calibrate the simulator on approved operator traces, check its fidelity on separate held-out traces, and repeat timing and rollback checks in isolated staging. Simulation alone does not authorize deployment.
