pipeline {
    agent any

    environment {
        API_NAME      = "loan-api"
        API_VERSION   = "1.0.0"
        PRODUCT_NAME  = "loan-product"
        CATALOG_NAME  = "prod"
        SERVER        = "https://small-1-mgmt-api-manager-cp4i.apps.ocp.prontefflabs.com"
        ORG           = "indusapi-np"
        USERNAME      = "umesh"
        PASSWORD      = "!n0r1t5@C"
        REALM         = "provider/default-idp-2"
    }

    stages {

        stage('Login') {
            steps {
                echo "🔐 Logging in to API Connect..."
                sh """
                apic login --server ${SERVER} \
                  --username ${USERNAME} \
                  --password '${PASSWORD}' \
                  --realm ${REALM}
                """
            }
        }

        stage('Validate API') {
            steps {
                echo "✅ Validating API YAML"
                sh """
                apis/${API_NAME}/${API_NAME}_${API_VERSION}.yaml
                apic validate apis/${API_NAME}/${API_NAME}_${API_VERSION}.yaml
                """
            }
        }

        stage('Check API') {
            steps {
                echo "🔎 Checking if API exists..."
                script {
                    def apiExists = sh(
                        script: """apic draft-apis:get --server ${SERVER} --org ${ORG} ${API_NAME}:${API_VERSION} || true""",
                        returnStdout: true
                    ).trim()

                    if (apiExists.contains("${API_NAME}:${API_VERSION}")) {
                        echo "♻️ API exists. Updating..."
                        sh """
                        apic draft-apis:update ${API_NAME}:${API_VERSION} apis/${API_NAME}/${API_NAME}_${API_VERSION}.yaml \
                          --server ${SERVER} --org ${ORG}
                        """
                    } else {
                        echo "🆕 API not found. Creating..."
                        sh """
                        apic draft-apis:create apis/${API_NAME}/${API_NAME}_${API_VERSION}.yaml \
                          --server ${SERVER} --org ${ORG}
                        """
                    }
                }
            }
        }

        stage('Fix Product API Refs') {
            steps {
                echo "🔧 Fixing API references in product YAML..."
                sh """
                /var/lib/jenkins/scripts/fix-refs.sh products/${PRODUCT_NAME} apis
                """
            }
        }

        stage('Check Product') {
            steps {
                echo "🔎 Checking if Product exists..."
                script {
                    def productExists = sh(
                        script: """apic draft-products:get --server ${SERVER} --org ${ORG} ${PRODUCT_NAME}:${API_VERSION} || true""",
                        returnStdout: true
                    ).trim()

                    if (productExists.contains("${PRODUCT_NAME}:${API_VERSION}")) {
                        echo "♻️ Product exists. Updating..."
                        sh """
                        apic draft-products:update ${PRODUCT_NAME}:${API_VERSION} products/${PRODUCT_NAME}/${PRODUCT_NAME}_${API_VERSION}.yaml \
                          --server ${SERVER} --org ${ORG}
                        """
                    } else {
                        echo "🆕 Product not found. Creating..."
                        sh """
                        apic draft-products:create products/${PRODUCT_NAME}/${PRODUCT_NAME}_${API_VERSION}.yaml \
                          --server ${SERVER} --org ${ORG}
                        """
                    }
                }
            }
        }

        stage('Publish Product') {
            steps {
                echo "🚀 Publishing Product ${PRODUCT_NAME}_${API_VERSION}.yaml to catalog ${CATALOG_NAME}"
                sh """
                apic products:publish products/${PRODUCT_NAME}/${PRODUCT_NAME}_${API_VERSION}.yaml \
                  --scope catalog \
                  --catalog ${CATALOG_NAME} \
                  --server ${SERVER} \
                  --org ${ORG}
                """
            }
        }

        stage('Backup') {
            steps {
                script {
                    def timestamp = sh(
                        script: "date +%Y%m%d_%H%M%S",
                        returnStdout: true
                    ).trim()

                    echo "📦 Creating backup with timestamp ${timestamp}"

                    sh """
                    cp apis/${API_NAME}/${API_NAME}_${API_VERSION}.yaml apis/${API_NAME}/${API_NAME}_${API_VERSION}_${timestamp}.yaml
                    cp products/${PRODUCT_NAME}/${PRODUCT_NAME}_${API_VERSION}.yaml products/${PRODUCT_NAME}/${PRODUCT_NAME}_${API_VERSION}_${timestamp}.yaml
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
