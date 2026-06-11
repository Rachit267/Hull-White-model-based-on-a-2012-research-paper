# Hull-White-model-based-on-a-2012-research-paper
Replication of the Two-Factor Hull-White (G2++) interest rate model — closed-form ZCB and caplet pricing, Bayesian calibration to GBP/SEK/EUR cap markets, and convergence analysis following Blanchard (KTH, 2012).
# Two-Factor Hull-White (G2++) Model
### Pricing & Calibration of Interest Rate Derivatives

Replication of the pricing framework and calibration methodology from:
> *"The Two-Factor Hull-White Model: Pricing and Calibration of Interest Rate Derivatives"*
> Arnaud Blanchard, KTH Royal Institute of Technology (2012)

---

## Model

The HW2F model drives the short rate via two correlated mean-reverting factors:

    dr = (θ(t) + u - a·r)dt + σ₁·dW₁
    du = -b·u·dt + σ₂·dW₂        corr(dW₁, dW₂) = ρ

This is equivalent to the **G2++ representation**:

    r(t) = x(t) + y(t) + φ(t)
    dx = -a·x·dt + σ·dZ₁
    dy = -b·y·dt + η·dZ₂

Where φ(t) is deterministic and chosen to fit the market yield curve exactly.
The G2++ form yields closed-form prices for bonds, caplets, and bond options.

**5 parameters:** `a`, `b`, `σ`, `η`, `ρ`

---

## What's Implemented

### Pricing (closed form)
- Zero-Coupon Bond — `P(t,T) = A(t,T) · exp(-Bₐx - B_by)`
- Zero-Coupon Bond Option — Jamshidian-style closed form
- Caplet — via bond put equivalence (caplet = put on ZCB with adjusted strike)
- Cap — sum of caplets
- Black implied volatility inversion via Brent's method

### Calibration
- Random grid search over `(a, b, σ, η, ρ)` minimising sum of squared price errors
- Multi-start with convergence logging at 100 / 1K / 5K / 10K / 1M iterations
- Applied to three markets: GBP, SEK, EUR

### Visualisations
- Implied vol curve: model vs market per tenor
- Price curve: model vs market in bps
- Three-market comparison chart
- Parameter convergence plots (log-scale iteration axis)

---

## Results

Calibrated parameters after 1,000,000 iterations (paper's values):

| Market | a | b | σ | η | ρ |
|--------|---|---|---|---|---|
| GBP | 96.9% | 24.0% | 0.47% | 1.18% | -78.8% |
| SEK | 90.5% | 11.6% | 0.91% | 0.90% | -97.2% |
| EUR | 11.5% | 10.3% | 0.47% | 0.07% | -85.7% |

Default `N_ITER = 10,000` runs in ~30 seconds and captures convergence behaviour.
Set `N_ITER = 1_000_000` (~5 minutes) to fully replicate the paper's parameters.

---

## Key Findings
- HW2F significantly outperforms the one-factor model on short-tenor vol fitting
- High mean reversion `a` in GBP/SEK reflects aggressive short-rate anchoring
- Strong negative correlation `ρ` across all markets — the two factors move inversely,
  producing the yield curve steepening that one-factor models cannot capture
- EUR's low `a` and `b` reflect a flatter, more persistent vol structure

---

## Data Note

Original paper uses Bloomberg cap market data as of November 15, 2011 (not publicly
available). Synthetic implied vol surfaces approximating the correct shape for each
market are used. All pricing formulas and calibration methodology are exact
replications of the paper.

---

## How to Run

```bash
git clone https://github.com/Rachit267/hw2f-model
cd hw2f-model
pip install numpy scipy matplotlib
python hw2f_notebook.py
