# ETF-Signals
Daily report from a thematic ETF multi-momentum screening system

# Thematic ETF Screener
Multi-momentum signal screener for a thematic ETF universe, run at three levels of aggregation.

## Strategy
- **Buy signal:** DMA crosses another DMA
- **Sell signal:** DMA crosses *below* another DMA.
- **Convergence probability:** When the dma is approaching either threshold, I estimate P(crossover within `horizon` trading days) by modelling the daily change in the spread (dma minus target MA) as `N(mu, sigma^2)`, where `mu` and `sigma` are the mean and stdev of the last `vol_window` daily changes. P is taken as the endpoint probability `P(spread_{t+h} on the wrong side of zero)` -- this is cheap and intuitive, but I know this understates true crossing probability when `horizon` is large.

## Three levels of analysis
1. **Full sample PC1** -- prcomp on standardized returns of the full universe. Loadings are sign-aligned to be long-biased and rescaled to unit gross exposure. The PC1 portfolio's daily return is compounded into a price series; MA signals run on that.
2. **Cluster PC1** -- ETFs are hierarchically clustered on **rolling 3-day cumulative log returns** (smoother than daily, set via `cluster_smoothing_window`) using `sqrt(2*(1 - rho))` distance with Ward.D2 linkage. Default cluster count is `max(2, round(sqrt(N)))` -- about 27 for a 730-ETF universe. **Cluster IDs are sorted by descending size** (cluster 1 is always the largest theme) so output ordering is stable across runs. Each cluster gets its own PC1 portfolio price series and its own set of MA signals.
3. **Individual ETF** -- MA signals computed directly on each ticker's adjusted-price series.
