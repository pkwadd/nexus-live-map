# NEXUS LIVE SYSTEM MAP — FULL
# Source: Live BigQuery graph queries — nexus-compendium.nexus_compendium
# Generated: 2026-06-12 | Status: CURRENT — COMPLETE SYSTEM VIEW

---

## WHAT NEXUS IS

Nexus is a pharma BD intelligence platform operating as a positioning radar.
Contract: NEXUS_CONTRACT_UNIFIED_V2_2.md (pkwadd/nexus-ingest) — LOCKED v2.3

Core question: "What is becoming true in pharma, and what capability will it require?"
Customer: Vendors selling recurring systems and execution capability into pharma.

Nexus is NOT lead gen, alerting, monitoring or news aggregation.
Nexus IS a positioning intelligence platform built on continuous observation of reality formation.

Architecture: 6 layers — Ingest → Asset/Company mapping → Trajectory → Opportunity → Stage 6 → Narrative (AI only at narrative edge)

---

## PLATFORM IDENTITY

- GCP Project: nexus-compendium
- Primary dataset: nexus_compendium
- Region: europe-west2
- GitHub: pkwadd/nexus-ingest (PRIVATE)
- Live map repo: pkwadd/nexus-live-map (PUBLIC)

---

## CLOUD RUN SERVICES (europe-west2)

NOTE: All URLs changed 2026-06-11. Pattern: nexus-*-542997761585.europe-west2.run.app

nexus-ingest     https://nexus-ingest-542997761585.europe-west2.run.app     Orchestrator + API
nexus-bd         https://nexus-bd-542997761585.europe-west2.run.app         BD frontend (vendor-facing)
nexus-guardian   https://nexus-guardian-542997761585.europe-west2.run.app   Guardian health service
nexus-bq-mcp     https://nexus-bq-mcp-knza2hsdxa-nw.a.run.app              BigQuery MCP for ChatGPT
nexus-github-mcp https://nexus-github-mcp-knza2hsdxa-nw.a.run.app          GitHub MCP for ChatGPT

---

## SCHEDULER JOBS (europe-west2)

nexus-ingest-nightly    0 18 * * *      ENABLED   last: 2026-06-11T18:00   /run          <- SOLE ACTIVE TRIGGER
nexus-guardian-check    */15 * * * *    ENABLED   last: 2026-06-11T23:45   /check
nexus-guardian-snapshot 0 6 * * *      ENABLED   last: 2026-06-11T06:00   /snapshot
nexus-ingest-daily      0 2 * * *      PAUSED    NEVER RUN
nexus-runner-a-hourly   5 * * * *      PAUSED    NEVER RUN
nexus-commercial        2-59/5 * * * * PAUSED    NEVER RUN                /run_commercial
nexus-clinical          */5 * * * *    PAUSED    NEVER RUN                /run_clinical

BLACKOUT WINDOW: Do NOT deploy nexus-ingest between 18:00–22:00 UTC.
nexus-clinical would decouple clinical trials from nightly run — currently paused and unstarted.

---

## API ROUTES (nexus-ingest)

/                -> health check
/run             -> full orchestrator trigger (scheduler target)
/run_clinical    -> clinical trials only (scheduler target — NOT YET ACTIVE)
/run_commercial  -> commercial ingest only (scheduler target — NOT YET ACTIVE)
/feed            -> vw_nexus_opportunity_v2          PRIMARY INTELLIGENCE FEED
/account         -> nexus_operational_state          ACCOUNT PAGE (company state)
/assets          -> vw_nexus_opportunity_v2          ASSET DETAIL
/ep/profile      -> vw_ep_relationship_edges         EP PROFILE
/ep/league-table -> ep_observed_profile_v1           EP LEAGUE TABLE — 500 ERROR BROKEN
/ep/summary      -> ep_observed_profile_v1           EP SUMMARY
/ep/opportunity-map -> vw_vendor_opportunity_scoring_v6  EP OPPORTUNITY MAP
/intelligence    -> vw_bd_command_briefing (1,066 rows)   INTELLIGENCE TAB
/api/narrative   -> claude-sonnet-4-6 (Anthropic API)    AI NARRATIVE — only AI in system

---

## BIGQUERY DATASETS

nexus_compendium          PRIMARY — all live objects
nexus_ai_visibility       Diagnostic views (null rates, row counts, source inventory)
compendium                Legacy forward inbox signals
nexus_compendium_archive  Archived canonical backups
nexus_compendium_backup   Backup tables

### Object counts (nexus_compendium)
BASE TABLE:     75
VIEW:          141
SNAPSHOT:       31 (daily snapshots 2026-04-14 through 2026-06-11)
PROCEDURE:       3
PROPERTY GRAPH:  2

---

## LAYER 1 — INGEST PIPELINE

### Raw ingest tables
signals_raw_ingest_guarded_v1    437,273 rows   ACTIVE WRITE TARGET
signals_raw_ingest_v1            (legacy)
signals_stage                  11,906,417 rows   STAGING (includes all history)

### Eco pipeline (raw → staged → qualified)
eco_raw_clinicaltrials           0 rows     (raw cleared / superseded)
eco_staged_clinicaltrials      103,285 rows
eco_qualified_clinicaltrials   103,285 rows

eco_raw_edgar_ex10               2,364 rows
eco_staged_edgar_ex10              152 rows
eco_qualified_edgar_ex10           304 rows

eco_raw_edgar_8k                23,704 rows
eco_staged_edgar_8k                 46 rows
eco_qualified_edgar_8k           1,400 rows

eco_raw_fda_gmp                 (present)
eco_staged_fda_gmp              (present)
eco_qualified_fda_gmp           (present)
eco_raw_fda_citations           (present — 483 citations)
eco_raw_fda_inspections         (present — 483 inspections)

eco_raw_pressrelease            (present)

### Regulatory landing tables
regulatory_fda_landing_v1       (present)
regulatory_mhra_landing_v1      (present)
sam_gov_raw                     (present)

### Write procedure (LOCKED — never bypass)
NX_STAGE_INSERT_ONLY      signals_raw_ingest_guarded_v1 → signals_stage
NX_CANONICAL_INSERT_ONLY  signals_stage → canonical_signals_v3_lock
run_ingest_cycle_v2       full cycle wrapper

RULE: NEVER write directly to canonical_signals_v3_lock. Always via procedures in sequence.

---

## LAYER 2 — CANONICAL SIGNAL STORE

canonical_signals_v3_lock       467,197 rows   WRITE TARGET (append-only)
canonical_signals_v3            467,197 rows   READ ALIAS (= lock table)
canonical_signals_v3_read       (read-only view alias)
canonical_signals_v3_protected  (protected copy)

### Canonical schema (12 cols)
raw_id, observed_at, source_name, raw_payload, fetched_at,
title, notes, company_name, asset_name, asset_key, phase, signal_class

NOTE: nct_id, condition, intervention NOT in canonical — extract via JSON_VALUE(raw_payload).
NOTE: observed_at contains a data quality issue — some rows show 2028-10-31.

### Signal class breakdown (467,197 total)
existence:               190,457   40.8%
irreversibility:         103,838   22.2%
ecosystem:                62,596   13.4%
commercial_relationship:  44,397    9.5%
null (unclassified):      29,352    6.3%
publication:              27,953    6.0%
publication_activity:      3,604    0.8%
eu_marketing_authorisation: 2,313   0.5%
market_exit:               1,054    0.2%
ep_article_scan:             687
ep_trade_press:              383
terminated:                  372
vendor_partnership_announcement: 68
operational_hiring:           62

### Top signal sources (209 total)
clinicaltrials_update:         88,826   existence
patent_activity:               82,564   irreversibility
clinicaltrials_backfill:       46,481   existence
usaspending:                   23,723   commercial_relationship
eco_edgar_ex10_qualified:      22,210   commercial_relationship
openfda_drug_label:            14,322   ecosystem
regulatory_fda:                13,874   irreversibility
eco_drug_label_signal:         12,773   irreversibility
europepmc:                     10,225   existence
medaff_openalex:               10,183   publication
pubmed:                         8,487   publication
medaff_europepmc:               7,674   publication
openfda_drug_enforcement:       6,319   ecosystem
nih_reporter:                   6,195   existence
openfda_faers:                  5,100   ecosystem
fda_fei_registration:           5,008   existence
eco_fda_approval_signal:        3,624   irreversibility

### Snapshots (daily, 06:00 UTC)
31 snapshots: canonical_signals_snapshot_20260414_now through 20260611_0200
Guardian last snapshot: 2026-06-11

### Backups
canonical_signals_v3_backup
canonical_signals_v3_lock_backup_20260529_pre_btd_recovery
canonical_signals_v3_lock_old_20260508
canonical_backup_20260508

---

## LAYER 3 — SPINE & ENGINE

### Spine views
vw_signals_master_resolved_v3      master resolved (9 cols, no asset_name/asset_key)
vw_signals_engine_ready_v1         engine-ready filter (pharma, mapped, classified)
vw_signals_engine_clean_v1/v2/v3   cleaning variants
vw_signals_regulatory_all_spine_ready_v1

### Asset resolution
vw_asset_owner_authority_v1          Gate 1 (pharma entity gate — 1,110 sponsors passing)
vw_asset_owner_authority_v1_candidate
vw_asset_owner_map_v1 / v2
vw_asset_company_map_v1
vw_asset_company_lookup_v1
vw_nexus_asset_company_map_v1
vw_asset_resolution_feasibility
asset_originator_map_v1
canonical_entity_registry
canonical_entity_aliases
canonical_entity_override
owner_resolution_blocklist_v1
ct_unresolved_collaborators

### Company state engine
nexus_operational_state           65,684 rows total (14,288 distinct companies in latest partition)
  Last computed: 2026-06-05 — STALE BY 7 DAYS
  heat >= 70 (high):   1,380
  heat 40–70 (mid):      910
  heat < 40 (low):    11,998

vw_nexus_operational_state_current_v1  (current partition view)
vw_entity_operational_profile
vw_entity_progression_state
vw_entity_progression_timeline
vw_entity_intelligence_profile
vw_entity_resolution
vw_entity_transition_timeline
vw_entity_vendor_matrix

### Formation classifier (trigger layer)
vw_trigger_classifier_v1         63,416 companies classified
  Four-bucket deterministic: NEW / DISRUPTED / MAINTAINED / REMOVED
  Approved NEW sources: eco_fda_approval_signal, eco_ema_approval_signal,
                        eco_ema_expanded_signal, eco_who_drug_signal
  DISRUPTED: "Refusal"/"Withdrawal" strings in eco_ema_approval_signal.title
  STATUS: NOT WIRED TO INTELLIGENCE FEED

### Radar
vw_nexus_radar_v1                2,897 rows  (BUILT — wired status unknown)

### Action queue
action_queue_LOCKED              2,659,176 rows
vw_action_queue_v1               (view over action queue)

---

## LAYER 4 — TRAJECTORY & OPPORTUNITY

### Trajectory
vw_nexus_asset_trajectory_v2     78,789 assets tracked

### Law 23 / Formation path
vw_nexus_law23_v1                (law 23 base)
vw_nexus_law23_formation_v1      646 formations
vw_nexus_formation_opportunity_v1 647 formation opportunities
vw_nexus_formation_v1            (formation base view)
vw_nexus_opportunity_v2          PRIMARY FEED (vendor-facing)
vw_nexus_opportunity_surface     6 rows  (surface summary)
vw_nexus_law07_v1

### Opportunity caches
cache_opportunity_summary        (present)
cache_opportunity_capabilities   (present)
opportunity_feed_cache           (present)

### Execution pressure
execution_pressure_events        45,362 rows
vw_execution_pressure_inventory
vw_pressure_balance_score
vw_pressure_topology_activation

### Why now / inevitability
vw_asset_why_now_v2 / v3 / clean_v1
vw_operational_inevitability_surface
vw_irreversibility_accumulation
vw_operational_progression_path

---

## LAYER 5 — STAGE 6 (VENDOR MATCHING)

### Match / score / rank chain
vw_stage6_vendor_opportunity_match_v2    434,284 rows   MATCH LAYER
vw_vendor_opportunity_scoring_v6          81,618 rows   SCORE LAYER (v6 = current)
vw_vendor_opportunity_ranked_v6           81,618 rows   RANK LAYER
vw_vendor_opportunity_final_v1/v2/v4      (versioned finals)
vw_stage6_top_opportunities_v1             5,342 rows   TOP OPPS
vw_stage6_top_opportunities_top50_v1      (top 50)
vw_stage6_bd_output_v1 / final_v2         (BD output)
vw_stage6_reverse_need_v1

NOTE: vw_vendor_opportunity_scoring_v4 MISSING — breaks scoring chain dependency if referenced.
Scoring chain: match_v2 → scoring_v6 → ranked_v6 → final → BD output

### Vendor registry
forward_vendor_registry_v1          682 vendors   CURRENT
forward_vendor_registry_vs1_snapshot
forward_vendor_registry_vs2_snapshot
forward_vendor_registry_v1_backup_20260603
gov_vendor_registry_changelog

### Capabilities
lifecycle_capability_bundles         79 capabilities
capability_map_v1                     7 domains
ecosystem_capability_layers
eco_capability_signal_map

### Vendor intelligence
eco_vendor_stance_conditions       188,958 rows   BUILT BUT UNWIRED FROM API
eco_vendor_stance_signal_map
eco_stance_condition_vocabulary
nexus_vendor_pharma_relationships     199 relationships
vw_execution_partner_stance
vw_vendor_stance_explorer
vw_vendor_stance_surface
vw_vendor_timing_posture
vw_vendor_action_recommendation
vw_vendor_capability_exposure
vw_vendor_entity_alignment
vw_vendor_by_capability / v2
vw_vendor_opportunity_explain_v1
vw_vendor_opportunity_ranked_v3/v4/v5/v6
vw_vendor_opportunity_scoring_v5/v6
vw_entity_vendor_matrix

cache_vendor_matches               (present)

---

## LAYER 6 — EP INTELLIGENCE

execution_partner_profiles_v1      (profiles table)
ep_observed_profile_v1             339 EPs
execution_partner_attachments
vw_ep_relationship_edges           (relationship edges)
vw_ep_demand_surface_v1            339 rows
vw_ep_convergence_v1               339 rows
vw_ep_trajectory_v1
vw_execution_partner_emergence
vw_execution_partner_enrichment_candidates
vw_execution_partner_signal_triage
vw_unresolved_execution_partners

Key EPs: Syneos Health, Lonza, Charles River, Veeva, IQVIA, Medpace,
         Certara, Evotec, Icon, Fortrea, Almac, Labcorp, WuXi,
         Samsung Biologics, Parexel, AGC Biologics, Eversana

---

## LAYER 7 — BIGQUERY PROPERTY GRAPH (launched April 2026)

Two graphs in nexus_compendium dataset:
nexus_graph_v1   created: 2026-05-30   (previous version)
nexus_graph      created: 2026-06-06   (CURRENT)

Purpose: Enables cross-AI graph traversal so Claude, ChatGPT and Perplexity
can query the entire Nexus system as a connected graph rather than isolated tables.

### Node tables (backed by views/tables)
graph_node_signal      -> canonical_signals_v3_lock       467,197 rows
graph_node_asset       -> vw_nexus_asset_trajectory_v2     94,959 rows
graph_node_company     -> nexus_operational_state          65,684 rows
graph_node_trajectory  -> vw_nexus_asset_trajectory_v2     78,789 rows
graph_node_opportunity -> vw_nexus_law23_v1                   685 rows
graph_node_capability  -> lifecycle_capability_bundles         77 rows
graph_node_vendor      -> forward_vendor_registry_v1          412 rows

### Edge tables
graph_edge_signal_to_asset              467,197 edges  (MENTIONS: Signal → Asset)
graph_edge_asset_to_owner               272,209 edges  (OWNED_BY: Asset → Company)
graph_edge_asset_to_trajectory           94,959 edges  (HAS_TRAJECTORY: Asset → Trajectory)
graph_edge_trajectory_to_opportunity     78,789 edges  (GENERATES: Trajectory → Opportunity)
graph_edge_opportunity_to_capability      9,510 edges  (REQUIRES: Opportunity → Capability)
graph_edge_capability_to_vendor           1,404 edges  (FULFILLED_BY: Capability → Vendor)
graph_edge_opportunity_to_signal_provenance 26,185 edges (EVIDENCED_BY: Opportunity → Signal)

### Graph traversal status
Signal → Asset → Company:          WORKING
Opportunity → Capability → Vendor: WORKING
Full Signal → Vendor end-to-end:   ZERO ROWS
  Reason: graph_edge_trajectory_to_opportunity has 78,789 edges but
  opportunity_id is null in 99% of rows — no real join to graph_node_opportunity (685 rows)
  This is an architectural gap, not a graph engine fault.

### Navigation views (graph-derived)
vw_nav_entity_neighbors
vw_nav_capability_adjacency
vw_nav_cross_vertical_convergence
vw_nav_operational_clusters
vw_nav_progression_sequences
vw_nav_temporal_transition_paths
vw_nav_vendor_capability_topology

---

## INTELLIGENCE SURFACE (BD APP)

### Primary feed
vw_nexus_opportunity_v2          PRIMARY FEED (Law 23 formation opportunities)
vw_bd_command_briefing           1,066 rows   INTELLIGENCE TAB
vw_stage6_top_opportunities_v1   5,342 rows   TOP OPPORTUNITIES

### Eco intelligence
eco_intelligence_conditions
eco_intelligence_condition_vocabulary
eco_interpretation_entity_state
eco_interpretation_signal_map
eco_interpretation_versions
eco_entity_normalisation_v1
vw_intelligence_condition_surface
vw_intelligence_explainability
vw_condition_distribution
vw_condition_explainability

### Ecosystem views
vw_ecosystem_convergence_surface
vw_ecosystem_layer_density
vw_ecosystem_skew
vw_ecosystem_topology
vw_ecosystem_layer_density
vw_cross_domain_entities
vw_cross_source_progression
vw_dark_domain_priority
vw_capability_domain_coverage

### Governance / audit
vw_gov_capability_mapping_diagnostics
vw_gov_lineage_integrity_diagnostics
vw_gov_ontology_integrity_dashboard
vw_gov_stance_quality_diagnostics
vw_gov_vendor_registry_audit
gov_ontology_integrity_log
gov_vendor_registry_changelog

### Signal diagnostics
vw_signal_class_nulls
vw_signal_lineage_drilldown
vw_low_confidence_edgar_noise
vw_pubmed_ack_validation
vw_regulatory_submission_whitespace
vw_483_warning_letter_risk
vw_substrate_health
vw_evidence_confidence

---

## GUARDIAN & HEALTH MONITORING

nexus-guardian service runs independently (europe-west2)
guardian_runs                    2,026 runs logged
guardian_last_run:               2026-06-11T03:30 UTC
guardian_alerts                  1,841 total alerts
guardian_canonical_check
ingest_guardian_daily_snapshot   last: 2026-06-11
ingest_guardian_pipeline_health  2,923 rows
ingest_guardian_source_health    458,708 rows (209 sources tracked)

nexus_run_log                    1,318 orchestrator runs logged
last_orchestrator_run:           2026-06-11T15:57 UTC (manual trigger)
ingest_orchestrator_heartbeat    2 rows

vw_ingest_health                 189 rows
vw_ingest_run_health
vw_source_staleness
vw_substrate_health
vw_topology_daily_delta
vw_programme_master_v2

---

## ORCHESTRATOR RUNTIME (last nightly: 2026-06-11T18:00 UTC)

Run sequence (each script sequential):
1. ingest_clinical_trials.py    -> TIMEOUT after 3600s every run (known, structural)
2. clinicaltrials_results_ingest.py  -> OK (runs after timeout)
3. ct_protocol_amendments_ingest.py  -> OK (20 pressure events written)
4. clinical_site_network_ingest.py   -> WARN (CT.gov API 400 Bad Request)
5. compassionate_use_ingest.py       -> OK (1 row written)
6. openfda_ingest.py                 -> OK (15 rows written)
7. openfda_drug_label_ingest.py      -> OK
8. openfda_enforcement_ingest.py     -> OK
9. openfda_faers_ingest.py           -> OK
10. fda_warning_letters_ingest.py    -> OK
11. fda_gmp_ingest.py                -> OK
12. fda_drug_approvals_ingest.py     -> OK
13. fda_designations_ingest.py       -> OK (breakthrough_therapy 404 WARN)
14. fda_label_expansions_ingest.py   -> WARN (FDA API 500)
15. fda_orange_book_ingest.py        -> OK (0 rows — existing)
16. fda_rems_ingest.py               -> OK (38 rows written)
17. fda_digital_health_ingest.py     -> OK (33 rows written)
18. ema_chmp_ingest.py               -> OK
19. ema_epar_ingest.py               -> OK
20. ema_orphan_ingest.py             -> WARN (EMA 403/404)
21. ema_referrals_ingest.py          -> WARN (EMA 404)
22. nice_ingest.py                   -> OK (0 rows)
23. europe_hta_ingest.py             -> OK (17 SMC Scotland rows)
24. mhra_refresh_ingest.py           -> OK (0 rows)
25. pmda_ingest.py                   -> OK (0 rows, biologics 404 WARN)

Known structural failures (persistent, not transient):
- compute_vendor_relationships: BQ syntax error (Expected ")" but got keyword NOT [20:22])
- eco_promote_edgar_8k_*/cro_relationships/usaspending/hiring/fda_import_alerts: FAILED
- ep_article_scraper: FAILED

---

## KNOWN BROKEN / OPEN ITEMS

CRITICAL:
1. ingest_clinical_trials.py TIMEOUT every nightly run — nexus-clinical scheduler never enabled
2. compute_vendor_relationships BQ syntax error — persistent, every run
3. /api/ep/league-table 500 error — BQ query failure in ep_routes.py

STRUCTURAL:
4. Full graph traversal Signal→Vendor returns ZERO ROWS (GENERATES edge 99% null opportunity_id)
5. nexus-ingest-daily / nexus-runner-a-hourly / nexus-commercial — PAUSED, NEVER RUN
6. eco_promote_edgar_8k / cro_relationships / usaspending / hiring / fda_import_alerts — all FAILING

STALE:
7. nexus_operational_state last computed 2026-06-05 (7 days stale)
8. observed_at data quality issue — some signals show 2028-10-31

UNWIRED:
9. eco_vendor_stance_conditions (188,958 rows) — built but not connected to API
10. vw_trigger_classifier_v1 / vw_nexus_radar_v1 — not wired to Intelligence Feed
11. radar_explain.js — not appended to nexus-bd index.html
12. nexus-clinical scheduler (/run_clinical endpoint) — never activated

---

## MCP ACCESS FOR AI AGENTS

Claude Desktop MCPs (pkwad machine):
  bigquery    -> nexus-compendium (key.json) — full BQ access
  github      -> pkwadd/nexus-ingest — full repo access
  filesystem  -> C:\Users\pkwad\
  cloud-run   -> europe-west2 (ADC)
  gcp-nexus   -> scheduler_list/pause/resume/update, logging_query

ChatGPT Desktop MCPs (permanent Cloud Run):
  nexus-bq-mcp     https://nexus-bq-mcp-knza2hsdxa-nw.a.run.app/mcp
  nexus-github-mcp https://nexus-github-mcp-knza2hsdxa-nw.a.run.app/mcp

Web Claude (this interface):
  BigQuery, Gmail, Google Drive, Google Calendar (via connectors)
  + Cloud Run read tools (get_service, get_service_log, deploy)

BQ MCP config: --project nexus-compendium --location US --key-file C:\Users\pkwad\key.json
GitHub MCP owner: pkwadd (not pkwaddingham)
FREEZE RULE: Never run two AIs concurrently against live canonical.

---

*Full system map — nexus-compendium.nexus_compendium — 2026-06-12*
*Queried live from BigQuery property graph and all dataset tables*
