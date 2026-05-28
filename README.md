# ETF-Signals

Daily reports from a thematic ETF multi-momentum screening system.

This repo hosts two PDFs, each updated by the daily pipeline:

- **[Defensive Strategy](SM-TETF-Report.pdf)** — dual-momentum on cluster PC1 portfolios. Lower-vol defensive sleeve.
- **[Aggressive Strategy](SM-TETF-Aggressive.pdf)** — relative-strength rotation: leading ETFs within leading sectors. Higher-octane sleeve.

## How the system is built

Both strategies operate on the same universe of ~700 thematic ETFs and share the upstream screen:

- **Full-sample PC1** — prcomp on standardised returns of the full universe; the PC1 portfolio is compounded into a price series for the market trend filter.
- **Cluster PC1** — ETFs hierarchically clustered on rolling-3-day cumulative log returns using `sqrt(2*(1 - rho))` distance with Ward.D2 linkage. Default cluster count is `max(2, round(sqrt(N)))` (~27 for a 730-ETF universe). Each cluster has its own PC1 portfolio price series.
- **Individual ETF** — MA signals computed directly on each ticker.

## The two strategies

### Defensive — Cluster Dual Momentum
Trades the cluster PC1 portfolios. 5/50 moving-average crossover with a vol-scaled cushion on the sell trigger, a 252-day absolute-momentum entry filter (Antonacci-style), and a 30-day compliance min-hold. Cluster weights via inverse-vol → vol-target (10%) → per-cluster cap (20%), rebalanced monthly. Locked Sharpe 0.63, CAGR 5.2%, Max DD −11.6%.

### Aggressive — Relative-Strength Rotation
Same signal machinery but applied to *relative-strength lines*: each cluster vs. the market, and each ETF vs. its own cluster. Each month picks the top 6 leading clusters by RS momentum, then the top 3 ETFs within each by ETF-RS momentum (~9–13 holdings), inverse-vol weighted across the selected set, with a full-market 200d-MA trend brake that pulls the book toward cash in risk-off regimes. Locked Sharpe 0.93, CAGR 12.1%, Max DD −13.5%, 2022 −2.9%.

The two are designed as complements — the Defensive sleeve emphasises low vol and shallow drawdowns; the Aggressive sleeve concentrates on sector leadership for higher return with the trend brake as the principal downside control.

## Signals

- **Convergence probability** — when a fast MA is approaching either threshold, `P(crossover within horizon days)` is estimated by modelling the daily change in the spread as `N(mu, sigma^2)` with `mu`, `sigma` from the last `vol_window` daily changes. Endpoint probability `P(spread_{t+h} on the wrong side of zero)`. Cheap and intuitive; understates true crossing probability when horizon is large (no first-passage-time correction).
