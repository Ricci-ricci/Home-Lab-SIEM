# Roadmap — Cybersecurity Home Lab (SIEM + Purple Team)

This roadmap breaks the lab into milestones you can complete, validate, and iterate on. Treat each checkbox as “done when validated,” not just “installed.”

> Documentation rule: if it isn’t written down (what/why/how-to-validate), it didn’t happen.

---

## How to use this roadmap

For each milestone:
- **Change log:** capture what changed + why
- **Validation:** record proof (commands you ran, screenshots, log entries, alerts)
- **Rollback:** note how you’d undo it if it breaks

---

## Phase 0 — Repo + documentation baseline

- [ ] Create `docs/` structure (guides, troubleshooting, architecture)
- [ ] Keep `README` as an entrypoint (links to docs, goals, quick start)
- [ ] Add a changelog pattern (date + summary + validation)

**Done when:**
- You can navigate the repo in <2 minutes and find “what to do next.”

---

## Phase 1 — Lab network + isolation (safety first)

- [ ] Define network model (Host-Only / Internal / NAT strategy)
- [ ] Ensure vulnerable targets never touch your real LAN
- [ ] Document IP plan and naming conventions
- [ ] Confirm VM-to-VM connectivity on the lab network
- [ ] Confirm controlled internet access (only when needed)

**Validations:**
- `ip a` and `ip route` screenshots/notes for each VM role
- Ping tests between SIEM and endpoints
- “Attack traffic stays on lab network” statement + configuration evidence

---

## Phase 2 — SIEM foundation (Wazuh all-in-one)

- [ ] Build dedicated Wazuh VM (CPU/RAM/Disk documented)
- [ ] Configure stable management IP (prefer static on lab network)
- [ ] Install Wazuh all-in-one (manager + indexer + dashboard)
- [ ] Secure admin access (no secrets committed; store creds safely)
- [ ] Baseline health checks (services up, storage OK)

**Done when:**
- Dashboard is reachable from your admin host on the lab network
- Wazuh core services are running and survive reboot

---

## Phase 3 — Endpoint onboarding (telemetry first)

### Linux endpoint (Ubuntu 24)
- [ ] Install and enroll Wazuh agent
- [ ] Confirm agent shows **Active** in dashboard
- [ ] Confirm auth/system logs arrive

**Known note:**
- Ubuntu 24 may require certificate setting to be `none` for enrollment in some setups.
- If “Add agent” flow doesn’t appear/behaves oddly, follow a focused Wazuh Dashboard troubleshooting guide and document the fix.

### Windows 10 endpoint
- [ ] Install and enroll Wazuh agent
- [ ] (Recommended) Install Sysmon with a known-good config
- [ ] Confirm event channels are ingested (Security, Sysmon, System)

**Done when:**
- Both endpoints are **Active**
- You can locate endpoint events in Wazuh without guessing

---

## Phase 4 — Generate test telemetry (purple-team loop v1)

- [ ] Linux: SSH failed logins, sudo usage, new user creation
- [ ] Windows: failed logons, process creation, service creation (lab-safe)
- [ ] Network: basic scan activity inside the lab (no real LAN exposure)
- [ ] Confirm events appear in Wazuh and can be investigated

**Done when:**
- For each test, you have:
  - what you did
  - expected telemetry
  - where you found it in Wazuh
  - any alerts triggered (or why not)

---

## Phase 5 — Detection tuning + investigator workflow

- [ ] Create a “noise” list (what’s noisy, what’s valuable)
- [ ] Tag/label important hosts (SIEM, endpoints, attacker, vulnerable)
- [ ] Write at least 3 investigation playbooks (short, repeatable steps)
- [ ] Tune rules or dashboards to reduce false positives

**Playbook ideas:**
- Linux auth anomaly triage
- Windows suspicious process triage
- Brute-force pattern investigation

---

## Phase 6 — Architecture realism (v2)

- [ ] Add Windows Server (Active Directory)
- [ ] Domain-join Windows 10
- [ ] Segment lab with pfSense/OPNsense (logging enabled)
- [ ] Centralize DNS/DHCP decisions (document where these live)
- [ ] Confirm identity/auth logs improve detections and investigations

**Done when:**
- You can trace an auth event end-to-end (client → DC → SIEM)

---

## Phase 7 — Vulnerability + exposure management

- [ ] Add vulnerability scanning (e.g., OpenVAS) inside lab-only scope
- [ ] Document scan schedules + scope controls
- [ ] Track remediation tasks (misconfigs, missing patches, weak creds)
- [ ] Validate improvements by re-scanning

---

## Phase 8 — Portfolio-grade outputs

- [ ] “Build journal” entries for each milestone
- [ ] At least 3 incident reports (lab simulations only)
- [ ] At least 3 detection write-ups (logic + validation evidence)
- [ ] Diagrams: network + asset inventory + data flows

**Done when:**
- Someone can clone the repo and understand what you built, why, and how you validated it.

---

## Backlog (nice-to-haves)

- [ ] Add EDR-like endpoint tooling (lab-safe)
- [ ] Add log sources: firewall, web server, DNS, proxy
- [ ] Add Sigma rule mapping / ATT&CK mapping notes
- [ ] Add automated health checks (scripts) and runbooks