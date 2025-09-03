
# CCPI Framework Toolkit

This toolkit implements a Composite Coup Precursor Index (CCPI) that aligns your common pre-coup conditions with global indicators and integrates with SAT and ISO 31000 workflows. It also includes a watch-card template with CARVER fields that follow your recuperability rule.

## Files
- `ccpi_config.json` config with subindices, weights, indicator directions, optional fixed bounds, momentum settings, and bands.
- `ccpi_inputs_sample.csv` example CSV with three fictional countries for quick tests.
- `ccpi_scorer.py` script that reads inputs and config, computes subindices and CCPI, assigns bands, and exports watch cards.
- `watch_card_template.json` JSON structure for analyst-facing watch cards.

## Input schema
The CSV must include a `country` column and any indicator columns you want to use from the config. Missing columns are ignored. If you include year-over-year deltas, use the suffix `_delta` as defined in the config.

Examples of indicator columns:
- Governance Integrity: `wgi_control_of_corruption`, `wgi_rule_of_law`, `wgi_voice_accountability`, `fiw_aggregate`, `cpi_score`, `vdem_electoral_democracy_index`
- Political Contestation Stress: `fiw_military_in_politics_score`, `gpi_unrest_score`, `fsi_factionalized_elites`
- Security Pressure: `fsi_security_apparatus`, `fsi_refugees_idps`, `vdem_internal_conflict`
- Socioeconomic Distress: `vdem_inflation_annual_pct`, `gdp_per_capita_ppp`, `bti_status_index`, `wgi_government_effectiveness`
- External Intervention: `fsi_external_intervention`
- Optional event flags that directly boost CCPI: `violent_transfer_cue`, `military_ultimatum_cue`

## Normalization and direction
Each indicator is normalized to 0..1 where 1 means higher coup risk. The config sets `direction` to invert indicators where high values imply lower risk. You can also provide `fixed_min` and `fixed_max` to stabilize scoring across vintages and missing data. If you omit fixed bounds, the script uses dataset min and max.

## Momentum
If `_delta` columns exist, the script creates a momentum component within each subindex. Positive deltas increase risk where `direction=risk_increases_when_high`, while negative deltas increase risk where `direction=risk_increases_when_low`. The momentum weight within a subindex defaults to 0.25 and can be changed in the config.

## Bands
- Watch 0.30..0.49
- Elevated 0.50..0.69
- High 0.70..0.84
- Critical 0.85..1.00

## Outputs
- `ccpi_scores.csv` with `country`, `CCPI`, `band`, and per-subindex scores.
- `watch_cards.json` an array of country objects that include CCPI, band, ISO 31000 scenario with suggested likelihood, CARVER block, and a snapshot of raw signposts.

## Run
From a terminal with Python 3.9+ and pandas:
```bash
python /mnt/data/ccpi_scorer.py --config /mnt/data/ccpi_config.json --inputs_csv /mnt/data/ccpi_inputs_sample.csv --out_scores_csv /mnt/data/ccpi_scores.csv --out_watchcards_json /mnt/data/watch_cards.json --watch_template /mnt/data/watch_card_template.json
```

The command above produces two files:
- `/mnt/data/ccpi_scores.csv`
- `/mnt/data/watch_cards.json`

## Integrating with ISO 31000 and CARVER
- The watch card includes an ISO 31000 scenario named "Coup attempt within 6 months". The script sets a suggested likelihood 1..5 from CCPI bands. Analysts should fill `impact_1to5`, `mitigations`, `owner`, and `review_date`.
- The CARVER section uses your 1..5 scale and preserves your rule that high Recuperability means low resilience and harder recovery. Keep that invariant across all analyses.

## Customization
- Adjust subindex weights in `ccpi_config.json`.
- Add or remove indicators under each subindex. Use fixed bounds for stable comparisons across years.
- Tune event flag weights for your GSOC alerting thresholds.

## Notes
- The included sample CSV uses fictional data for demonstration only.
- For production, connect your annual indicator ETL and compute deltas between vintages.
