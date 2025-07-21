pipeline {
  agent any

  environment {
    VAULT_ADDR          = credentials('VAULT_ADDR')              // e.g. https://vault.sslip.io
    VAULT_ROLE_ID       = credentials('VAULT_ROLE_ID')   // From vault role-id read
    VAULT_WRITER_TOKEN  = credentials('VAULT_WRITER_TOKEN')      // Token to generate secret_id
  }

  stages {
    stage('Vault Login & Fetch Jenkins Secret') {
      steps {
        script {
          // 1. Generate secret_id
          def sidResp = sh(script: """
            curl -s --header "X-Vault-Token: ${VAULT_WRITER_TOKEN}" \\
              --request POST \\
              ${VAULT_ADDR}/v1/auth/approle/role/jenkins-role/secret-id
          """, returnStdout: true).trim()

          def secret_id = new groovy.json.JsonSlurperClassic()
                            .parseText(sidResp).data.secret_id

          echo "✅ Jenkins Secret_id: ${secret_id}" 

          // 2. Login to Vault
          def loginResp = sh(script: """
            curl -s --request POST \\
              --data '{ "role_id": "${VAULT_ROLE_ID}", "secret_id": "${secret_id}" }' \\
              ${VAULT_ADDR}/v1/auth/approle/login
          """, returnStdout: true).trim()

          def vault_token = new groovy.json.JsonSlurperClassic()
                              .parseText(loginResp).auth.client_token

          // 3. Fetch Jenkins secret from kv/jenkins (KV v2 uses /data/)
          def secretResp = sh(script: """
            curl -s --header "X-Vault-Token: ${vault_token}" \\
              ${VAULT_ADDR}/v1/kv/data/jenkins
          """, returnStdout: true).trim()

          def secretData = new groovy.json.JsonSlurperClassic()
                             .parseText(secretResp).data.data

          echo "✅ Jenkins Secret Key: ${secretData['api-token']}"  // Change key as needed
        }
      }
    }
  }
}
