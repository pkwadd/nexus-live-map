# NEXUS LIVE SYSTEM MAP
# Source: Live BigQuery + Cloud Run logs — nexus-compendium.nexus_compendium
# Generated: 2026-06-12 | Status: CURRENT

---

## PLATFORM IDENTITY

- Project: nexus-compendium
- Dataset: nexus_compendium
- Region: europe-west2
- GitHub: pkwadd/nexus-ingest (PRIVATE)
- BD App URL: https://nexus-bd-542997761585.europe-west2.run.app
- Ingest URL: https://nexus-ingest-542997761585.europe-west2.run.app

---

## PRODUCT DEFINITION

Nexus is a pharma BD intelligence platform operating under the Radar Doctrine (Contract v2.3).
It observes the global pharmaceutical market through 467,197 signals
from 209 sources and converts them into ranked commercial opportunity
intelligence for vendors selling into pharma.

Core question answered:
"What is becoming true in pharma, and what capability will it require?"

ICP: Companies selling recurring systems and execution capability
into pharma operational growth and execution cycles.

Contract: NEXUS_CONTRACT_UNIFIED_V2_2.md (pkwadd/nexus-ingest) — LOCKED

---

## CLOUD RUN SERVICES (europe-west2)

nexus-ingest    https://nexus-ingest-542997761585.europe-west2.run.app    Orchestrator + API
nexus-bd        https://nexus-bd-542997761585.europe-west2.run.app        BD frontend app
nexus-guardian  https://nexus-guardian-542997761585.europe-west2.run.app  Guardian service
nexus-bq-mcp    https://nexus-bq-mcp-knza2hsdxa-nw.a.run.app             BigQuery MCP (ChatGPT)
nexus-github-mcp https://nexus-github-mcp-knza2hsdxa-nw.a.run.app        GitHub MCP (ChatGPT)

NOTE: Service URLs changed 2026-06-11 after gcloud services update.
New pattern: nexus-*-542997761585.europe-west2.run.app

---

## SCHEDULER JOBS (europe-west2)

nexus-ingest-nightly   0 18 * * *    ENABLED   last: 2026-06-11T18:00  /run
nexus-guardian-check   */15 * * * *  ENABLED   last: 2026-06-11T23:45  /check
nexus-guardian-snapshot 0 6 * * *   ENABLED   last: 2026-06-11T06:00  /snapshot
nexus-ingest-daily     0 2 * * *    PAUSED    never run
nexus-runner-a-hourly  5 * * * *    PAUSED    never run
nexus-commercial       2-59/5 * * * * PAUSED  never run
nexus-clinical         */5 * * * *  PAUSED    never run               /run_clinical

ACTIVE TRIGGER: nexus-ingest-nightly fires 18:00 UTC (01:00 Saigon)
BLACKOUT: Do NOT deploy nexus-ingest 18:00-22:00 UTC

---

## API ROUTES (nexus-ingest)

/feed            -> vw_nexus_opportunity_v2          PRIMARY INTELLIGENCE FEED
/account         -> nexus_operational_state          ACCOUNT PAGE
/assets          -> vw_nexus_opportunity_v2          ASSET DETAIL
/ep/profile      -> vw_ep_relationship_edges         EP PROFILE
/ep/league-table -> ep_observed_profile_v1           EP LEAGUE (500 ERROR - BROKEN)
/ep/summary      -> ep_observed_profile_v1           EP SUMMARY
/intelligence    -> vw_bd_command_briefing            INTELLIGENCE TAB
/api/narrative   -> claude-sonnet-4-6 (Anthropic)    AI NARRATIVE EDGE

---

## BIGQUERY OBJECT COUNTS (live)

BASE TABLE: 75
VIEW: 141
SNAPSHOT: 31
PROCEDURE: 3
PROPERTY GRAPH: 2

---

## LIVE ROW COUNTS (queried 2026-06-12)

canonical_signals_v3:          467,197  (209 sources, 63,415 companies, 94,960 assets)
eco_signals:                    62,596
nexus_operational_state:        65,684 rows (last computed: 2026-06-05)
  state_companies:              14,288 distinct companies
  heat >= 70:                    1,380
  heat 40-70:                      910
  heat < 40:                    11,998
forward_vendor_registry_v1:        682
ep_observed_profile_v1:            339
vw_nexus_asset_trajectory_v2:   78,789
vw_nexus_law23_formation_v1:       646
vw_nexus_formation_opportunity_v1: 647
vw_vendor_opportunity_scoring_v6: 82,574

NOTE: latest_signal_date shows 2028-10-31 — data quality issue in observed_at field.

---

## SIGNAL CLASS BREAKDOWN

existence:               190,457   40.8%
irreversibility:         103,838   22.2%
ecosystem:                62,596   13.4%
commercial_relationship:  44,397    9.5%
null:                     29,352    6.3%
publication:              27,953    6.0%
publication_activity:      3,604    0.8%
eu_marketing_authorisation: 2,313   0.5%
market_exit:               1,054    0.2%
ep_article_scan:             687
ep_trade_press:              383
terminated:                  372
vendor_partnership_announcement: 68
operational_hiring:           62

---

## TOP SIGNAL SOURCES

clinicaltrials_update:         88,826
patent_activity:               82,564
clinicaltrials_backfill:       46,481
usaspending:                   23,723
eco_edgar_ex10_qualified:      22,210
openfda_drug_label:            14,322
regulatory_fda:                13,874
eco_drug_label_signal:         12,773
europepmc:                     10,225
medaff_openalex:               10,183
pubmed:                         8,487
medaff_europepmc:               7,674
openfda_drug_enforcement:       6,319
nih_reporter:                   6,195
openfda_faers:                  5,100
fda_fei_registration:           5,008
eco_fda_approval_signal:        3,624

---

## PROCEDURES (3)

NX_STAGE_INSERT_ONLY       Write signals_stage -> canonical
NX_CANONICAL_INSERT_ONLY   Promote stage to canonical_signals_v3_lock
run_ingest_cycle_v2        Full orchestrator cycle

Write rule: NEVER write to canonical_signals_v3_lock directly.
Always: NX_STAGE_INSERT_ONLY -> NX_CANONICAL_INSERT_ONLY in sequence.

---

## PROPERTY GRAPHS (2)

nexus_graph_v1   created: 2026-05-30   7 nodes, 7 edges
nexus_graph      created: 2026-06-06   7 nodes, 7 edges (CURRENT)

### Graph nodes
Signal      -> graph_node_signal      -> canonical_signals_v3_lock
Company     -> graph_node_company     -> nexus_operational_state
Capability  -> graph_node_capability  -> lifecycle_capability_bundles
Asset       -> graph_node_asset       -> vw_nexus_asset_trajectory_v2
Trajectory  -> graph_node_trajectory  -> vw_nexus_asset_trajectory_v2
Opportunity -> graph_node_opportunity -> vw_nexus_law23_v1             (725 real opportunities)
Vendor      -> graph_node_vendor      -> forward_vendor_registry_v1

### Graph edges
MENTIONS        Signal -> Asset
OWNED_BY        Asset -> Company
HAS_TRAJECTORY  Asset -> Trajectory
GENERATES       Trajectory -> Opportunity   (108x duplication, 99% null opportunity_id)
REQUIRES        Opportunity -> Capability
FULFILLED_BY    Capability -> Vendor
EVIDENCED_BY    Opportunity -> Signal

Sub-chain traversals: WORKING (Signal->Asset->Company, Opportunity->Capability->Vendor)
Full end-to-end Signal->Vendor: ZERO ROWS (null opportunity_id in GENERATES)

---

## PIPELINE (RAW -> FRONTEND)

RAW: signals_raw_ingest_guarded_v1
  -> signals_stage
  -> [NX_STAGE_INSERT_ONLY -> NX_CANONICAL_INSERT_ONLY]
  -> CANONICAL: canonical_signals_v3 (467,197 rows, append-only)
  -> SPINE: vw_signals_engine_ready_v1
  -> LAW 23 PATH: vw_nexus_asset_trajectory_v2
               -> vw_nexus_law23_formation_v1 (646)
               -> vw_nexus_formation_opportunity_v1 (647)
               -> vw_nexus_opportunity_v2 (PRIMARY FEED)
  -> FORMATION PATH: vw_trigger_classifier_v1 (NOT YET WIRED TO FEED)
  -> STAGE 6: vw_stage6_vendor_opportunity_match_v2
           -> vw_vendor_opportunity_scoring_v6 (82,574 rows)
           -> ranking
  -> API: Cloud Run nexus-ingest europe-west2

---

## COMPANY STATE ENGINE

nexus_operational_state: 14,288 companies (last run: 2026-06-05)
heat >= 70 (high):  1,380
heat 40-70 (mid):     910
heat < 40 (low):   11,998

State engine has NOT run since 2026-06-05. Stale by 7 days.

---

## EP INTELLIGENCE

339 execution partners (ep_observed_profile_v1)
682 vendors registered (forward_vendor_registry_v1)
82,574 vendor-scored opportunities (vw_vendor_opportunity_scoring_v6)

---

## FORMATION CLASSIFIER (vw_trigger_classifier_v1)

Four-bucket deterministic model (no AI):
NEW / DISRUPTED / MAINTAINED / REMOVED

Approved NEW sources:
  eco_fda_approval_signal, eco_ema_approval_signal,
  eco_ema_expanded_signal, eco_who_drug_signal

DISRUPTED: detected via literal "Refusal"/"Withdrawal" in eco_ema_approval_signal.title
NOT YET WIRED TO INTELLIGENCE FEED.

---

## ORCHESTRATOR STATUS

Trigger: nexus-ingest-nightly — 18:00 UTC daily
Last run: 2026-06-11T18:00 UTC
Run result: clinical trials TIMEOUT after 3600s, rest of pipeline completes

Known timeout: ingest_clinical_trials.py always times out (60min Cloud Run limit)
Fix needed: split clinical trials to nexus-clinical scheduler (paused, never run)

---

## KNOWN BROKEN / OPEN ITEMS

1. /api/ep/league-table — 500 error (BQ query failure in ep_routes.py)
2. compute_vendor_relationships — BQ syntax error (Expected ")" but got keyword NOT at [20:22])
3. nexus-clinical scheduler — PAUSED, never run (would fix clinical trials timeout)
4. nexus-ingest-daily / nexus-runner-a-hourly / nexus-commercial — all PAUSED, never run
5. radar_explain.js — NOT wired into nexus-bd index.html
6. state engine stale — nexus_operational_state last computed 2026-06-05
7. Formation classifier not wired to Intelligence Feed
8. Full graph traversal Signal->Vendor returns zero rows (GENERATES edge null population)
9. latest_signal_date shows 2028 — data quality issue in observed_at

---

## KEY TABLES

canonical_signals_v3_lock        CORE - append only
nexus_operational_state          CORE - 14,288 company states (stale: 2026-06-05)
forward_vendor_registry_v1       CORE - 682 vendors
lifecycle_capability_bundles     CORE - 77 capabilities
ep_observed_profile_v1           CORE - 339 EPs
action_queue_LOCKED              CORE
eco_vendor_stance_conditions     BUILT - UNWIRED

---

## KEY VIEWS

vw_nexus_opportunity_v2                PRIMARY FEED
vw_trigger_classifier_v1               FORMATION CLASSIFIER
vw_nexus_asset_trajectory_v2           TRAJECTORY (78,789 assets)
vw_nexus_law23_formation_v1            LAW 23 (646)
vw_nexus_formation_opportunity_v1      FORMATION OPPORTUNITIES (647)
vw_asset_owner_authority_v1            GATE 1
vw_signals_engine_ready_v1             SPINE
vw_stage6_vendor_opportunity_match_v2  STAGE 6
vw_vendor_opportunity_scoring_v6       SCORING (82,574)
vw_ep_relationship_edges               EP EDGES
vw_bd_command_briefing                 INTELLIGENCE TAB

---

## MCP ACCESS

Claude Desktop MCPs: bigquery, github, filesystem, cloud-run, gcp-nexus
ChatGPT MCPs: nexus-bq-mcp, nexus-github-mcp (both permanent Cloud Run services)
Web Claude MCPs: BigQuery, Gmail, Google Drive, Google Calendar (connectors)

BigQuery MCP args: --project nexus-compendium --location US --key-file C:\Users\pkwad\key.json
GitHub MCP owner: pkwadd (not pkwaddingham)

---

*Live — nexus-compendium.nexus_compendium — 2026-06-12*
