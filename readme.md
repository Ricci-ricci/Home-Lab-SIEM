# Home Lab SIEM Project

## Overview

This project is a personal Home Lab implementation of a Security Information and Event Management (SIEM) system. The goal of this environment is to simulate a real-world security operations center (SOC) setup to monitor, detect, and analyze security events within a controlled network.

By ingesting logs from various sources, parsing them, and visualizing the data, this lab serves as a playground for learning about threat detection, incident response, and log analysis.

## Objectives

- **Log Aggregation:** Centralize logs from different endpoints (servers, firewalls, workstations).
- **Threat Detection:** Create and test custom detection rules (e.g., Sigma rules) to identify malicious activity.
- **Visualization:** Build dashboards to visualize network traffic, authentication attempts, and system health.
- **Incident Response:** Simulate attacks and practice investigation workflows.

## Architecture

*Note: This section is a placeholder. As the project grows, list the specific technologies used.*

Common components in this type of setup may include:
- **Log Shippers:** (e.g., Filebeat, Winlogbeat, Syslog-ng)
- **SIEM Core:** (e.g., Elastic Stack (ELK), Splunk, Wazuh, Graylog)
- **Endpoints:** (e.g., Windows Server, Ubuntu, pfSense)

## Getting Started

*(Add instructions here on how to deploy the lab, e.g., via Docker Compose or Ansible scripts)*

1. Clone the repository.
2. Configure environment variables.
3. Start the services.

## Roadmap

- [ ] Set up core SIEM infrastructure.
- [ ] Onboard first log source (e.g., SSH logs or Windows Event Logs).
- [ ] Create basic dashboards.
- [ ] Simulate a brute-force attack and detect it.

## Disclaimer

This project is for educational purposes only. All security testing is performed in an isolated environment owned by the maintainer.