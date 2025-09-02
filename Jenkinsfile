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
                  echo "🔐 Logging in..."
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

        stage('Check API Exists') {
            steps {
                sh """
                  echo "🔎 Checking API exists with list-all..."
                  apic draft-apis:list-all \
                    --server "${SERVER}" \
                    --org "${ORG}" | grep -w "${API_NAME}:${API_VERSION}" || true
                """
            }
        }

        stage('Create or Update API') {
            steps {
                script {
                    def exists = sh(
                        script: """
                          apic draft-apis:list-all \
                            --server "${SERVER}" \
                            --org "${ORG}" | grep -w "${API_NAME}:${API_VERSION}" >/dev/null && echo true || echo false
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

        stage('Check Product Exists') {
            steps {
                sh """
                  echo "🔎 Checking Product exists with list-all..."
                  apic draft-products:list-all \
                    --server "${SERVER}" \
                    --org "${ORG}" | grep -w "${PRODUCT_NAME}:${API_VERSION}" || true
                """
            }
        }

        stage('Create or Update Product') {
            steps {
                script {
                    def prodExists = sh(
                        script: """
                          apic draft-products:list-all \
                            --server "${SERVER}" \
                            --org "${ORG}" | grep -w "${PRODUCT_NAME}:${API_VERSION}" >/dev/null && echo true || echo false
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
