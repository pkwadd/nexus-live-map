# NEXUS LIVE SYSTEM MAP
# Source: Live BigQuery + Cloud Run logs — nexus-compendium.nexus_compendium
# Generated: 2026-06-09 | Status: CURRENT

---

## PLATFORM IDENTITY

- Project: nexus-compendium
- Dataset: nexus_compendium
- Region: europe-west2
- GitHub: pkwadd/nexus-ingest (PRIVATE)
- BD App URL: https://nexus-bd-knza2hsdxa-nw.a.run.app
- Ingest URL: https://nexus-ingest-knza2hsdxa-nw.a.run.app

---

## PRODUCT DEFINITION

Nexus is a pharma BD intelligence platform.
It observes the global pharmaceutical market through 466,936 signals
from 107 sources and converts them into ranked commercial opportunity
intelligence for vendors selling into pharma.

Core question answered:
"Which pharma companies are in execution demand right now,
what do they need, and which vendors can fill it?"

ICP: Companies selling recurring systems and execution capability
into pharma operational growth and execution cycles.

---

## CLOUD RUN SERVICES (europe-west2)

nexus-ingest     https://nexus-ingest-knza2hsdxa-nw.a.run.app     Orchestrator + API
nexus-bd         https://nexus-bd-knza2hsdxa-nw.a.run.app         BD frontend app
nexus-api        https://nexus-api-knza2hsdxa-nw.a.run.app         API layer
nexus-ui         https://nexus-ui-knza2hsdxa-nw.a.run.app          UI layer
nexus-guardian   https://nexus-guardian-knza2hsdxa-nw.a.run.app    Guardian service
nexus-trigger    https://nexus-trigger-knza2hsdxa-nw.a.run.app     Trigger service
nexus-runner-a   https://nexus-runner-a-knza2hsdxa-nw.a.run.app    Runner service
nexus-engine-guardian https://nexus-engine-guardian-knza2hsdxa-nw.a.run.app

Cloud Run (us-central1):
nexus-runner-a   https://nexus-runner-a-knza2hsdxa-uc.a.run.app
nexus-runner     https://nexus-runner-knza2hsdxa-uc.a.run.app
nexus-reverse-identity https://nexus-reverse-identity-knza2hsdxa-uc.a.run.app

---

## API ROUTES (nexus-ingest)

/feed            -> vw_nexus_opportunity_v2          PRIMARY INTELLIGENCE FEED
/account         -> nexus_operational_state          ACCOUNT PAGE
/assets          -> vw_nexus_opportunity_v2          ASSET DETAIL
/ep/profile      -> vw_ep_relationship_edges         EP PROFILE
/ep/league-table -> ep_observed_profile_v1           EP LEAGUE (500 ERROR - BROKEN)
/ep/summary      -> ep_observed_profile_v1           EP SUMMARY
/intelligence    -> vw_bd_command_briefing            INTELLIGENCE TAB
/api/narrative   -> claude-haiku-4-5 (Anthropic)     AI NARRATIVE EDGE

---

## BIGQUERY OBJECT COUNTS (live)

BASE TABLE: 75
VIEW: 141
SNAPSHOT: 31
PROCEDURE: 3
PROPERTY GRAPH: 2

---

## LIVE ROW COUNTS (queried 2026-06-09)

action_queue_LOCKED:           2,533,185
graph_node_signal:               466,936
canonical_signals_v3_lock:       466,936
eco_vendor_stance_conditions:    188,958  <- UNWIRED FROM API
nexus_operational_state:          65,684
graph_node_company:               65,684
vw_trigger_classifier_v1:         63,382
forward_vendor_registry_v1:          682
ep_observed_profile_v1:              339
vw_ep_relationship_edges:            122
lifecycle_capability_bundles:         79
graph_node_capability:                77

---

## PROCEDURES (3)

NX_STAGE_INSERT_ONLY       Write signals_stage -> canonical
NX_CANONICAL_INSERT_ONLY   Promote stage to canonical_signals_v3_lock
run_ingest_cycle_v2        Full orchestrator cycle

Write rule: NEVER write to canonical_signals_v3_lock directly.
Always: NX_STAGE_INSERT_ONLY -> NX_CANONICAL_INSERT_ONLY in sequence.

---

## PROPERTY GRAPHS (2)

nexus_graph_v1   created: 2026-05-29   7 nodes, 7 edges
nexus_graph      created: 2026-06-06   7 nodes, 7 edges (CURRENT)

Graph traversal: WORKING as of 2026-06-09

### Graph nodes
Signal      -> graph_node_signal      -> canonical_signals_v3_lock    466,936 rows  LIVE
Company     -> graph_node_company     -> nexus_operational_state        65,684 rows  LIVE
Capability  -> graph_node_capability  -> lifecycle_capability_bundles       77 rows  LIVE
Asset       -> graph_node_asset       -> vw_nexus_asset_trajectory_v2         LIVE
Trajectory  -> graph_node_trajectory  -> vw_nexus_asset_trajectory_v2         LIVE
Opportunity -> graph_node_opportunity -> vw_nexus_law23_v1                     LIVE (725 real opportunities)
Vendor      -> graph_node_vendor      -> forward_vendor_registry_v1            LIVE

### Graph edges
MENTIONS        Signal -> Asset
OWNED_BY        Asset -> Company
HAS_TRAJECTORY  Asset -> Trajectory
GENERATES       Trajectory -> Opportunity
REQUIRES        Opportunity -> Capability
FULFILLED_BY    Capability -> Vendor
EVIDENCED_BY    Opportunity -> Signal

### Edge duplication audit
MENTIONS:        466,936 raw / 210,188 unique  2.2x
OWNED_BY:        272,016 raw /  50,038 unique  5.4x
HAS_TRAJECTORY:   94,783 raw /  78,613 unique  1.2x
GENERATES:        78,613 raw /     725 unique  108x (99% null)
REQUIRES:          9,989 raw /   8,801 unique  1.1x
FULFILLED_BY:      1,404 raw /   1,185 unique  1.2x
EVIDENCED_BY:     27,313 raw /  27,199 unique  1.0x

---

## PIPELINE (RAW -> FRONTEND)

RAW: eco_raw_* / regulatory_fda_landing / sam_gov_raw
  -> ECO STAGING: eco_staged_* -> eco_qualified_* -> eco_interpretation_*
  -> SIGNALS: signals_raw_ingest_guarded_v1 -> signals_stage
  -> [NX_STAGE_INSERT_ONLY -> NX_CANONICAL_INSERT_ONLY]
  -> CANONICAL: canonical_signals_v3_lock (466,936 rows, append-only)
  -> SPINE: vw_signals_engine_ready_v1
  -> LAW 23 PATH: vw_nexus_asset_trajectory_v2 -> vw_nexus_law23_v1
              -> vw_nexus_opportunity_v2 (PRIMARY FEED: 362 co / 815 assets)
  -> FORMATION PATH: vw_trigger_classifier_v1 (NOT YET WIRED TO FEED)
  -> STAGE 6: vw_stage6_vendor_opportunity_match_v2 -> scoring -> ranking
  -> API: Cloud Run nexus-ingest europe-west2

---

## SIGNAL UNIVERSE (466,936 signals)

clinicaltrials_update (existence):          85,781  16,319 cos  35,781 assets
patent_activity (irreversibility):          57,125  15,680 cos
clinicaltrials_backfill (existence):        46,481  10,741 cos  39,422 assets
usaspending (commercial_relationship):      23,714   2,199 cos
eco_edgar_ex10_qualified (commercial):      19,532   1,354 cos
openfda_drug_label (ecosystem):             14,222   1,117 cos   2,156 assets
regulatory_fda (irreversibility):           13,774   1,689 cos   2,340 assets  1939-2026
eco_drug_label_signal (irreversibility):    12,773   1,053 cos
europepmc (existence):                       9,904               9,608 assets
medaff_openalex (publication):               8,083   1,126 cos
openfda_drug_enforcement (ecosystem):        6,319     639 cos
nih_reporter (existence):                    5,193               2,233 assets
eco_fda_approval_signal (irreversibility):   3,624     761 cos   1,165 assets
fda_warning_letters (ecosystem):             2,835   1,951 cos
health_canada (irreversibility):             3,000     379 cos
regulatory_ema_expanded (irreversibility):   2,497     740 cos
fda_rems (irreversibility):                    433     259 cos
eco_exec_appointment_signal (ecosystem):       689     361 cos
manufacturing_capacity (commercial):           423     155 cos
hiring_signal:                                 164      34 cos
EP trade press (30+ sources):               ~1,500

---

## COMPANY STATE ENGINE (65,684 companies)

Contestable  2,831  heat 93.0  30d: 14,553  total: 83,048
Pre-Position 1,759  heat 77.8  30d:  4,100  total: 21,734
Locked         529  heat 53.2  30d:  6,606  total: 28,988
Hardening    3,380  heat 51.6  30d: 22,254  total: 104,772
Formation   57,185  heat 30.5  30d: 37,169  total: 485,801

CONTESTABLE = heat >80, open vendor window, no visible governance
PRE-POSITION = approaching contestable
HARDENING = relationships forming, window narrowing
LOCKED = incumbents in place
FORMATION = early stage, monitoring only

Top contestable (heat=100):
pfizer | astrazeneca | eli lilly | glaxosmithkline | abbvie
bristol-myers squibb | sanofi | merck sharp & dohme | janssen
takeda | amgen | novo nordisk a/s

---

## VENDOR REGISTRY (412 vendors, 8 verticals)

Medical Affairs:       83 vendors  20.1%
Commercial:            66 vendors  16.0%
Data & AI:             58 vendors  14.1%
Patient Engagement:    52 vendors  12.6%
Clinical Operations:   50 vendors  12.1%
Market Access:         49 vendors  11.9%
Regulatory:            44 vendors  10.7%
Clinical Data Systems: 10 vendors   2.4%

74% commercial-operator-driven
15% pipeline-focused
Most connected vendor: Syneos Health (44 capabilities, 6 verticals)

---

## EP INTELLIGENCE

339 execution partners (ep_observed_profile_v1)
122 relationship edges (vw_ep_relationship_edges)
Key EPs: Syneos Health, Lonza, Charles River, Veeva, IQVIA, Medpace,
Certara, Evotec, Icon, Fortrea, Almac, Labcorp, WuXi,
Samsung Biologics, Parexel, AGC Biologics, Eversana

---

## FORMATION CLASSIFIER

63,382 companies classified
NEW: 400 | DISRUPTED: 60 | MAINTAINED: 111 | REMOVED: 186 | UNCLASSIFIED: 62,621

Top Formation companies:
1. Avyxa Holdings — rank 1, 10 assets, 20 events
2. Vertex Pharma  — rank 2,  4 assets, 14 events
3. Novo Nordisk   — rank 3,  5 assets, 14 events

NOT YET WIRED TO INTELLIGENCE FEED.

---

## CT-ABSENT COMMERCIAL POPULATION

316 companies / 1,992 approved assets — invisible to current feed.
Formation is the only observation layer for this population.

Hospital injectables: Hospira (89 assets), Meitheal (66), AM Regent (35)
Ophthalmology: Harrow Eye, Alcon, Bausch, Glaukos
Specialty generics: ANI Pharma (51), Sun Pharma (67), Dr Reddy's (39)
Rare disease: Amicus, Alnylam, Sarepta, Eton, Genzyme, Sentynl
CNS: Neurocrine, Axsome, Sumitomo Pharma America
Oncology: Avyxa Holdings, Eagle Pharma, Acrotech

---

## ORCHESTRATOR STATUS

Scheduler: nexus-ingest-nightly 02:00 UTC daily
BLACKOUT: Do NOT deploy nexus-ingest 02:00-04:00 UTC
CURRENT ISSUE: Container killed mid-run by morning deployments
Last full run: 2026-06-05 (93 OK, 11 failed)

Persistent failures every run:
compute_vendor_relationships (BQ syntax error)
eco_promote_edgar_8k_101/502/direct (FAILED)
eco_promote_cro_relationships (FAILED)
eco_promote_usaspending (FAILED)
eco_promote_hiring_signals (FAILED)
eco_promote_fda_import_alerts (FAILED)
ep_article_scraper (FAILED)

---

## KNOWN BROKEN OBJECTS

vw_vendor_opportunity_scoring_v4  MISSING - breaks vendor scoring chain
vw_action_queue_v1               BROKEN - depends on broken scoring chain
/api/ep/league-table             500 ERROR in ep_routes.py
eco_vendor_stance_conditions     188,958 rows UNWIRED from API
compute_vendor_relationships      BQ syntax error every run

---

## OPEN BUILD ITEMS

1. Wire Formation to Intelligence Feed (Option C approved)
2. Fix /api/ep/league-table 500 error
3. Fix compute_vendor_relationships syntax error
4. Wire eco_vendor_stance_conditions to API
5. Move orchestrator to nexus-runner-a

---

## KEY TABLES

canonical_signals_v3_lock        CORE - append only
nexus_operational_state          CORE - 65,684 company states
forward_vendor_registry_v1       CORE - 412 vendors
lifecycle_capability_bundles     CORE - 77 capabilities
ep_observed_profile_v1           CORE - 339 EPs
action_queue_LOCKED              CORE - 2,533,185 rows
eco_vendor_stance_conditions     BUILT - UNWIRED

---

## KEY VIEWS

vw_nexus_opportunity_v2          PRIMARY FEED
vw_trigger_classifier_v1         FORMATION
vw_nexus_asset_trajectory_v2     TRAJECTORY
vw_nexus_law23_v1                LAW 23
vw_asset_owner_authority_v1      GATE 1
vw_signals_engine_ready_v1       SPINE
vw_stage6_vendor_opportunity_match_v2  STAGE 6
vw_vendor_opportunity_scoring_v6  SCORING
vw_vendor_opportunity_ranked_v6   RANKING
vw_ep_relationship_edges          EP EDGES
vw_bd_command_briefing            INTELLIGENCE TAB

---

## SNAPSHOTS

31 daily: canonical_signals_snapshot_20260414 through 20260609

---
*Live — nexus-compendium.nexus_compendium — 2026-06-09*