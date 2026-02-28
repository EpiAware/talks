# Estimating epidemiological delay distributions: from R/Stan to Julia

## Abstract

Delay distributions describe the time between epidemiological events, such as infection to symptom onset or symptom onset to hospitalisation.
Estimating these distributions from outbreak data is difficult because both the primary event (e.g. infection) and the secondary event (e.g. symptom onset) are usually only known to have occurred within a time window, such as a day.
Real-time outbreak data is also often right-truncated as longer delays have not yet been observed.
Ignoring double interval censoring and truncation biases parameter estimates which are then used for forecasting and transmission modelling.

Adjusting distributions for primary event censoring addresses this by integrating the delay CDF over the primary event window, weighted by the density of when, within the window, the event occurred.
This can then be combined with truncation and secondary interval-censoring adjustments to produce a double-interval-censored and right-truncation-adjusted distribution.

In this talk, we present [CensoredDistributions.jl](https://censoreddistributions.epiaware.org), which implements these adjustments as `primary_censored`, `interval_censored`, and `double_interval_censored`, composable [Distributions.jl](https://github.com/JuliaStats/Distributions.jl) wrappers.
Multiple dispatch selects closed-form CDFs for delay and primary event distribution pairs where these are available, and falls back to numerical integration otherwise.
We demo the package standalone and with [Turing.jl](https://turinglang.org/) for parameter estimation.

We then compare to [primarycensored](https://primarycensored.epinowcast.org), our equivalent R package, which also ships a duplicate set of [Stan](https://mc-stan.org/) functions so users can fit models in either language.
Maintaining two parallel implementations required reimplementing distribution functions in Stan, building tooling to vendor Stan code into downstream projects, and replacing types with integer distribution identifiers.
Stan's integral solver was also unstable for this problem, so we had to recast it as an ODE.
Julia's multiple dispatch and ecosystem composability eliminates all of this.

We then summarise our plans to build a composed Julia version of our [epidist](https://epidist.epinowcast.org) R package, using CensoredDistributions.jl as a foundation with Turing.jl submodels for partially pooled and flexible delay estimation.

## Talk outline (12 min)

| Segment | Time | Content |
|---|---|---|
| The problem | 2 min | What double censoring looks like in outbreak data |
| The maths | 2 min | Primary censored CDF, composition with truncation and intervals |
| Package demo | 3 min | CensoredDistributions.jl standalone and with Turing.jl |
| R/Stan comparison | 3 min | Side-by-side with primarycensored, why dispatch wins |
| Future plans | 2 min | Partially pooled fitting, composable submodels |

## Metadata

- **Conference**: JuliaCon 2026
- **Format**: Short talk (12 + 3 min Q&A)
- **Track**: General
- **Package**: [CensoredDistributions.jl](https://censoreddistributions.epiaware.org)
- **Comparison**: [primarycensored (R/Stan)](https://primarycensored.epinowcast.org)
