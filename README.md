# Jenkins (CI/CD Basics for PostgreSQL DBA)

This folder contains a simple Jenkins pipeline that demonstrates CI/CD
concepts using PostgreSQL DBA automation examples.

## Files
- `Jenkinsfile` - Jenkins pipeline definition

## What the Pipeline Does
1) **Checkout** the repository
2) **Quick Validation**
   - Bash script syntax checks (if any exist under `scripts/bash/`)
   - Detects Ansible playbooks
3) **Optional: Run Ansible**
   - Pings inventory group `dbservers`
   - Runs `ansible/playbooks/install_postgresql17_rocky.yml`
4) **Optional: Run SQL Health Check**
   - Runs `scripts/sql/postgres_basic_health.sql` using `psql`

## Parameters
- `RUN_ANSIBLE` (true/false): run Ansible PostgreSQL install/config playbook
- `RUN_SQL_HEALTHCHECK` (true/false): run SQL checks via psql
- `PG_HOST`, `PG_PORT`, `PG_DB`, `PG_USER`, `PG_PASSWORD`: connection settings (lab defaults)

## Requirements (Jenkins Agent)
- `bash`
- `ansible` (only if using RUN_ANSIBLE)
- `psql` client (only if using RUN_SQL_HEALTHCHECK)
- SSH connectivity to the target node(s) defined in `ansible/inventory/hosts.ini` (for Ansible runs)

## How to Use
1) Create a Jenkins Pipeline job
2) Point it to this repository
3) Jenkins will automatically detect and run the `Jenkinsfile`
4) Enable optional stages using parameters
