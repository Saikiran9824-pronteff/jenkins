pipeline {
    agent any

    environment {
        API_NAME     = "loan-api"
        API_VERSION  = "1.0.0"
        PRODUCT_NAME = "loan-product"
        CATALOG_NAME = "prod"

        SERVER   = "https://small-1-mgmt-api-manager-cp4i.apps.ocp.prontefflabs.com"
        ORG      = "indusapi-np"
        USERNAME = "umesh"
        PASSWORD = "!n0r1t5@C"
        REALM    = "provider/default-idp-2"
    }

    stages {

        stage('Login') {
            steps {
                sh """
                  echo "🔐 Logging in to API Connect..."
                  apic login --server "$SERVER" --username "$USERNAME" --password "$PASSWORD" --realm "$REALM"
                """
            }
        }

        stage('Validate API') {
            steps {
                sh """
                  echo "✅ Validating API YAML for ${API_NAME}_${API_VERSION}"
                  apic validate "apis/${API_NAME}/${API_NAME}_${API_VERSION}.yaml"
                """
            }
        }

        stage('API Check/Create/Update') {
            steps {
                script {
                    def apiExists = sh(
                        script: """apic draft-apis:get "${API_NAME}:${API_VERSION}" --server "$SERVER" --org "$ORG" || true""",
                        returnStdout: true
                    ).trim()

                    if (apiExists.contains("${API_NAME}:${API_VERSION}")) {
                        sh """
                          echo "♻️ Updating existing draft API ${API_NAME}:${API_VERSION}"
                          apic draft-apis:update "${API_NAME}:${API_VERSION}" "apis/${API_NAME}/${API_NAME}_${API_VERSION}.yaml" \
                            --server "$SERVER" --org "$ORG"
                        """
                    } else {
                        sh """
                          echo "🆕 Creating new draft API ${API_NAME}:${API_VERSION}"
                          apic draft-apis:create "apis/${API_NAME}/${API_NAME}_${API_VERSION}.yaml" \
                            --server "$SERVER" --org "$ORG"
                        """
                    }
                }
            }
        }

        stage('Fix Product Refs') {
            steps {
                sh """
                  echo "🔧 Fixing API references in product YAML..."
                  bash /var/lib/jenkins/scripts/fix-refs.sh "products/${PRODUCT_NAME}" "apis"
                """
            }
        }

        stage('Product Check/Create/Update') {
            steps {
                script {
                    def productExists = sh(
                        script: """apic draft-products:get "${PRODUCT_NAME}:${API_VERSION}" --server "$SERVER" --org "$ORG" || true""",
                        returnStdout: true
                    ).trim()

                    if (productExists.contains("${PRODUCT_NAME}:${API_VERSION}")) {
                        sh """
                          echo "♻️ Updating existing draft Product ${PRODUCT_NAME}:${API_VERSION}"
                          apic draft-products:update "${PRODUCT_NAME}:${API_VERSION}" "products/${PRODUCT_NAME}/${PRODUCT_NAME}_${API_VERSION}.yaml" \
                            --server "$SERVER" --org "$ORG"
                        """
                    } else {
                        sh """
                          echo "🆕 Creating new draft Product ${PRODUCT_NAME}:${API_VERSION}"
                          apic draft-products:create "products/${PRODUCT_NAME}/${PRODUCT_NAME}_${API_VERSION}.yaml" \
                            --server "$SERVER" --org "$ORG"
                        """
                    }
                }
            }
        }

        stage('Publish Product') {
            steps {
                sh """
                  echo "🚀 Publishing Product ${PRODUCT_NAME}_${API_VERSION}.yaml to catalog ${CATALOG_NAME}"
                  apic products:publish "products/${PRODUCT_NAME}/${PRODUCT_NAME}_${API_VERSION}.yaml" \
                    --scope catalog --catalog "$CATALOG_NAME" --server "$SERVER" --org "$ORG"
                """
            }
        }

        stage('Backup') {
            steps {
                script {
                    def timestamp = new Date().format("yyyyMMdd_HHmmss")
                    sh """
                      echo "📦 Backing up API & Product YAMLs with timestamp $timestamp"

                      cp "apis/${API_NAME}/${API_NAME}_${API_VERSION}.yaml" \
                         "apis/${API_NAME}/${API_NAME}_${API_VERSION}_${timestamp}.yaml"

                      cp "products/${PRODUCT_NAME}/${PRODUCT_NAME}_${API_VERSION}.yaml" \
                         "products/${PRODUCT_NAME}/${PRODUCT_NAME}_${API_VERSION}_${timestamp}.yaml"

                      echo "✅ Backup created: $timestamp"
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
            echo "❌ Pipeline failed!"
        }
    }
}
