pipeline {
    agent any
    environment {
        API_NAME     = "loan-api"
        API_VERSION  = "1.0.0"
        PRODUCT_NAME = "loan-product"
        CATALOG_NAME = "prod"
        SERVER       = "https://small-1-mgmt-api-manager-cp4i.apps.ocp.prontefflabs.com"
        ORG          = "indusapi-np"
        USERNAME     = "umesh"
        PASSWORD     = "!n0r1t5@C"
        REALM        = "provider/default-idp-2"
    }

    stages {
        stage('Login') {
            steps {
                sh """
                  echo "🔐 Logging in to API Connect..."
                  apic login \
                    --server "${SERVER}" \
                    --username "${USERNAME}" \
                    --password "${PASSWORD}" \
                    --realm "${REALM}"
                """
            }
        }

        stage('Validate API') {
            steps {
                sh """
                  echo "✅ Validating API YAML..."
                  apic validate apis/${API_NAME}/${API_NAME}_${API_VERSION}.yaml
                """
            }
        }

        stage('Check API') {
            steps {
                sh """
                  echo "🔎 Checking if API exists..."
                  apic draft-apis:get \
                    --server "${SERVER}" \
                    --org "${ORG}" \
                    ${API_NAME}:${API_VERSION} || true
                """
            }
        }

        stage('Create or Update API') {
            steps {
                script {
                    def exists = sh(
                        script: """
                          apic draft-apis:get \
                            --server "${SERVER}" \
                            --org "${ORG}" \
                            ${API_NAME}:${API_VERSION} >/dev/null 2>&1 && echo true || echo false
                        """,
                        returnStdout: true
                    ).trim()

                    if (exists == "true") {
                        sh """
                          echo "♻️ Updating API..."
                          apic draft-apis:update \
                            --server "${SERVER}" \
                            --org "${ORG}" \
                            ${API_NAME}:${API_VERSION} apis/${API_NAME}/${API_NAME}_${API_VERSION}.yaml
                        """
                    } else {
                        sh """
                          echo "🆕 Creating API..."
                          apic draft-apis:create \
                            --server "${SERVER}" \
                            --org "${ORG}" \
                            apis/${API_NAME}/${API_NAME}_${API_VERSION}.yaml
                        """
                    }
                }
            }
        }

        stage('Fix Product References') {
            steps {
                sh """
                  echo "🔧 Fixing API references in product YAML..."
                  bash /var/lib/jenkins/scripts/fix-refs.sh products/${PRODUCT_NAME} apis
                """
            }
        }

        stage('Check Product') {
            steps {
                sh """
                  echo "🔎 Checking if Product exists..."
                  apic draft-products:get \
                    --server "${SERVER}" \
                    --org "${ORG}" \
                    ${PRODUCT_NAME}:${API_VERSION} || true
                """
            }
        }

        stage('Create or Update Product') {
            steps {
                script {
                    def prodExists = sh(
                        script: """
                          apic draft-products:get \
                            --server "${SERVER}" \
                            --org "${ORG}" \
                            ${PRODUCT_NAME}:${API_VERSION} >/dev/null 2>&1 && echo true || echo false
                        """,
                        returnStdout: true
                    ).trim()

                    if (prodExists == "true") {
                        sh """
                          echo "♻️ Updating Product..."
                          apic draft-products:update \
                            --server "${SERVER}" \
                            --org "${ORG}" \
                            ${PRODUCT_NAME}:${API_VERSION} products/${PRODUCT_NAME}/${PRODUCT_NAME}_${API_VERSION}.yaml
                        """
                    } else {
                        sh """
                          echo "🆕 Creating Product..."
                          apic draft-products:create \
                            --server "${SERVER}" \
                            --org "${ORG}" \
                            products/${PRODUCT_NAME}/${PRODUCT_NAME}_${API_VERSION}.yaml
                        """
                    }
                }
            }
        }

        stage('Publish Product') {
            steps {
                sh """
                  echo "🚀 Publishing Product..."
                  apic products:publish \
                    --server "${SERVER}" \
                    --org "${ORG}" \
                    --scope catalog \
                    --catalog ${CATALOG_NAME} \
                    products/${PRODUCT_NAME}/${PRODUCT_NAME}_${API_VERSION}.yaml
                """
            }
        }

        stage('Backup') {
            steps {
                script {
                    def ts = sh(script: "date +%Y%m%d_%H%M%S", returnStdout: true).trim()
                    sh """
                      echo "📦 Backing up with timestamp ${ts}"
                      cp apis/${API_NAME}/${API_NAME}_${API_VERSION}.yaml apis/${API_NAME}/${API_NAME}_${API_VERSION}_${ts}.yaml
                      cp products/${PRODUCT_NAME}/${PRODUCT_NAME}_${API_VERSION}.yaml products/${PRODUCT_NAME}/${PRODUCT_NAME}_${API_VERSION}_${ts}.yaml
                    """
                }
            }
        }
    }

    post {
        success {
            echo "🎉 CI/CD pipeline completed successfully!"
        }
        failure {
            echo "❌ Pipeline failed. Check logs."
        }
    }
}
