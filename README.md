# ccpi-subindex-calculator
The QA project's CCPI Global subindex calculator. CCPI, QA's in-house Geopolitical risk calculator based on an aggregation of several global indices 

# Quanta Analytica MNS CCPI Subindex Web App

A single file, front end analyst tool that calculates the Composite Coup Precursor Index (CCPI) using global indicators from reputable sources. Built to support independent conflict systems research and horizon scanning.

**CCPI analytic concept**: created by **M. Nuri Shakoor** as a passion project to advance his independent conflict systems research for **ARAC International Inc.** and partners. This repository operationalizes that concept so researchers can score countries, track momentum, and export watch cards for analytic workflows.

---

## Repository contents

* `index.html` - Main multi-tab application with import and export, schema templates, and a live bar chart.
* `Quanta_CCPI_Calculator.html` - Quick calculator lite version for rapid scoring.
* `CCPI_Subindex_Workbook_Template_v2.xlsx` - Spreadsheet template mirroring the app. Best opened in Google Sheets for full chart interactivity.

---

## Quick start (GitHub Pages)

1. Create or open a GitHub repository and add the three files above.
2. In the repository, go to **Settings → Pages**.
3. Under **Build and deployment**, set **Source** to **Deploy from a branch**, then select the branch and root folder.
4. Save. After the page publishes, open the site URL. The app loads from `index.html`.

No server code is required. Everything runs locally in the browser.

---

## What CCPI measures

CCPI is a composite risk score in the range 0 to 1 where higher values indicate greater near-term coup risk. It blends five subindices that map to widely observed conditions preceding coup attempts:

1. **Governance Integrity (GI)** - quality of institutions, corruption control, rule of law, democratic voice.
2. **Political Contestation Stress (PCS)** - factionalization, protest violence, political instability.
3. **Security Pressure (SP)** - repressive security apparatus, forced displacement, internal conflict.
4. **Socioeconomic Distress (SED)** - macro stress such as inflation, weak state capacity, stalled development.
5. **External Intervention (EXT)** - exposure to foreign military or political intervention.

Each subindex aggregates several indicators that are normalized to a common 0 to 1 risk scale and can include momentum.

---

## Global indicators and why they matter

The app ships with default rows you can customize. Indicators are grouped by subindex with suggested bounds and a direction to risk. You can add or remove rows as needed.

### GI - Governance Integrity

* **World Governance Indicators (WGI): Control of Corruption, Rule of Law, Voice and Accountability**
  *Why*: Coups are more common where corruption is entrenched, rule of law is weak, and public participation is constrained.
  *Scale*: −2.5 to +2.5. **Direction**: Lower is riskier (inverted to risk).

* **Freedom House aggregate score (0 to 100)**
  *Why*: Democratic backsliding often precedes unconstitutional transfers of power.
  **Direction**: Lower is riskier (inverted).

* **Transparency International Corruption Perceptions Index (0 to 100)**
  *Why*: Perceived corruption undermines regime legitimacy and elite cohesion.
  **Direction**: Lower is riskier (inverted).

* **V-Dem Electoral Democracy Index (0 to 1)**
  *Why*: Reduced electoral competitiveness and constraints on civil liberties often correlate with coup-prone environments.
  **Direction**: Lower is riskier (inverted).

### PCS - Political Contestation Stress

* **Fragile States Index (FSI) Factionalized Elites (0 to 10)**
  *Why*: Elite splits and factional competition create coup incentives.
  **Direction**: Higher is riskier.

* **Global Peace Index (GPI) Violent demonstrations and Political instability (1 to 5)**
  *Why*: Escalating protest violence and instability raise the likelihood of military intervention.
  **Direction**: Higher is riskier.

### SP - Security Pressure

* **FSI Security Apparatus (0 to 10)**
  *Why*: Politicized or repressive security institutions are both a driver and a channel for coups.
  **Direction**: Higher is riskier.

* **FSI Refugees and IDPs (0 to 10)**
  *Why*: Large displacement signals severe insecurity and state weakness.
  **Direction**: Higher is riskier.

* **V-Dem Internal conflict index (0 to 1)**
  *Why*: Internal armed conflict strains cohesion and increases coup volatility.
  **Direction**: Higher is riskier.

### SED - Socioeconomic Distress

* **Annual inflation, percent (0 to 50 default bounds)**
  *Why*: Inflation shocks erode purchasing power and regime legitimacy.
  **Direction**: Higher is riskier.

* **GDP per capita trend, percent change (−10 to +10 default)**
  *Why*: Income contraction increases grievance and elite predation.
  **Direction**: Lower is riskier.

* **BTI Status Index (1 to 10)**
  *Why*: Developmental stagnation and weak transformation capacity raise systemic stress.
  **Direction**: Lower is riskier.

* **WGI Government Effectiveness (−2.5 to +2.5)**
  *Why*: Low effectiveness limits service delivery and crisis response.
  **Direction**: Lower is riskier.

### EXT - External Intervention

* **FSI External Intervention (0 to 10)**
  *Why*: Overt foreign backing or interference can destabilize civil-military relations and trigger coups.
  **Direction**: Higher is riskier.

> Sources named above are globally recognized public data providers. This app does not fetch data automatically. Users import current values from official releases or trusted compendia and keep citations in the table’s `source` and `explanation` fields.

---

## Indicator schema

Each row in the Indicators table follows this schema.

| Field               | Type            | Description                                                         |
| ------------------- | --------------- | ------------------------------------------------------------------- |
| `subindex`          | enum            | GI, PCS, SP, SED, EXT                                               |
| `indicator_name`    | string          | Human-readable label                                                |
| `source`            | string          | Data source and vintage hint                                        |
| `direction_to_risk` | enum            | “Higher” or “Lower” meaning higher values increase or decrease risk |
| `min_bound`         | number          | Lower bound for normalization                                       |
| `max_bound`         | number          | Upper bound for normalization                                       |
| `current_value`     | number or blank | Year t                                                              |
| `prior_value`       | number or blank | Year t−1 for momentum                                               |
| `level_normalized`  | computed        | 0 to 1 risk level                                                   |
| `momentum_mstar`    | computed        | 0 to 1 momentum measure                                             |
| `explanation`       | string          | One-line rationale or footnote                                      |

**CSV header**:

```
subindex,indicator_name,source,direction_to_risk,min_bound,max_bound,current_value,prior_value,explanation
```

A JSON template and Excel template are generated by the app in the Schema Template tab.

---

## Formulas used by the app

Let `x` be `current_value`, `xmin` the lower bound, and `xmax` the upper bound. All results are clipped to the interval \[0, 1].

### Normalization to risk scale

* If `direction_to_risk` is **Higher**
  `level_normalized = clip((x − xmin) / (xmax − xmin), 0, 1)`

* If `direction_to_risk` is **Lower**
  `level_normalized = clip((xmax − x) / (xmax − xmin), 0, 1)`

### Momentum

Let `delta = x_t − x_(t−1)` and `share = delta / (xmax − xmin)`.

* If **Higher** increases risk: `m = share`
* If **Lower** increases risk: `m = −share`

Then `momentum_mstar = clip(0.5 + 0.5 * m, 0, 1)`.

### Subindex aggregation

For each subindex S:

* `avg_level_S` is the mean of `level_normalized` over valid rows for S.
* `avg_momentum_S` is the mean of `momentum_mstar` over valid rows for S.
* Momentum weight `w_S` is user adjustable in \[0, 0.25]. Defaults: GI 0.25, PCS 0.20, SP 0.20, SED 0.15, EXT 0.10.
* `subindex_final_S = (1 − w_S) * avg_level_S + w_S * avg_momentum_S`.

### Composite CCPI and flags

`CCPI_base = 0.30*GI + 0.25*PCS + 0.20*SP + 0.15*SED + 0.10*EXT`

Event flags, optional and independent:

* `violent_transfer_cue` adds `+0.05`
* `military_ultimatum_cue` adds `+0.05`

`CCPI = min(CCPI_base + flags, 1.00)`

### Band thresholds

* Watch: 0.30 to 0.49
* Elevated: 0.50 to 0.69
* High: 0.70 to 0.84
* Critical: 0.85 to 1.00

---

## Practical use cases

* **Country risk briefs**: Import the latest indicator values, record data vintages, and export the Watch-card JSON for dashboards.
* **Event monitoring**: Keep the app open during crisis periods, update the two event flags as cues appear, and track band changes.
* **Horizon scanning**: Create sessions for multiple countries and save each as a JSON file, then re-load to compare trajectories.
* **Teaching and training**: Use the Momentum sliders and the chart to demonstrate how level and change interact to affect risk.

---

## How to use the app

1. **Open `index.html`** in a modern browser.
2. Go to **Indicators** and edit or import your rows. The app validates fields and reports any errors.
3. Review **Calculator** to see subindex averages, momentum weights, CCPI base, flags, and CCPI.
4. Export data from **Data I/O** as JSON, CSV, or Excel. Export a **Watch-card JSON** with country, CCPI, band, time stamp, owner, and review date.
5. Use **Schema Template** to download clean templates for batch updates.
6. Save or load a **Session** to persist work between runs.

---

## About the quick calculator

`Quanta_CCPI_Calculator.html` is a lighter single page with the same math but fewer controls. Use it for quick demos or when working on low power devices.

---

## Spreadsheet template

`CCPI_Subindex_Workbook_Template_v2.xlsx` mirrors the app’s structure and formulas. It is designed for Google Sheets to take advantage of built-in charts and easy sharing. Use the workbook for batch data entry, then export to CSV or Excel and import into the app.

---

## Watch-card JSON structure

```json
{
  "country": "Example",
  "ccpi": 0.62,
  "band": "Elevated",
  "last_updated_utc": "2025-01-15T12:34:56Z",
  "owner": "Analyst Name",
  "review_date": "2025-06-30",
  "signposts_snapshot": []
}
```

---

## Reproducibility and quality controls

* All normalization uses explicit bounds and clipping to avoid NaN propagation.
* A status bar shows the last calculation time and count of valid indicators included in averages.
* The Indicators table has an `explanation` field for short rationales and footnotes so users can track data vintage and any imputation.
* Sessions are saved as JSON so analyses are portable and auditable.

---

## Limitations and analyst guidance

* CCPI is a structured heuristic, not a prediction market. It supports analyst judgment.
* Data series differ in cadence and methodology. Always record the year or month for each value.
* Default bounds are sensible ranges for cross-country comparisons. Adjust bounds when an indicator’s empirical range is narrower or wider for your sample.
* Momentum uses simple year over year change scaled by the same bounds used for levels. If sub-annual updates are available, you may adapt the prior value to the most recent period.

---

## Credits and attribution

* **Concept and methodology**: M. Nuri Shakoor for ARAC International Inc and partners.
* **Implementation**: Quanta Analytica MNS Consulting.
* **Data providers**: World Bank WGI, Freedom House, Transparency International CPI, V-Dem, Bertelsmann BTI, Fund for Peace FSI, Institute for Economics and Peace GPI, and other reputable public sources. All trademarks belong to their respective owners.

This project is provided for research and educational use. Cite your sources in your products.

---

## License

Unless otherwise noted, source code in this repository is released under the MIT License. Data you import remains subject to the licenses of the original providers.

---

## Contributing

Issues and pull requests that improve accessibility, documentation, or add optional import adapters are welcome. Please avoid adding server dependencies to keep the app lightweight and easy to host on GitHub Pages.
