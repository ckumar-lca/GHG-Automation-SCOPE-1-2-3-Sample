# Rheinwerk Komponenten GmbH: ESG / GHG case dataset

A fictional automotive supplier with 3 plants (Duisburg DE01, Plzen CZ01, Pune IN01), covering 2023–2025 (CBAM: 2026).
All company data is **synthetic** and was made messy on purpose, the way real exports from SAP, utilities and fuel-card providers are.
Do not open `answer_key/` until you've profiled the data yourself.

## raw/ (company data, what the sustainability team receives)
| File | Content | Use for |
|---|---|---|
| sites.csv | 3 plants, floor area, processes | dim_site |
| production_and_revenue.csv | monthly output (t) and revenue (EUR) | intensity KPIs (ESRS E1-6) |
| electricity_invoices.csv | monthly grid + onsite PV, mixed units | Scope 2 LB/MB |
| guarantees_of_origin.csv | HKN/GO certificates | Scope 2 market-based |
| gas_and_lpg_invoices.csv | natural gas (kWh Hs, m³), LPG (kg) | Scope 1 stationary + degree-day normalisation |
| fuel_card_export_DE.csv | daily fleet transactions, German format (`;`, decimal comma) | Scope 1 mobile |
| refrigerant_log.csv | F-gas top-ups | Scope 1 fugitive |
| sap_purchase_orders_EKPO.csv | purchase orders with SAP field names, 4 currencies | Scope 3 Cat 1, 2, 4, 6 |
| supplier_pcf_responses.csv | supplier PCFs, mixed quality | Scope 3 hybrid method, primary data share |
| sap_bom_export_CS12.csv | multi-level BOM for 2 parts | Project 2: PCF engine |
| cbam_imports_2026.csv | CBAM goods imports, partial supplier data | Project 2: CBAM report |

SAP field names: EBELN = PO number, BEDAT = PO date, WERKS = plant, LIFNR = vendor, NAME1 = vendor name, TXZ01 = item text, MENGE = quantity, MEINS = unit, NETWR = net value, WAERS = currency, MATKL = material group (empty, you classify it).

## reference/
- `heating_degree_days_SYNTHETIC.csv`: replace with real DWD Gradtage for Duisburg (and a CZ station).
- `emission_factors_ILLUSTRATIVE.csv`, `refrigerant_gwp_ILLUSTRATIVE.csv`: placeholder values to get the pipeline running. **Replace them with official sources** and version them (valid_from / source / version columns).

## Real open data to plug in
- **DESNZ/DEFRA GHG Conversion Factors** (gov.uk, yearly): fuels, refrigerants, freight, flights
- **UBA (Umweltbundesamt)**: German electricity mix CO2 factor, fuel emission factors
- **AIB European Residual Mix**: market-based Scope 2
- **India CEA "CO2 Baseline Database"**: Indian grid factor
- **DWD Climate Data Center**: Gradtagzahlen / heating degree days
- **EXIOBASE 3** (Zenodo, free): spend-based Scope 3 factors per sector and country
- **IPCC AR5/AR6 GWP100**: refrigerants (ESRS expects current IPCC values)
- **EU CBAM default values** (European Commission, Taxation & Customs Union): Project 2
- **ecoinvent** (if you have a licence), otherwise the **EF database / ELCD** or **Agribalyse/USLCI** via Brightway for the BOM-to-PCF engine
- **PACT Methodology / Catena-X PCF Rulebook**: PCF data exchange format

## Suggested build order
1. Profile → clean → load into PostgreSQL/DuckDB (staging → core star schema)
2. Unit and currency harmonisation, data-quality flags table
3. Scope 1 & 2 (LB + MB) → Scope 3 Cat 1 (spend-based, then hybrid with PCFs)
4. Degree-day normalised gas KPI per plant
5. SQL views mapped to ESRS E1-6 datapoints → Power BI
6. Project 2: BOM explosion → material mapping → Brightway PCF → CBAM embedded emissions + cost
