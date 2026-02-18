# FOSSEE Summer Internship – System Administration (IIT Bombay)

This repository documents the complete technical work carried out during my **FOSSEE Summer Internship 2026** under IIT Bombay.

The internship focused on **System Administration, Infrastructure Automation, DevOps, IAM Migration, Email Systems, CI/CD, and AI-powered Security Pipelines**.

---

## 👨‍💻 Intern Details

- **Name:** Srijan Maurya
- **University:** VIT Bhopal University
- **Project:** System Administration
- **Mentors:** Lee Thomas Stephen, Raghavjit Rana
- **Duration:** Summer Internship 2026

---

# 📌 Overview of Work Completed

This repository is structured task-wise and documents the complete implementation, scripts, configurations, and architecture diagrams.

---

# 🔹 Task 1 – Host-Only Network Setup (VirtualBox)

### Objective

Establish stable static communication between host (Windows 11) and Linux VMs.

### Key Work

- Configured `vboxnet0`
- Disabled DHCP
- Static IP assignment via `nmcli`
- Firewall trusted zone configuration
- SSH validation

### Outcome

Stable IP architecture enabling SSO, IAM, and server communication.

---

# 🔹 Task 2 – Drupal + Keycloak SSO (Fully Automated)

### Architecture

- Server A → Keycloak (IAM)
- Server B → Drupal (Web App)
- OpenID Connect Authentication
- CLI-based client automation

### Automation Highlights

- Keycloak client creation using `kcadm.sh`
- Automatic client-secret generation
- Drupal OIDC configuration via `drush`
- Script-driven reproducible setup
- No manual UI dependency

### Technologies

- Rocky Linux 10
- Podman
- Keycloak
- Drupal
- OpenID Connect

---

# 🔹 Task 3 – Keycloak Version Upgrade & Data Migration

### Objective

Upgrade Keycloak v21 → v26.4.0 without losing data.

### Work Done

- Designed backup script (`mysqldump + config archive`)
- Cron-based automated backup
- Fresh environment restore validation
- In-place blue/green upgrade strategy
- JDBC driver migration
- Schema auto-migration validation

### Real-World Challenge Solved

- Proxy misconfiguration causing iframe timeout
- Fixed reverse-proxy mode conflict

---

# 🔹 Task 4 – Mailman 3 Deployment & Migration

### Objective

Migrate from Mailman 2 (CentOS 7) → Mailman 3 (AlmaLinux 10)

### Stack

- Mailman Core
- Postorius
- Hyperkitty
- PostgreSQL
- Nginx
- uWSGI

### Migration Work

- Import21 command usage
- mbox archive ingestion
- Search index rebuilding
- pg_hba authentication fix

### Result

Fully modernized mailing list infrastructure with searchable archives.

---

# 🔹 Task 5 – Postfix + Dovecot + Roundcube Mail Server

### Objective

Deploy lightweight mail server without MySQL dependency.

### Architecture

- Postfix → SMTP
- Dovecot → IMAP + PAM Auth
- Roundcube → Webmail (SQLite backend)
- Maildir storage in user home

### Security

- SELinux configuration
- SASL authentication
- Firewalld rules

### Result

Self-hosted email server with system-user authentication.

---

# 🔹 Task 6 – Automated CI/CD Pipeline (Astro Deployment)

### Objective

Zero-downtime atomic deployment using Jenkins.

### Pipeline Flow

1. GitHub push
2. Webhook trigger
3. Jenkins build (`pnpm`)
4. rsync to production
5. Atomic symlink switch

### DevOps Features

- SSH key-based deployment
- GitHub PAT authentication
- SELinux compatibility
- Timestamped release folders

### Result

Fully automated production deployment pipeline.

---

# 🔹 Task 7 – AI-Powered Intelligent Mail Filter

### Objective

Build multi-layer spam defense system.

### Intelligent Funnel Architecture

Layer 1 – Postfix  
Layer 2 – Rspamd (Heuristic Engine)  
Layer 3 – AI (DistilBERT via Hugging Face)

### Implementation

- Rspamd container via Podman
- FastAPI microservice
- Lua async callback integration
- 99%+ phishing detection confidence
- SMTP rejection integration

### Innovation

Hybrid heuristic + Transformer-based filtering to reduce CPU waste.

---

# 🔹 Task 8 – Kanboard Deployment & Zulip Integration

### Objective

Project management with real-time notifications.

### Stack

- Docker Compose
- Kanboard
- Zulip (Slack compatibility mode)

### Integration

- Incoming Webhook bot
- Slack-compatible API endpoint
- SSL verification workaround (dev mode)
- Live task notifications

---

# 🛠 Technologies Used

- Rocky Linux 10
- AlmaLinux 10
- PostgreSQL
- MySQL
- Keycloak
- Drupal
- Podman / Docker
- Jenkins
- Nginx
- Postfix
- Dovecot
- Roundcube
- Mailman 3
- Rspamd
- FastAPI
- Hugging Face Transformers
- Lua scripting
- Bash automation
- Systemd services

---

# 🏗 Architecture Philosophy

Throughout the internship, the following principles were applied:

- Infrastructure as Code mindset
- Script-driven automation
- Zero manual UI dependency
- Container isolation when necessary
- Atomic deployments
- Reproducibility
- Secure defaults
- Real-world upgrade simulation
- AI-enhanced infrastructure security

---

# 📂 Repository Structure (Example)

```
Host-Only-Network/
Drupal-Keycloak-SSO/
Keycloak-Upgrade/
Mailman-Migration/
Roundcube-Mail-Server/
Astro-CI-CD/
AI-Spam-Filter/
Kanboard-Zulip/
```

---

# 📈 Key Learning Outcomes

- Enterprise IAM migration strategy
- Email infrastructure modernization
- CI/CD production workflows
- Container networking
- SELinux troubleshooting
- Reverse proxy debugging
- Database authentication models
- AI integration into infrastructure
- Secure DevOps pipeline design

---

# 🚀 Final Outcome

This internship resulted in:

- Fully automated SSO architecture
- Production-ready CI/CD pipeline
- Enterprise-grade email server
- Modern mailing list system
- AI-enhanced spam filtering pipeline
- Real-time project notification system    

All deployments were tested, validated, and documented thoroughly.

---

# 📜 License

This repository is maintained for academic and documentation purposes under FOSSEE Internship guidelines.

---
