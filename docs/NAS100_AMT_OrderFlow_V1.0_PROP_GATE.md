# NAS100 AMT + Order Flow V1.0 — Prop Deployment Gate

**Audit date:** 2026-11-20  
**Repository branch:** `nas100-amt-orderflow-research-from-main`  
**Status:** RESEARCH CANDIDATE / SHADOW-TEST ONLY / LIVE EXECUTION DISABLED

## 1. Why this document exists

This is the candidate selected from the NQ futures historical research. It is not yet a verified The5ers NAS100/US100 system. Keep this specification separate from the older generic OrderFlow-Scalper implementation until the live-data adapter and prop compliance gates are completed.

No prop orders may be submitted by this project at this stage.

## 2. Historical result (research evidence, not a forecast)

The candidate was tested on NQ futures tick and one-minute data covering 2026-05-20 to 2026-10-16. The strict quality-screened sample contained 74 RTH sessions and 54 trades.

- Net result: **+41.87R**
- Profit factor: **2.45**
- Positive trades: **26/54 (48.1%)**
- Average: **+0.775R per trade**
- Maximum drawdown: **4.11R**
- Longs: 28 trades, +23.22R
- Shorts: 26 trades, +18.65R

The model was selected using this same history. The chronological split is not an untouched out-of-sample test. The 54-trade sample is small and the win-rate 95% exact interval is approximately 34.3%–62.2%. These statistics do not establish a future edge or justify live risk.

## 3. Session and reference levels

- All strategy times use the IANA timezone `America/New_York`; do not hard-code EST because daylight-saving changes apply.
- Regular session: 09:30–16:00 New York time.
- Opening range: first 30 minutes, 09:30–10:00.
- No candidate entry before 10:00 or after 15:30.
- Long references: ONL, ORL, PDL.
- Short references: ONH, ORH, PDH, and prior-session VAH.
- Prior-session VAH is eligible only when the prior session passed the same data-quality gate.
- The approximate 70% value area was computed from tick-volume-at-price, not bid/ask classified volume. VAL and POC were not retained as entry references.

## 4. Candidate signal rules

### Long
1. Sweep ONL, ORL, or PDL by at least 2.0 NQ points.
2. Completed M1 candle closes at least 0.25 points above the reference.
3. Close is in the upper half of the candle (`close_pos >= 0.50`).
4. Lower wick is at least 25% of candle range.
5. Large sell-side tick-rule proxy volume exceeds large buy-side proxy volume and is positive.
6. Entry is the next M1 open.
7. Stop: signal low minus 1.5 NQ points.
8. Target: 3R.

### Short
1. Sweep ONH, ORH, PDH, or eligible prior-session VAH by at least 2.0 NQ points.
2. Completed M1 candle closes at least 0.25 points below the reference.
3. Close is in the lower 35% of the candle (`close_pos <= 0.35`).
4. Upper wick is at least 25% of candle range.
5. Large buy-side tick-rule proxy volume exceeds large sell-side proxy volume and is positive.
6. Entry is the next M1 open.
7. Stop: signal high plus 1.0 NQ point.
8. Target: 2.5R.

### Portfolio controls used in the historical test
- Maximum 2 trades per session.
- 30-minute cooldown after each completed trade.
- One open trade at a time.
- One trade per swept reference per session.
- Risk distance 4–30 NQ points.
- If stop and target are both touched in the same minute, assume stop first.
- Incomplete outcomes crossing missing-minute gaps are discarded.
- Base cost deduction: 0.5 NQ points round-trip.

All point distances above are NQ futures points. They are **not** The5ers CFD points and must not be copied without a symbol-specific conversion and fill-cost audit.

## 5. Important data limitations

The source tick files contain timestamp, price, volume, and an ask field, but no reliable aggressor-side flag, bid, trade ID, or historical order-book depth. Direction was inferred using a tick-rule proxy. Identical timestamp/price/volume records were deduplicated despite the possibility that some were distinct prints.

The tested large-print threshold is **8 source-volume units**, not independently verified as eight NQ contracts. The signal is therefore not proven true buyer/seller aggression, CVD, absorption, or DOM imbalance.

Do not transfer the 8-unit threshold to a The5ers CFD feed. Do not label a CFD tick-volume proxy as genuine exchange order flow. Any new proxy threshold requires separate calibration and prospective testing.

NQ futures, The5ers NAS100/US100 CFD pricing, contract size, spread, commission, financing, and execution are not interchangeable. No The5ers quote/specification or FBS data is used in this result.

## 6. Prop-firm gate — The5ers Bootcamp

Official The5ers information checked 2026-11-20:

- Bootcamp has three evaluation stages followed by a $20K Pro Trader stage. Published targets/max-loss figures differ by stage; the $20K Pro stage shows a 5% target, 4% maximum loss, and a 3% daily pause. The 3% daily pause applies only to Pro accounts.
- Bootcamp leverage is published as 1:30.
- News trading is allowed except for bracketing strategies.
- The5ers says custom automated trading software requires prior written approval. Its EA FAQ says the trader must own the source code and prohibits tick scalping and high-frequency trading, among other practices.
- The5ers terms also prohibit third-party EAs and state that approval is required before use of automated trading software.

Official references:
- https://the5ers.com/bootcamp/
- https://the5ers.com/faqs/how-does-the-bootcamp-program-work/
- https://the5ers.com/faqs/can-i-use-an-ea-expert-advisor-can-i-set-a-stealth-mode-stop-loss/
- https://the5ers.com/en/terms-and-conditions/
- https://the5ers.com/faqs/prohibited-trading-practices/

**Deployment rule:** Do not attach or run this project's EA, auto-signal software, or order automation on a The5ers account until the current account stage and exact program terms are checked and The5ers has provided written approval for the specific automated software. A 100 ms polling / 500 ms analysis loop in the existing bot is not suitable for use as a prop deployment shortcut.

Manual chart review and discretionary trade decisions are a separate workflow; this document does not authorize an automated prop-account connection.

## 7. Existing repository is not V1.0

The existing `config.py` currently specifies:
- `SYMBOL = "BTCUSDm"`
- `IS_CRYPTO = True`
- `RISK_PER_TRADE = 1%`
- `MAX_DAILY_LOSS = 3%`
- `MIN_RR_RATIO = 1.5`
- fixed 4-tick stop and 6/10/16-tick partial targets
- `DRY_RUN = True`

The current scanner uses a generic five-confirmation confluence model and passes signals to the existing 50/30/20 scale-out executor. That is not the candidate rule set above. Do not simply change the symbol to NAS100 and run it.

## 8. Required gates before any prop deployment

1. Confirm whether the account is Bootcamp Step 1, Step 2, Step 3, or $20K Pro Trader. Record the exact account stage and current rules.
2. Obtain written The5ers approval for this specific custom automated software before any automated use on the account.
3. Verify the exact MT5 symbol name from Market Watch; do not assume it is `NAS100` or `US100`.
4. Read the symbol's digits, point, trade tick size/value, contract size, volume minimum/step/maximum, current spread, trading sessions, commissions, and swaps from the actual account terminal.
5. Compare synchronized NQ and prop-symbol prices and estimate basis drift and execution-cost distribution. If no suitable NQ/CFD overlap data exists, record the conversion as unavailable.
6. Replace the non-portable 8-source-volume-unit rule with a separately calibrated, documented feed-specific rule. Do not claim true aggressor flow unless the feed supplies reliable aggressor-side data.
7. Implement the V1.0 M1 rules independently of the old generic signal detector. Add a hard live-order lock that cannot be disabled by a single config toggle.
8. Shadow-test at least 20–30 sessions and 50–100 valid candidate signals, recording missed signals, spread, entry quote, stop/target, MAE/MFE, outcome, and exact data source.
9. Reconcile every trade against the prop terminal and journal before considering any further stage.

## 9. Current operating mode

- **Live orders:** DISABLED.
- **Prop EA attachment:** NOT APPROVED / NOT DEPLOYED.
- **Current scanner:** NOT RUNNING as a verified live market scanner.
- **Alerts:** Discord channel exists, but no automated strategy feed is active.
- **Next work:** obtain terminal symbol/specification evidence and The5ers written automation approval; then implement and test a separate V1.0 shadow scanner.

This status must be changed only after the corresponding gate is evidenced in the project journal.
