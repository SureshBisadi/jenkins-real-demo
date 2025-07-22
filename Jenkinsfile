pipeline {
  agent any


  stages {
    stage('Fetch Secret using Vault Agent Token') {
      steps {
        sh '''
          #!/bin/bash
          echo "🔐 Step 1: Reading Vault token from agent..."

          if [ -f /etc/vault/token ]; then
            echo "✅ Vault token file exists."
            VAULT_TOKEN=$(cat /etc/vault/token)
            echo "✅ VAULT_TOKEN read from file: $VAULT_TOKEN"
          else
            echo "❌ Vault token file does not exist!"
            exit 1
          fi

          echo "📦 Step 2: Fetching secret from Vault..."
          export VAULT_TOKEN
          vault kv get kv/jenkins
        '''
      }
    }
  }
}
