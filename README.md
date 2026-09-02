# SIEM/Threat Hunting System — ELK Stack Implementation

![Architecture](https://img.shields.io/badge/Architecture-SIEM%2FThreat%20Hunting-blue)
![Stack](https://img.shields.io/badge/Stack-Elastic%20(ELK)-orange)
![Course](https://img.shields.io/badge/Course-NT204.Q22.ANTT-yellow)
![Domain](https://img.shields.io/badge/Domain-Network%20Security-green)
![Status](https://img.shields.io/badge/Status-Completed-brightgreen)

## Overview

This repository contains the final report for the course *Hệ thống Tìm kiếm, Phát hiện và Ngăn ngừa Xâm nhập* (Intrusion Detection & Prevention Systems), University of Information Technology — VNU-HCM.

The project researches and implements a centralized network security monitoring system based on the **ELK Stack** (Elasticsearch, Logstash, Kibana) combined with **Elastic Agent** and **Elastic Defend**, deployed across a simulated enterprise network with segmented DMZ, internal user, database, and SIEM zones behind a pfSense firewall.

## Project Objectives

- Build a centralized log collection pipeline using Elastic Agent across endpoints.
- Normalize and enrich log data through Elasticsearch Ingest Pipelines.
- Enable fast, filtered search of security events via Kibana Discover / KQL.
- Design visual monitoring dashboards for system and security event tracking.
- Deploy endpoint monitoring and EDR capabilities with Elastic Defend.
- Explore anomaly detection using Elastic's built-in Machine Learning features.

## System Architecture

![System Architecture](./architecture.png)

**Data flow:** `Elastic Agent/Fleet → Elasticsearch Ingest Pipeline → Elasticsearch → Kibana`

The simulated network is segmented into:
- **DMZ** — public-facing services (Apache/DVWA web server)
- **Internal User network** — end-user workstations
- **Database network** — restricted-access data stores
- **SIEM network** — centralized ELK Stack for log collection and analysis

## Deployment Scenarios

- Centralized log collection (pfSense via Syslog, Apache access/error logs)
- Log normalization to ECS via custom Ingest Pipelines
- Search and filtering with KQL in Kibana Discover
- Detection & response: ICMP blocking, port scanning, SQL Injection, path traversal, EICAR malware detection
- Statistical monitoring dashboards
- Machine Learning–based anomaly detection (event-rate analysis)
- *Bonus:* log ingestion performance benchmarking across multiple nodes
- *Bonus:* custom log format enrichment into full ECS-compliant JSON

## Results

All experimental scenarios — including ICMP blocking, port scanning, SQL Injection, path traversal, EICAR detection, and ML-based anomaly detection — were successfully implemented and verified on the deployed system. Incident response was also demonstrated through Elastic Defend's host-isolation feature.

## Technologies Used

- Elasticsearch, Logstash, Kibana (ELK Stack)
- Elastic Agent & Fleet
- Elastic Defend (EDR)
- pfSense (firewall)
- Apache HTTP Server, DVWA (test web application)
- Ubuntu, VirtualBox

## Future Work

- Scale to a multi-node Elasticsearch cluster for higher availability and throughput
- Add more log sources: Windows Event Logs, Active Directory, VPN, IDS/IPS, Docker, Kubernetes, Cloud logs
- Test against more realistic malware in a sandboxed environment
- Extend ML detection with network, process, and user-behavior features
- Explore AI/LLM-assisted log analysis and investigation
- Integrate SOAR (Security Orchestration, Automation and Response) for automated incident response
