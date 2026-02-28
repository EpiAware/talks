# Building a composable Julia ecosystem for infectious disease modelling: a roadmap, challenges, and questions

## Abstract

Infectious disease models that integrate multiple data sources provide better evidence for outbreak response than chains of separate models, but building them is slow and requires expertise across domains.
Composable modelling, where validated components combine into joint models that properly propagate uncertainty, addresses this but requires an ecosystem of reusable infectious disease model components.
We believe Julia is the best language for this ecosystem due to: its type system, multiple dispatch, automatic differentiation support, and existing scientific computing infrastructure (SciML, Turing.jl, Distributions.jl) provide the foundations composable modelling needs.
In this talk we present our roadmap for creating and sustaining that ecosystem, our current progress, and our questions for the Julia community.

In R we have built the epinowcast ecosystem (packages, community forum, seminar series) and developed several other widely used packages including EpiNow2 and scoringutils.
We want to create something equivalent in Julia: a domain-focused ecosystem in the mould of SciML or Turing.jl, with the community infrastructure of rOpenSci and the domain specificity of SpeedyWeather.jl.

So far we have CensoredDistributions.jl, which handles common biases in epidemiological delay distributions, and an R interface prototype (EpiAwareR).
We initially plan to implement packages covering distribution extensions for epidemiological use, delay and generation time estimation, disease dynamics components, and forecast evaluation, alongside a centralised documentation site.

At the package level, we need to answer questions about what makes a good Julia package in our ecosystem: consistent documentation via DocStringExtensions and DocumenterCiterepress, robust testing with Aqua.jl and JET.jl, automatic differentiation backend testing via DifferentiationInterfaceTest, and where we need package extensions (e.g. for Turing.jl integration).

At the ecosystem level, we need to understand how to manage releases so that package versions work together, how to run reverse dependency checks before publishing, how to set up shared CI and centralised documentation across many packages, and how to help users understand which automatic differentiation backends are compatible when they combine multiple packages.

## Talk outline (12 min)

| Segment | Time | Content |
|---|---|---|
| Why composable modelling | 2 min | The problem in R/Stan, why reuse fails, why Julia |
| What we have built | 3 min | CensoredDistributions.jl, ecosystem map, what exists vs planned |
| Package and ecosystem standards | 3 min | Documentation, testing, AD compatibility, release management |
| Questions for the community | 2 min | AD backends, porting vs redesign, onboarding R users |
| Call for help | 2 min | What we need from the Julia community, how to get involved |

## Poster layout (if poster)

| Panel | Content |
|---|---|
| Left column | Why composable modelling, what we have in R (epinowcast ecosystem, packages, community) |
| Centre | What we want in Julia: ecosystem map, role models (SciML, Turing, rOpenSci, SpeedyWeather), what exists so far |
| Right column | Challenges and call for help: automatic differentiation, porting, onboarding, how to get involved; QR code to discussion thread |

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
