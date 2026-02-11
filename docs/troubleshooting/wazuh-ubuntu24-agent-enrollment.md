# Wazuh Agent Enrollment Troubleshooting (Ubuntu 24.04)

This doc captures common issues I hit (or expect to hit) when enrolling a **Wazuh agent on Ubuntu 24.04** from the **Wazuh Dashboard**, plus what to check to get the agent to show as **Active**.

---

## Symptoms

- The Dashboard **“Add agent”** flow is missing, not visible, or behaves unexpectedly.
- The agent installs, but it **does not enroll** with the manager.
- The agent is installed and started, but stays **Disconnected / Never becomes Active** in the Dashboard.
- Enrollment fails unless I change certificate-related settings.

---

## Key Ubuntu 24.04 note: set certificate to `none`

On Ubuntu 24.04, enrollment can fail unless a specific setting is adjusted:

- **Certificate must be set to `none`.**

If you’re following the Dashboard “Add agent” instructions and enrollment fails, this is the first thing I check/try.

> If the “Add agent” option is not showing or the UI looks different on Ubuntu 24, search for a tutorial specifically about:
> “dashboard add agent not showing in ubuntu 24 Wazuh”
> and compare your dashboard version + configuration to what they show.

---

## Baseline checks (before deep debugging)

### 1) Confirm manager reachability from the agent VM
From the Ubuntu 24 agent VM, validate it can reach the Wazuh manager:
- Ping/route to the manager IP (if ICMP is allowed)
- TCP connectivity to the manager’s ports (depends on your Wazuh setup)
- DNS resolution if you’re using hostnames instead of IPs

If the lab VM networking is misconfigured, the agent can’t enroll even with perfect commands.

### 2) Confirm you’re using the correct manager IP for the lab network
In VirtualBox-style homelabs, it’s easy to copy the wrong interface/IP:
- Use the manager’s **LAB-INT / host-only** IP for enrollment (the one reachable from your endpoints).
- Don’t accidentally use a NAT-only address that endpoints can’t reach.

### 3) Time sync matters
If TLS/cert-related flows are involved, time skew can break things:
- Ensure the Ubuntu 24 VM time is correct.
- Ensure the manager time is correct.

---

## If the agent installs but won’t show “Active”

### 1) Confirm the agent service is running
- Check service status and logs on the Ubuntu 24 VM.
- Restart the agent after any config changes.

### 2) Confirm enrollment actually occurred
Depending on how you enrolled (Dashboard-generated command vs manual):
- Verify the agent is registered/known by the manager.
- Verify the agent is pointing at the right manager address in its config.

### 3) Check firewall rules (host + VM)
In a homelab, “it pings but doesn’t connect” is often:
- host firewall
- VM firewall
- incorrect adapter selection (NAT vs host-only vs internal network)

---

## If the Dashboard “Add agent” flow is missing or not showing

This can be version/UI/config related. What I check:
- Dashboard is fully installed and reachable
- I’m logged in with a user/role that can manage agents
- The Wazuh components are healthy (manager/indexer/dashboard)
- Compare with a known-good tutorial for Ubuntu 24 + current Wazuh versions

Search keywords that usually help:
- “Wazuh dashboard add agent not showing Ubuntu 24”
- “Wazuh add agent missing”
- “Wazuh enrollment certificate none Ubuntu 24”

---

## Validation checklist (success criteria)

After applying fixes, I consider it “done” when:
- The Ubuntu 24 agent appears in the Dashboard
- Agent status is **Active**
- I can see fresh events from the Ubuntu endpoint (auth/system activity)

---

## Notes / journal (what I did)

- Installed a Wazuh agent from the Dashboard “Add agent” instructions.
- On Ubuntu 24.04, I had to change a configuration so the **certificate = `none`**.
- After enrollment + any needed VM configuration changes, the agent showed **Active** in the Dashboard.