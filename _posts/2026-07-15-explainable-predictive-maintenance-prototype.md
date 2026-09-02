---
layout: post
title: "Building an Explainable Predictive-Maintenance Prototype for Industrial Generators"
subtitle: "How I combined verified technical documentation, physics-based residuals, and machine learning — while being explicit about what the prototype cannot yet prove."
excerpt: "A predictive-maintenance prototype for industrial diesel generators, built on synthetic telemetry — what it measured, and what it still cannot claim."
tags: [Predictive Maintenance, Machine Learning, Industrial IoT, Python, Engineering]
---

Over the past few months I built a predictive-maintenance prototype for industrial diesel generator sets. This post is a record of what it does, what I actually measured, and — the part I think matters more — everything it still cannot claim.

The objective was never to ship a product. It was to find out whether an explainable, honest foundation could be built at all: a system that shows its evidence, states its uncertainty, and refuses to answer when it has no basis to. That turned out to be a harder and more interesting problem than adding a model.

<nav class="post-toc" aria-labelledby="toc-heading">
  <h2 id="toc-heading">Contents</h2>
  <ol>
    <li><a href="#the-problem">The problem</a></li>
    <li><a href="#what-the-prototype-includes">What the prototype includes</a></li>
    <li><a href="#citations">Citations, and the layer I did not build</a></li>
    <li><a href="#expected-versus-actual">Expected versus actual</a></li>
    <li><a href="#what-i-tested">What I tested</a></li>
    <li><a href="#limitations">Limitations I designed around</a></li>
    <li><a href="#what-next">What I would do next</a></li>
    <li><a href="#closing">Closing reflection</a></li>
  </ol>
</nav>

## The problem {#the-problem}

A generator fault is usually discovered in one of two ways: an alarm trips, or the machine stops. Both are late. By the time coolant temperature crosses a shutdown threshold, whatever caused it has been developing for days or weeks.

The obvious fix — set the threshold lower — does not work, and the reason is worth sitting with. **A fixed threshold assumes a reading means the same thing in every condition, and it doesn't.** Ninety-two degrees of coolant temperature at ninety percent load on a hot afternoon is unremarkable. The same ninety-two degrees at twenty percent load on a cool morning means something is wrong with the cooling system. Lower the threshold enough to catch the second case and you generate nuisance alarms across every hot afternoon until someone disables it. That is how condition monitoring dies in practice: not from being wrong, but from being ignored.

There is a second problem, quieter but just as limiting. Engineers working on these machines need answers from product documentation — service intervals, sensor locations, alarm meanings — and they need those answers to be traceable to a source. An assistant that produces a confident, plausible, unsourced number is worse than no assistant, because someone will act on it.

Both problems have the same shape. The useful system is not the one that produces the most output. It is the one whose output you can check.

## What the prototype includes {#what-the-prototype-includes}

The prototype is a pipeline. Each stage produces something the next stage can use, and each stage can be inspected on its own.

<div class="arch">
  <ol>
    <li>
      <p class="arch-t">Engineering knowledge base</p>
      <p class="arch-d">A component model of the machine — 14 systems, roughly 260 sub-components, the failure physics and how faults propagate between subsystems. Every statement carries a provenance tag recording where it came from.</p>
    </li>
    <li>
      <p class="arch-t">Cross-linked registers</p>
      <p class="arch-d">164 telemetry signals, 116 alarms, 65 failure modes scored for risk, and 46 maintenance tasks. They reference each other, and the build fails if a reference does not resolve.</p>
    </li>
    <li>
      <p class="arch-t">Telemetry quality checks</p>
      <p class="arch-d">Nine validation checks — gaps, stuck sensors, out-of-range values — run before anything else. A model fed bad data still produces confident numbers, which is the failure mode worth guarding.</p>
    </li>
    <li>
      <p class="arch-t">Physics residuals and normalisation</p>
      <p class="arch-d">Each signal is compared against what it should be at the current load, ambient temperature and — for oil pressure — oil viscosity. The residual, not the raw value, is what gets modelled.</p>
    </li>
    <li>
      <p class="arch-t">Anomaly detection and fault typing</p>
      <p class="arch-d">An isolation forest scores the whole residual vector and reports which signals drove the score. A classifier attempts to name which fault mode it resembles.</p>
    </li>
    <li>
      <p class="arch-t">Risk, diagnosis and trend indication</p>
      <p class="arch-d">Attribution maps to subsystems, subsystems map to the failure-mode register, and time-to-threshold is estimated only where a genuine monotonic trend exists.</p>
    </li>
  </ol>
</div>

Everything is written in the Python standard library with no external dependencies — no scikit-learn, no charting library. That sounds like a strange constraint until you see the deployment target: a gateway box sitting next to a generator inside a plant network, where installing a package is a procurement conversation rather than a command.

## Citations, and the layer I did not build {#citations}

I want to be careful here, because this is the part of the project most easily overstated.

What exists is a **provenance discipline**. Every factual statement in the knowledge base carries a tag recording its basis:

<div class="legend">
  <div class="legend-row"><span class="chip chip-verified">M:p.xx</span><span>Stated in a source document, with the page cited.</span></div>
  <div class="legend-row"><span class="chip chip-verified">DERIVED</span><span>Computed from a cited value, with the arithmetic shown.</span></div>
  <div class="legend-row"><span class="chip chip-assumed">ENG</span><span>Standard engineering principle. Not from any manual.</span></div>
  <div class="legend-row"><span class="chip chip-assumed">ASSUMED</span><span>A working assumption, stated along with what breaks if it is wrong.</span></div>
  <div class="legend-row"><span class="chip chip-gap">GAP</span><span>The source is silent and it matters. Logged, not papered over.</span></div>
</div>

This is not decoration. It is the difference between a knowledge base and a plausible-sounding summary, and it is what makes the rest of the work auditable — anyone can ask "where did this number come from?" and get an answer rather than a shrug.

**What does not exist is a document-retrieval layer.** There is no search over manufacturer documents, no citation-answering assistant, no semantic index. I wrote the ingestion script for it. It has never run, for a simple reason: no service manual was ever supplied to the project.

That gap propagated everywhere. Alarm setpoints, maintenance intervals, controller register addresses — all of them are currently engineering estimates rather than manufacturer values, and every one of them is tagged as such. It has been the project's gating dependency since the first week.

<div class="callout">
  <p>An assistant that admits “the source does not contain this information” is safer than one that invents an answer. The same applies to the engineer building it — the honest move was to tag 164 signals as unverified rather than fill them in from plausible-looking numbers.</p>
</div>

I could have built the retrieval layer against public documents and called it done. I decided that a citation system pointed at sources that don't actually specify this machine would be worse than no citation system, because it would look authoritative. That call is still the one I'd defend most strongly.

## Expected versus actual {#expected-versus-actual}

The core idea is small enough to state in a sentence: **compare what the machine is doing against what it should be doing at this operating point, and model the difference.**

Take coolant temperature again. Instead of asking "is this above 98 degrees?", the pipeline asks "given the current load and the ambient temperature, what should this reading be — and how far off is it?" A residual near zero means the machine is behaving as its conditions predict. A residual drifting upward over days means something is degrading, even while the absolute value stays comfortably inside the alarm band.

A few consequences follow from building it this way.

**The evidence is interpretable.** When the system flags something, it can say which residual moved and in which direction, which maps to a subsystem, which maps to a documented failure mode. That chain is inspectable end to end. An alert that says "anomaly score 0.83" and nothing else gets ignored after its second false alarm.

**Slow degradation shows up in the trend, not the reading.** Rolling averages and slopes over hours and days catch things a single sample cannot.

**Time-to-threshold is only estimated where it means something.** Remaining-life estimation requires a monotonic degradation path, a defined threshold, and an observable signal. Most components on a generator fail at least one of those tests, so the system does not attempt an estimate for them. Of the failure modes in the register, 44 of 65 look tractable for machine learning at all — and considerably fewer than that qualify for a remaining-life number.

**Normalisation has to be fitted on clean data only.** The expected-value models are fitted on a verified-healthy baseline window. Fit them across the whole dataset and the faults get absorbed into the definition of "expected", the residuals go flat, and the pipeline detects nothing while appearing to work perfectly. That failure mode is silent, which is what makes it dangerous.

## What I tested {#what-i-tested}

### Synthetic prototype evaluation — not field validation

Everything below was measured on a synthetic dataset: 30 days at one-minute resolution, 43,200 rows, with eight fault scenarios injected at known times. **These results demonstrate that the software pipeline and its logic work. They do not demonstrate performance on a real generator.**

There is a circularity that has to be stated plainly: the synthetic signals were generated from the same documented physics the feature engineering assumes. A model that scores well here has learned what I encoded into the data. It has not discovered anything about a real machine.

With that said, the comparisons are still informative — and several of them contradicted what I expected going in.

**A control chart beat both machine-learning detectors.** I compared an isolation forest and an autoencoder against plain four-sigma control limits on the same normalised residuals.

<div class="table-scroll">
<table class="table">
  <caption class="text-muted" style="text-align:left;font-size:12px;padding-bottom:8px">Detection on synthetic data. Lower detection lag is better.</caption>
  <thead>
    <tr><th scope="col">Detector</th><th scope="col" class="num">Episodes found</th><th scope="col" class="num">Mean detection lag</th><th scope="col" class="num">False-alarm runs</th></tr>
  </thead>
  <tbody>
    <tr><td>Isolation forest</td><td class="num">2 / 3</td><td class="num">2.84 d</td><td class="num">0</td></tr>
    <tr><td>Autoencoder</td><td class="num">3 / 3</td><td class="num">0.67 d</td><td class="num">6</td></tr>
    <tr><td><strong>Four-sigma control limits</strong></td><td class="num"><strong>3 / 3</strong></td><td class="num"><strong>0.50 d</strong></td><td class="num">2</td></tr>
  </tbody>
</table>
</div>

The reason is instructive rather than disappointing. Once the residuals are properly conditioned on load, ambient and viscosity, a single-signal limit works very well — because the normalisation has already removed the confounding that a multivariate model exists to handle. The feature engineering was doing the work and the model layer was collecting the credit. I kept the isolation forest in the pipeline, but for attribution and its lower false-alarm rate, not for lead time.

**A three-coefficient physics model beat the statistical estimators on remaining life.** For the one consumable that genuinely qualifies for a remaining-life estimate, I compared linear extrapolation, an ARIMA model, and a parametric fit that encodes a known physical shape — filter restriction accelerates as the clean area shrinks, so the loading curve is convex.

<div class="table-scroll">
<table class="table">
  <caption class="text-muted" style="text-align:left;font-size:12px;padding-bottom:8px">Remaining-life error against a known synthetic ground truth.</caption>
  <thead>
    <tr><th scope="col">Estimator</th><th scope="col" class="num">Mean absolute error</th><th scope="col" class="num">No estimate produced</th></tr>
  </thead>
  <tbody>
    <tr><td>Linear extrapolation</td><td class="num">95.1 h</td><td class="num">23 / 72</td></tr>
    <tr><td>ARIMA</td><td class="num">89.5 h</td><td class="num">16 / 72</td></tr>
    <tr><td><strong>Physics-informed fit</strong></td><td class="num"><strong>13.7 h</strong></td><td class="num"><strong>0 / 72</strong></td></tr>
  </tbody>
</table>
</div>

All three estimators are biased in the same direction — they predict more remaining life than there is, which schedules maintenance late. On a curve that accelerates, any locally-linear extrapolation has to do that.

**The confidence interval failed.** The main argument for using ARIMA was that it produces a confidence range, which is what turns a trend into a schedulable date. Measured against the known answer, its nominal ninety-percent interval covered none of the true values. The cause is structural rather than a tuning problem: the band represents random variance, while the error here is a systematic bias in one direction. So the prototype returns the estimate and withholds the upper bound rather than publishing a range it cannot support.

**The classifier failed a test I built specifically to catch it.** The synthetic data contains a deliberate trap: two faults that both pull oil pressure down. One is genuine bearing wear. The other is the pressure sensor drifting, with nothing physically wrong. The only discriminator is temperature dependence — real wear worsens as oil thins; a sensor offset does not care.

The classifier assigned 83.2% of the pure sensor fault to genuine bearing wear. Acted on, that means opening an engine because a transducer drifted. The asymmetry is telling: real wear was never mistaken for sensor drift, so the model had learned a one-way rule rather than the physics.

I did not treat that as a number to improve later. The prediction endpoint for that target now refuses to answer, returning the measurement as its reason and "obtain oil-analysis labels" as the remedy. It seemed better for the system to decline than to be confidently wrong once and distrusted afterwards.

**Some smaller findings.** A random forest outperformed gradient boosting on fault typing (0.843 against 0.764 macro-F1). Two features that only ever increase — accumulated running hours and thermal life — turned out to be a clock in disguise: they consumed 44.7% of the model's decision capacity and made it measurably worse, so they were removed. And a 23-test suite caught two bugs that would never have thrown an error, only produced quietly wrong numbers.

## Limitations I designed around {#limitations}

<div class="limits">
  <h3>What this prototype cannot claim</h3>
  <p>None of the following are hedges added after the fact. Each one shaped a design decision.</p>
  <dl>
    <dt>All telemetry is synthetic</dt>
    <dd>No data from a real generator has been used at any point. The dataset is a test harness that lets the pipeline be built and proven end to end. It is not evidence about a machine.</dd>

    <dt>No service manual was obtained</dt>
    <dd>Alarm setpoints, maintenance intervals and service limits are engineering estimates, tagged as such throughout. Every remaining-life figure scales directly with a threshold that has not been confirmed against manufacturer documentation.</dd>

    <dt>The controller register map is missing</dt>
    <dd>Which of the 164 catalogued signals can actually be read from the controller is unknown. Without the register map, the data-acquisition layer is a design rather than an implementation.</dd>

    <dt>Roughly a third of the required sensors are absent</dt>
    <dd>Against the target machine's standard build, about 40% of the catalogued signals appear to be available. 64 signals are both required by a documented failure mode and missing. Some would need retrofit; several existing points are switches rather than transducers, and a boolean has no slope to extrapolate.</dd>

    <dt>Physics constants and fault signatures are assumed</dt>
    <dd>The thermal and viscosity relationships in the normalisation layer are standard engineering forms with assumed coefficients. The injected fault signatures reflect my understanding of how these failures develop, not observed events.</dd>

    <dt>Nothing has been validated against a real failure</dt>
    <dd>There is no ground truth from an actual breakdown anywhere in this work. Detection lead times, error figures and health scores describe behaviour on generated data only.</dd>

    <dt>This is not production software</dt>
    <dd>The service layer has a bearer token and nothing else — no transport security, no per-machine authorisation, no rate limiting, no audit log. Those are listed rather than half-built, because a partial security layer looks finished and is worse than an obviously absent one.</dd>
  </dl>
</div>

## What I would do next {#what-next}

Ordered roughly by how much each one unblocks.

<ul class="checklist">
  <li>Obtain the Operation and Maintenance manual, and replace every estimated interval and setpoint with a cited value.</li>
  <li>Obtain the controller register map, and establish which of the 164 signals are actually readable.</li>
  <li>Connect one instrumented generator and begin collecting data, even before any modelling.</li>
  <li>Collect a verified-healthy baseline period, and refit the normalisation models on real behaviour rather than assumed physics.</li>
  <li>Review the fault signatures with a qualified service engineer — the assumptions most likely to be wrong are the ones I have least ability to check myself.</li>
  <li>Gather alarm and service history before attempting any supervised learning; without labelled real failures, supervised models have nothing to learn from.</li>
  <li>Add a calibrated confidence interval to the remaining-life estimator, since the current one demonstrably does not hold.</li>
</ul>

Only the last of these is a modelling task. That ordering is deliberate.

## Closing reflection {#closing}

The part of this project I found most valuable was not the machine learning. Measured honestly, the machine learning came third — behind the normalisation that made the residuals meaningful, and behind a physics model with three coefficients that beat every statistical estimator I put against it.

What I think was actually worth building was a system that can show its evidence. Every number traces to a source or is tagged as an assumption. Every model result carries the protocol that produced it. Three prediction targets refuse to answer at all, and each says why and what would fix it. When a measurement contradicted something I had written down earlier, the write-up says so rather than quietly dropping the earlier claim.

None of that makes the prototype useful on a real machine yet — it plainly is not, and the limitations section above is longer than the results section for good reason. But I would rather hand over a system that is clear about its own boundaries than one that produces confident numbers nobody can check. In this domain, the second kind gets switched off within a month, and the person who built it never finds out why.
