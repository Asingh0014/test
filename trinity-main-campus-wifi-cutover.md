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

This cutover is a **network segmentation migration**: moving WiFi client
traffic off the flat legacy VLAN (22) onto new segmented VLANs (2300, 2900,
etc.) per SSID/user-group, following successful piloting via the
`Trinity NetSegPilot` / `*2300` / `*2900` duplicate SSID profiles already
present in the controller. `[Confirm: is the intent to cut production SSIDs
over to the pilot VLANs and retire the VLAN-22 profiles, or is this a
broader controller/vendor migration as well?]`

### Scope

- In scope: `[buildings / floors / AP count]` — all SSIDs currently
  broadcasting on VLAN 22 that have a corresponding pilot/segmented profile
  (see table below).
- Out of scope: `[anything explicitly excluded — e.g. guest network, IoT VLAN, residence halls]`

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
  20 clients — `[confirm whether 802.11r variant also needs a VLAN
  2300 counterpart, or whether it gets merged into the main profile as
  part of this cutover]`.

## 3. Current vs. Target State

| Item | Current | Target |
|---|---|---|
| Controller / management platform | `[confirm — appears to be a single controller managing all profiles above]` | Same (no platform change identified) |
| SSID(s) | `Trinity Wireless`, `eduroam`, `Trinity Visitor`, `Trinity Staff`, `Trinity College`, `Trinity IT Test Wireless` on VLAN 22 | Same SSID names, re-pointed to segmented VLANs (2300 / 2900 / other per group) |
| Authentication | 802.1X (staff/students/college/eduroam), OPEN (visitor, cafe, deanery, test PSK) | `[unchanged unless cutover also changes auth — confirm]` |
| VLAN / IP scheme | Flat: VLAN 22 (primary), VLAN 15 (cafe/deanery/misc) | Segmented: VLAN 2300 (primary pilot target), VLAN 2900 (eduroam pilot target), `[VLAN plan for Staff/College/Visitor if different from 2300]` |
| AP hardware/firmware | `[model / version]` | `[unchanged unless AP refresh is part of this project]` |
| DHCP / DNS | Scopes bound to VLAN 22 / 15 | New scopes required for VLAN 2300 / 2900 (and any other new segment) — `[confirm these exist and are sized correctly]` |

## 3. Stakeholders & Communications

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

## 4. Pre-Cutover Checklist

- [ ] Site survey / AP placement confirmed for all affected areas
- [ ] New controller/config fully built and validated in staging/lab
- [ ] New SSID(s), VLANs, and DHCP scopes provisioned and tested
- [ ] RADIUS/802.1X (if applicable) tested end-to-end with target auth method
- [ ] Firmware on all APs updated to target version and verified
- [ ] Switch ports / uplinks confirmed for correct VLAN tagging
- [ ] Backup of current (legacy) controller configuration taken and stored
- [ ] Rollback plan reviewed and confirmed with change owner
- [ ] Change request approved by `[change advisory board / IT leadership]`
- [ ] Comms sent per section 3
- [ ] On-call/support staffing confirmed for cutover window
- [ ] Test devices (laptop, phone) staged for post-cutover validation

## 5. Cutover Steps

> Execute in order. Each step should be checked off and timestamped during
> the actual cutover; capture actual start/end times for the post-cutover
> report.

1. **Freeze changes** — confirm no other network changes are in flight for the campus.
2. **Snapshot legacy config** — export/backup current controller and AP configuration.
3. **Notify support desk** — cutover window has started; escalation contact is `[name/number]`.
4. **Begin phased AP migration** (adjust to actual topology):
   - Building/zone `[A]`: apply new config / move to new controller, verify APs online.
   - Building/zone `[B]`: repeat.
   - Building/zone `[C]`: repeat.
5. **Cut over SSID broadcast** — disable legacy SSID broadcast, confirm target SSID(s) broadcasting correctly.
6. **Validate connectivity** per zone:
   - [ ] AP adoption/online status confirmed in new controller
   - [ ] Client can associate to new SSID
   - [ ] Client receives correct DHCP lease / VLAN
   - [ ] Authentication succeeds (PSK or 802.1X as applicable)
   - [ ] Internet/internal resource access confirmed
   - [ ] Roaming between APs within a zone verified (walk test)
7. **Decommission legacy infrastructure** (only after validation passes) — power down/remove legacy controller or old SSID broadcast entirely.
8. **Close out change window** — notify stakeholders, update status page to "complete."

## 6. Validation / Acceptance Criteria

- [ ] 100% of in-scope APs online and adopted by target controller
- [ ] No coverage gaps vs. pre-cutover baseline (spot-check walk test per building)
- [ ] Client authentication success rate at or above `[baseline %]`
- [ ] Helpdesk ticket volume related to WiFi returns to baseline within `[X hours]`
- [ ] No critical alerts from monitoring/NMS for `[X hours]` post-cutover

## 7. Rollback Plan

**Rollback trigger:** `[e.g. >X% of APs fail to adopt, widespread auth failures, no connectivity in a critical building after Y minutes]`

**Rollback steps:**
1. Re-enable legacy SSID broadcast / re-point APs to legacy controller.
2. Restore legacy controller configuration from backup (section 5, step 2).
3. Verify legacy network restored to pre-cutover functionality.
4. Notify stakeholders and support desk that rollback occurred.
5. Schedule retrospective and reschedule cutover.

## 8. Post-Cutover

- [ ] Send "maintenance complete" comms (section 3)
- [ ] Monitor helpdesk tickets and NMS alerts for `[24–48 hours]`
- [ ] Decommission/archive legacy configuration and hardware per `[retention policy]`
- [ ] Update network documentation / diagrams to reflect new state
- [ ] Hold post-cutover review meeting; log lessons learned below

## 9. Post-Cutover Review

| Item | Notes |
|---|---|
| Actual start/end time | `[ ]` |
| Issues encountered | `[ ]` |
| Rollback invoked? | `[Yes/No]` |
| Follow-up actions | `[ ]` |
