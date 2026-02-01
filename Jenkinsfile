pipeline {
  agent any

  options {
    timestamps()
    ansiColor('xterm')
  }

  parameters {
    booleanParam(name: 'RUN_ANSIBLE', defaultValue: false, description: 'Run Ansible PostgreSQL install/config playbook')
    booleanParam(name: 'RUN_SQL_HEALTHCHECK', defaultValue: false, description: 'Run SQL health check against a PostgreSQL host')
    string(name: 'PG_HOST', defaultValue: '127.0.0.1', description: 'PostgreSQL host for SQL health check')
    string(name: 'PG_PORT', defaultValue: '5432', description: 'PostgreSQL port')
    string(name: 'PG_DB', defaultValue: 'postgres', description: 'Database name')
    string(name: 'PG_USER', defaultValue: 'postgres', description: 'Database user')
    password(name: 'PG_PASSWORD', defaultValue: 'postgres', description: 'Database password (lab default)')
  }

  environment {
    // Keeps psql non-interactive
    PGPASSWORD = "${params.PG_PASSWORD}"
  }

  stages {

    stage('Checkout') {
      steps {
        checkout scm
      }
    }

    stage('Quick Repo Validation') {
      steps {
        sh '''
          set -euo pipefail
          echo "== Shell scripts (syntax check) =="
          if ls scripts/bash/*.sh >/dev/null 2>&1; then
            for f in scripts/bash/*.sh; do
              echo "Checking: $f"
              bash -n "$f"
            done
          else
            echo "No bash scripts found under scripts/bash/ - skipping"
          fi

          echo "== YAML presence check =="
          if ls ansible/playbooks/*.yml >/dev/null 2>&1; then
            echo "Ansible playbooks found."
          else
            echo "No Ansible playbooks found - skipping"
          fi
        '''
      }
    }

    stage('Run Ansible (Optional)') {
      when { expression { return params.RUN_ANSIBLE } }
      steps {
        sh '''
          set -euo pipefail
          echo "== Running Ansible Playbook =="
          cd ansible

          # Basic sanity
          ansible --version

          # Validate inventory connectivity
          ansible -m ping dbservers

          # Run playbook
          ansible-playbook playbooks/install_postgresql17_rocky.yml
        '''
      }
    }

    stage('Run SQL Health Check (Optional)') {
      when { expression { return params.RUN_SQL_HEALTHCHECK } }
      steps {
        sh '''
          set -euo pipefail
          echo "== SQL Health Check =="
          if [ ! -f "scripts/sql/postgres_basic_health.sql" ]; then
            echo "scripts/sql/postgres_basic_health.sql not found - skipping"
            exit 0
          fi

          # Require psql client
          if ! command -v psql >/dev/null 2>&1; then
            echo "psql not found on Jenkins agent. Install postgresql client package."
            exit 1
          fi

          psql -h "${PG_HOST}" -p "${PG_PORT}" -U "${PG_USER}" -d "${PG_DB}" -f scripts/sql/postgres_basic_health.sql
        '''
      }
    }
  }

  post {
    always {
      sh '''
        echo "== Workspace summary =="
        ls -la
      '''
    }
    failure {
      echo "Pipeline failed. Check console output for errors."
    }
    success {
      echo "Pipeline completed successfully."
    }
  }
}
