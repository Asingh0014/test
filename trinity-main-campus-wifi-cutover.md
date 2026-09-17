# Trinity Main Campus – WiFi Cutover Runbook

> **Status:** Draft — fill in the bracketed `[ ]` placeholders with site-specific
> details before this is used as the operational plan. Sections are organized
> so this can also be copy-pasted into a change-management ticket.

## 1. Overview

| Field | Value |
|---|---|
| Site | Trinity Main Campus |
| Change type | WiFi network cutover |
| Change owner | `[Name / team]` |
| Requested by | `[Name / team]` |
| Target cutover date | `[YYYY-MM-DD]` |
| Maintenance window | `[start time]` – `[end time]` `[timezone]` |
| Rollback deadline | `[time by which a go/no-go rollback decision must be made]` |

### Summary

**Confirmed: this is a network segmentation project.** WiFi client traffic
is moving off the flat legacy VLAN (22) onto segmented VLANs, following
successful piloting via the `Trinity NetSegPilot` / `*2300` / `*2900`
duplicate SSID profiles already present in the controller.

**Decisions taken (no strong preference given, so defaulted to the
lowest-risk / most-precedented option — override any of these anytime):**

- **Target VLAN:** all VLAN‑22 SSIDs consolidate onto **VLAN 2300**, matching
  the already-validated `Trinity NetSegPilot` and `Trinity Wireless 2300`
  pilot profiles. No per-SSID VLAN split.
- **VLAN 15 (`SimonWifi_RT380843`, `Trinity Cafe`, `Trinity Deanery`):**
  **out of scope** — no pilot/segmented counterpart exists for VLAN 15, so
  it is left untouched by this project.
- **Rollout approach:** **phased/staged**, not a single big-bang window —
  `Trinity Wireless` alone carries 622 live clients / 125.2 GB of traffic,
  so migration proceeds by building/AP group (see Section 6) with
  validation between stages rather than one campus-wide cutover.

### Scope

- **In scope:** all VLAN‑22 SSIDs — `Trinity Wireless` (incl. the
  `- 802.11R` fast-roaming variant), `eduroam 2026`, `Trinity Visitor`,
  `Trinity Staff`, `Trinity College`, `Trinity IT Test Wireless`,
  `PSK Performance Testing` — migrating to VLAN 2300. Building/floor/AP
  count: `[fill in]`.
- **Out of scope:** VLAN 15 (`SimonWifi_RT380843`, `Trinity Cafe`,
  `Trinity Deanery`) and `eduroam 2900` (already piloted separately on its
  own target VLAN — confirm it stays on 2900 rather than moving to 2300:
  `[confirm]`).

## 2. Current State — SSID Table (as of 2026-09-17)

| Name | SSID | Auth Method | Encryption | Clients | Traffic | VLAN | Tunneled | Status |
|---|---|---|---|---|---|---|---|---|
| PSK Performance Testing | Trinity_RT337220 | OPEN | WPA2/WPA3-M... | 0 | 0 | 22 | APBridg... | Active |
| SimonWifi_RT380843 | SimonWifi_RT380843 | OPEN | WPA3 | 0 | 0 | 15 | APBridg... | Active |
| Trinity Cafe | Trinity Cafe | OPEN | WPA2/WPA3-M... | 0 | 0 | 15 | APBridg... | Active |
| Trinity College | Trinity College | 802.1X | WPA2 | 0 | 0 | 22 | APBridg... | Active |
| Trinity Deanery | Trinity Deanery | OPEN | WPA2/WPA3-M... | 6 | 223.4 MB | 15 | APBridg... | Active |
| Trinity IT Test Wireless | Trinity IT Test Wireless | 802.1X | WPA2 | 0 | 0 | 22 | APBridg... | Active |
| Trinity NetSegPilot | Trinity NetSegPilot | 802.1X | WPA2 | 0 | 0 | 2300 | APBridg... | Active |
| Trinity Staff | Trinity Staff | 802.1X | WPA2 | 0 | 0 | 22 | APBridg... | Active |
| Trinity Visitor | Trinity Visitor | OPEN | NONE | 12 | 521.4 MB | 22 | APBridg... | Active |
| Trinity Wireless | Trinity Wireless | 802.1X | WPA2 | 622 | 125.2 GB | 22 | APBridg... | Active |
| Trinity Wireless - 802.11R | Trinity Wireless | 802.1X | WPA2 | 20 | 1.4 GB | 22 | APBridg... | Active |
| Trinity Wireless 2300 | Trinity Wireless | 802.1X | WPA2 | 0 | 0 | 2300 | APBridg... | Active |
| eduroam 2026 | eduroam | 802.1X | WPA2 | 69 | 8.8 GB | 22 | APBridg... | Active |
| eduroam 2900 | eduroam | 802.1X | WPA2 | 2 | 653.8 KB | 2900 | APBridg... | Active |

> Application Recognition is `Disabled` on all profiles above.

**Read of the table:**
- **VLAN 22** is the current production/legacy VLAN — carries the bulk of
  live traffic: `Trinity Wireless` (622 clients, 125.2 GB), `eduroam 2026`
  (69 clients, 8.8 GB), `Trinity Visitor` (12 clients, 521.4 MB), plus
  `Trinity College`, `Trinity Staff`, `Trinity IT Test Wireless`,
  `PSK Performance Testing` (all 0 active clients).
- **VLAN 15** carries `SimonWifi_RT380843`, `Trinity Cafe`, and
  `Trinity Deanery` (6 clients, 223.4 MB) — `[confirm whether VLAN 15 is in
  scope for this cutover or stays as-is]`.
- **VLAN 2300** is the pilot/target segment already provisioned for two
  profiles: `Trinity NetSegPilot` and `Trinity Wireless 2300` — both
  currently 0 clients, i.e. not yet receiving production traffic.
- **VLAN 2900** is a second pilot/target segment, already live for
  `eduroam 2900` with 2 clients — a small-scale pilot already in progress.
- `Trinity Wireless - 802.11R` is a separate profile (802.11r fast roaming
  enabled) still same-named SSID `Trinity Wireless`, also on VLAN 22, with
  20 clients — **decision: migrates alongside the main `Trinity Wireless`
  profile onto VLAN 2300, keeping 802.11r enabled** (no reason identified
  to drop fast roaming as part of a VLAN change) — `[confirm]`.

## 3. Target SSID → VLAN Mapping

| Name | SSID | Current VLAN | Target VLAN | In scope? |
|---|---|---|---|---|
| PSK Performance Testing | Trinity_RT337220 | 22 | 2300 | Yes |
| Trinity College | Trinity College | 22 | 2300 | Yes |
| Trinity IT Test Wireless | Trinity IT Test Wireless | 22 | 2300 | Yes |
| Trinity Staff | Trinity Staff | 22 | 2300 | Yes |
| Trinity Visitor | Trinity Visitor | 22 | 2300 | Yes |
| Trinity Wireless | Trinity Wireless | 22 | 2300 | Yes |
| Trinity Wireless - 802.11R | Trinity Wireless | 22 | 2300 | Yes |
| Trinity Wireless 2300 *(pilot)* | Trinity Wireless | 2300 | 2300 (becomes production) | Yes — pilot profile retired once main profile cut over |
| Trinity NetSegPilot *(pilot)* | Trinity NetSegPilot | 2300 | — | Retire post-cutover (pilot served its purpose) — `[confirm]` |
| eduroam 2026 | eduroam | 22 | 2300 | Yes |
| eduroam 2900 *(pilot)* | eduroam | 2900 | 2900 (unchanged) | Out of scope — stays on existing pilot VLAN — `[confirm]` |
| SimonWifi_RT380843 | SimonWifi_RT380843 | 15 | 15 (unchanged) | No — VLAN 15 out of scope |
| Trinity Cafe | Trinity Cafe | 15 | 15 (unchanged) | No — VLAN 15 out of scope |
| Trinity Deanery | Trinity Deanery | 15 | 15 (unchanged) | No — VLAN 15 out of scope |

## 4. Current vs. Target State

| Item | Current | Target |
|---|---|---|
| Controller / management platform | `[confirm — appears to be a single controller managing all profiles above]` | Same (no platform change identified) |
| SSID(s) | `Trinity Wireless`, `eduroam`, `Trinity Visitor`, `Trinity Staff`, `Trinity College`, `Trinity IT Test Wireless`, `PSK Performance Testing` on VLAN 22 | Same SSID names, all re-pointed to VLAN 2300 |
| Authentication | 802.1X (staff/students/college/eduroam), OPEN (visitor, test PSK) | Unchanged — segmentation only, no auth-method change |
| VLAN / IP scheme | Flat: VLAN 22 (primary, in-scope SSIDs) | Segmented: VLAN 2300 (all in-scope SSIDs consolidate here) |
| AP hardware/firmware | `[model / version]` | Unchanged — no AP refresh in this project |
| DHCP / DNS | Scopes bound to VLAN 22 | VLAN 2300 scope already exists (pilot profiles are live on it) — `[confirm capacity is sized for ~700+ concurrent clients once production moves over, not just pilot-scale]` |

## 5. Stakeholders & Communications

| Role | Name | Responsibility |
|---|---|---|
| Change owner | `[ ]` | Overall accountability, go/no-go decision |
| Network engineer(s) | `[ ]` | Execute config changes on APs/controller |
| Helpdesk/IT support | `[ ]` | Front-line support during and after cutover |
| Facilities | `[ ]` | Physical access to IDF/MDF rooms if needed |
| Campus comms | `[ ]` | Sends user-facing notices |

### Communications plan

- `[T-7 days]` — Notify campus community of upcoming maintenance window (email/portal banner).
- `[T-1 day]` — Reminder notice + expected impact (brief outage, need to reconnect to new SSID).
- `[T-0, start]` — "Maintenance in progress" notice posted to status page.
- `[T-0, end]` — "Maintenance complete" notice + instructions for reconnecting to new SSID if applicable.
- `[T+1 day]` — Follow-up notice if issues are still being triaged.

## 6. Pre-Cutover Checklist

- [ ] Site survey / AP placement confirmed for all affected areas
- [ ] VLAN 2300 DHCP scope re-validated for full production scale (~700+
      concurrent clients — currently only pilot-scale traffic), not just
      pilot capacity
- [ ] RADIUS/802.1X tested end-to-end against VLAN 2300 for each in-scope
      SSID (Trinity Wireless, Staff, College, IT Test, eduroam)
- [ ] Switch ports / uplinks confirmed for VLAN 2300 tagging on all
      affected APs/zones
- [ ] Backup of current controller configuration (all SSID profiles) taken
      and stored
- [ ] Rollback plan reviewed and confirmed with change owner
- [ ] Change request approved by `[change advisory board / IT leadership]`
- [ ] Comms sent per Section 5
- [ ] On-call/support staffing confirmed for each rollout stage
- [ ] Test devices (laptop, phone, one per auth type — 802.1X and OPEN)
      staged for post-migration validation per stage

## 7. Cutover Steps (Phased Rollout)

> Migrate by building/AP group, not all at once — `Trinity Wireless` alone
> has 622 live clients. Complete and validate each stage before starting
> the next. Timestamp each step during execution.

1. **Freeze changes** — confirm no other network changes are in flight for the campus.
2. **Snapshot config** — export/backup current SSID profiles (VLAN 22 and 2300 pilot profiles).
3. **Notify support desk** — stage 1 window has started; escalation contact is `[name/number]`.
4. **Stage rollout by building/AP group** — order and grouping: `[fill in, e.g. lowest-traffic building first, Deanery last]`:
   - Stage 1 — Building/zone `[A]` (start with a low-traffic/low-risk building, e.g. wherever `PSK Performance Testing` or `Trinity IT Test Wireless` already run): re-point APs to broadcast VLAN 2300 for all in-scope SSIDs.
   - Stage 2 — Building/zone `[B]`: repeat.
   - Stage 3 — Building/zone `[C]` (highest-traffic building last, once earlier stages are clean).
5. **Retire the pilot-only profile** once its production counterpart is confirmed stable: merge or decommission `Trinity Wireless 2300` (pilot) in favor of the migrated `Trinity Wireless` profile, and retire `Trinity NetSegPilot` once its purpose is served — `[confirm timing]`.
6. **Validate connectivity** per stage before proceeding to the next:
   - [ ] AP(s) in this stage broadcasting VLAN 2300 correctly for all in-scope SSIDs
   - [ ] Client can associate to each SSID (802.1X and OPEN types)
   - [ ] Client receives VLAN 2300 DHCP lease
   - [ ] Authentication succeeds (PSK / 802.1X as applicable)
   - [ ] Internet/internal resource access confirmed
   - [ ] Roaming between APs within the stage verified (walk test), including 802.11r on `Trinity Wireless`
   - [ ] No unexpected drop in client/traffic counts vs. pre-migration baseline for this zone
7. **Repeat steps 4–6** for each remaining stage.
8. **Decommission VLAN 22 broadcast** for in-scope SSIDs only after all stages pass validation.
9. **Close out change window** — notify stakeholders, update status page to "complete."

## 8. Validation / Acceptance Criteria

- [ ] All in-scope SSIDs broadcasting on VLAN 2300 campus-wide, VLAN 22 profile retired for those SSIDs
- [ ] `Trinity Wireless` client count on VLAN 2300 recovers to ~pre-cutover baseline (622 clients) after full rollout
- [ ] No coverage gaps vs. pre-cutover baseline (spot-check walk test per building)
- [ ] Client authentication success rate at or above `[baseline %]`
- [ ] Helpdesk ticket volume related to WiFi returns to baseline within `[X hours]` per stage
- [ ] No critical alerts from monitoring/NMS for `[X hours]` post-cutover per stage
- [ ] VLAN 15 and `eduroam 2900` unaffected throughout (out of scope — spot-check no regression)

## 9. Rollback Plan

**Rollback trigger (per stage):** `[e.g. auth failure rate above X%, no connectivity in the stage's building after Y minutes, DHCP scope exhaustion on VLAN 2300]`

**Rollback steps (per stage):**
1. Re-point the affected building/AP group's SSIDs back to VLAN 22.
2. Restore config from backup (Section 7, step 2) for that stage if needed.
3. Verify VLAN 22 connectivity restored to pre-cutover functionality for that stage.
4. Notify stakeholders and support desk that rollback occurred for that stage.
5. Root-cause before re-attempting that stage; later stages pause until resolved.

## 10. Post-Cutover

- [ ] Send "maintenance complete" comms (Section 5) once all stages finish
- [ ] Monitor helpdesk tickets and NMS alerts for `[24–48 hours]` after the final stage
- [ ] Decommission/archive retired VLAN‑22 SSID profiles and pilot profiles (`Trinity NetSegPilot`, `Trinity Wireless 2300`) per `[retention policy]`
- [ ] Update network documentation / diagrams to reflect VLAN 2300 as production
- [ ] Hold post-cutover review meeting; log lessons learned below

## 11. Post-Cutover Review

| Item | Notes |
|---|---|
| Actual start/end time | `[ ]` |
| Issues encountered | `[ ]` |
| Rollback invoked? | `[Yes/No]` |
| Follow-up actions | `[ ]` |
