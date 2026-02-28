# EpiAware: designing a composable Julia ecosystem for infectious disease modelling

## Abstract

Infectious disease models that integrate multiple data sources provide better evidence for outbreak response than chains of separate models, but building them is slow and requires expertise across domains.
In a companion presentation we argue that composable modelling, where validated components combine into joint models that properly propagate uncertainty, addresses this.
Here we focus on the practical question: how do you build and sustain an ecosystem of composable packages?

We maintain several R and Stan packages used in public health (EpiNow2, epinowcast, scoringutils).
In R, at least seven packages estimate reproduction numbers using renewal-based approaches in Stan, yet none share components despite overlapping features and, in some cases, authors.
This duplication motivated us to design a Julia ecosystem where the unit of reuse is the epidemiological concept, not the codebase.

We are decomposing a proof of concept (EpiAware.jl) into focused packages that compose through Distributions.jl and DynamicPPL interfaces, following the organisational model of SciML and Turing.jl.
CensoredDistributions.jl is the first production package (see companion talk).
Planned packages cover generation time estimation, ODE-based disease components, and distribution modifications, with a domain-specific language layered on top.
We want to replicate SciML's infrastructure for centralised documentation, shared CI, and reverse dependency testing.

We present our ecosystem plan and ask for community input on three problems.
First, autodiff backend compatibility: when composed packages each support different backends, the working intersection for a user's model is hard to predict and harder to communicate.
Second, porting R packages to Julia: when a direct translation makes sense versus a Julia-native redesign, how to coordinate with upstream maintainers, and whether LLM-assisted conversion is practical at scale.
Third, onboarding domain experts: most infectious disease modellers use R, and we need to decide between an R interface, R-user-focused tutorials, a domain-specific language, or focusing on Julia-native adoption.

## Talk outline (12 min)

| Segment | Time | Content |
|---|---|---|
| The problem | 2 min | Seven renewal packages, zero shared components; what the R/Stan ecosystem taught us about why reuse fails |
| The ecosystem plan | 3 min | Package map, composition via Distributions.jl and DynamicPPL, DSL layer, what exists vs what is planned |
| Borrowing from SciML/Turing | 2 min | Centralised docs, shared CI, reverse dependency testing; what we want to replicate and what we do not yet know how to |
| Where we need help | 5 min | AD backend compatibility across composed packages, porting vs redesign for R packages, onboarding domain experts from R |

## Poster layout (if poster)

| Panel | Content |
|---|---|
| Left column | The problem: duplication in R/Stan, pipeline vs joint modelling trade-off, why reuse fails |
| Centre | The ecosystem plan: package map with status (live / planned / empty), composition interfaces, SciML/Turing infrastructure to replicate |
| Right column | Where we need help: AD backends, porting strategy, onboarding domain experts; QR code to discussion thread |

## Notes

### Motivating examples (for speaker notes, not the abstract)

These are concrete examples that illustrate why composable modelling matters.
The composability argument itself is presented separately; these are here as brief touchpoints if the audience asks "why does this matter?"

- **COVID CIS pipeline**: ONS prevalence estimates fed into incidence estimates which fed into reproduction number estimates which informed variant severity analyses.
  At each stage, uncertainty was approximated and assumptions inherited without the ability to evaluate their impact.
- **Mpox inflexibility**: COVID models integrating case counts, prevalence, severity, and hospital data could not be adapted to mpox where contact structure needed explicit representation.
  New single-data-source models were built from scratch instead.
- **Seven renewal packages**: At least seven R/Stan packages estimate reproduction numbers using renewal approaches, none sharing components despite overlapping features and authors.
- **Wastewater estimation**: Existing tools were built by non-wastewater experts, share no components, and it is unclear which modelling choices matter.
- **Precedent in other Julia domains**: HydroModels.jl and SpeedyWeather.jl demonstrate that DSL-based composable modelling works in Julia for other scientific domains.

### Relationship to CensoredDistributions.jl talk

The companion talk covers CensoredDistributions.jl in depth: the censoring problem, the maths, a live demo, and a direct R/Stan comparison.
This talk/poster does not repeat that material.
It references CensoredDistributions.jl only as the first completed package in the ecosystem to illustrate the target quality and design pattern.

### Overlap boundaries

- **CensoredDistributions talk owns**: the censoring problem, dispatch vs Stan integers, package demo, Turing.jl integration.
- **This talk/poster owns**: the ecosystem plan, design considerations, R ecosystem lessons (organisational not technical), porting strategy, AD backend problem, community questions.

### R ecosystem lessons (organisational angle)

The CensoredDistributions talk covers the technical comparison (dispatch vs Stan vendoring).
This talk covers the organisational lessons instead.

Maintaining epinowcast in R taught us:
- Modular R packages work but CRAN's release cycle slows iteration.
- Stan's lack of a type system forced us to maintain integer-indexed distribution registries and vendor Stan code into downstream packages.
- Community adoption depends on documentation quality and entry-level tutorials more than package features.
- Reverse dependency testing is essential when packages compose, and R's tooling for this is limited compared to what SciML has built.

These lessons directly shaped the Julia ecosystem design.

### AD backend compatibility (concrete framing)

CensoredDistributions.jl works with ForwardDiff and ReverseDiff.
Mooncake fails on numerical integration code paths.
Enzyme has not been tested.

When a user composes CensoredDistributions.jl with another EpiAware package inside a Turing model, the composed model inherits the intersection of supported backends.
With many packages this intersection may be empty or hard to predict.

We test with DifferentiationInterface.jl but have no way to communicate results to users at composition time.
Options we want to discuss:
- Per-package compatibility badges in docs.
- A function that checks backend support for a composed model at runtime.
- Restricting the ecosystem to backends that work everywhere (limiting but simpler).

### Porting strategy (concrete framing)

We have three cases:

1. **Our own packages, redesign**: primarycensored (R) became CensoredDistributions.jl.
   The Julia version shares the mathematical approach but has a completely different API built on Distributions.jl.
   This worked well but required significant design effort.

2. **Our own packages, port**: scoringutils (R) could become a Julia package.
   The API is tabular (DataFrames-based) and may translate more directly.
   LLMs could accelerate the mechanical translation.
   Open question: does a direct port make sense or should we redesign around Julia's type dispatch?

3. **Others' packages, port**: scoringRules (R) implements proper scoring rules.
   We do not maintain it.
   Open question: fork and port, collaborate with maintainers, or write an independent Julia package?

### Barriers to entry (concrete framing)

Most infectious disease modellers use R.
They know ggplot2, tidyverse, and Stan.
They do not know multiple dispatch, abstract types, or DynamicPPL.

Concrete options we want to discuss:
- R interface via JuliaCall (EpiAwareR exists as a prototype).
- Julia tutorials written for R users, showing equivalent patterns.
- A domain-specific language that hides Julia's type system behind epidemiological concepts.
- Whether trying to attract R users is the right strategy or whether we should focus on Julia-native adoption.

## Ecosystem package map (text version, to become a diagram)

```
                    epiaware.github.io
                    (centralised docs)
                           |
        ┌──────────────────┼──────────────────┐
        |                  |                  |
  Distributions.jl    DynamicPPL        SciMLBase
   extensions          extensions        extensions
        |                  |                  |
  ┌─────┼─────┐           |            DEdisease-
  |     |     |           |            components.jl
  |     |  Modified-      |
  |     |  Distributions.jl
  |     |                 |
  |  Reparameterised-     |
  |  Distributions.jl     |
  |                       |
  Censored-          GenerationTime.jl
  Distributions.jl   (uses Censored-
  (v0.2.9, live)      Distributions.jl)
```

## Next steps

- [ ] Decide talk vs poster
- [ ] Check JuliaCon 2026 submission deadline and format requirements
- [ ] Refine ecosystem diagram into a proper figure
- [ ] Prepare one-slide examples for each open problem
- [ ] Coordinate submission with CensoredDistributions.jl talk

## Metadata

- **Submitted by**: [@seabbs](https://github.com/seabbs)
- **Conference**: JuliaCon 2026
- **Format**: Poster (preferred) or short talk (12 + 3 min Q&A)
- **Track**: General
- **Ecosystem**: [EpiAware](https://github.com/EpiAware)
- **Companion talk**: CensoredDistributions.jl (same authors)
- **Companion presentation**: Composable probabilistic infectious disease models (the "why"; this talk is the "how")
- **Design paper**: ComposableProbabilisticIDModels
