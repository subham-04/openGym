pipeline {
    agent any

    environment {
        // Set in Jenkins → Manage Jenkins → Credentials:
        //   FLY_API_TOKEN  — your Fly.io deploy token (flyctl tokens create deploy)
        FLY_API_TOKEN = credentials('FLY_API_TOKEN')

        // App names on Fly.io — change if you used different names
        FLY_APP_API = 'opengym-api'
        FLY_APP_WEB = 'opengym-web'

        // Node version to match the Dockerfiles
        NODE_VERSION = '22'
    }

    options {
        // Keep last 10 builds, discard older ones
        buildDiscarder(logRotator(numToKeepStr: '10'))
        // Fail if the whole pipeline takes more than 30 minutes
        timeout(time: 30, unit: 'MINUTES')
        // Don't run two builds of the same branch at once
        disableConcurrentBuilds()
    }

    stages {

        stage('Checkout') {
            steps {
                checkout scm
                echo "Building branch: ${env.BRANCH_NAME} @ ${env.GIT_COMMIT?.take(7)}"
            }
        }

        stage('Install dependencies') {
            parallel {
                stage('API deps') {
                    steps {
                        dir('api') {
                            bat 'npm ci --omit=dev --omit=optional'
                        }
                    }
                }
                stage('Frontend deps') {
                    steps {
                        dir('frontend') {
                            bat 'npm ci --ignore-scripts'
                        }
                    }
                }
                stage('MCP deps') {
                    steps {
                        dir('mcp') {
                            bat 'npm ci'
                        }
                    }
                }
            }
        }

        stage('Run tests') {
            parallel {
                stage('API tests') {
                    steps {
                        dir('api') {
                            bat 'node --test test/*.test.js'
                        }
                    }
                }
                stage('Frontend tests') {
                    steps {
                        dir('frontend') {
                            bat 'npx vitest run'
                        }
                    }
                }
                stage('MCP tests') {
                    steps {
                        dir('mcp') {
                            bat 'npx vitest run'
                        }
                    }
                }
            }
        }

        stage('Build Docker images') {
            when {
                // Only build images on main branch
                branch 'main'
            }
            parallel {
                stage('Build API image') {
                    steps {
                        bat 'docker build -t opengym-api:latest --target default ./api'
                    }
                }
                stage('Build Web image') {
                    steps {
                        bat 'docker build -t opengym-web:latest -f web/Dockerfile .'
                    }
                }
            }
        }

        stage('Deploy to Fly.io') {
            when {
                branch 'main'
            }
            steps {
                echo 'Deploying API to Fly.io...'
                bat """
                    set FLY_API_TOKEN=%FLY_API_TOKEN%
                    flyctl deploy --app %FLY_APP_API% --dockerfile api/Dockerfile --remote-only --yes
                """
                echo 'Deploying Web to Fly.io...'
                bat """
                    set FLY_API_TOKEN=%FLY_API_TOKEN%
                    flyctl deploy --app %FLY_APP_WEB% --dockerfile web/Dockerfile --remote-only --yes
                """
            }
        }
    }

    post {
        success {
            echo "Pipeline succeeded for ${env.BRANCH_NAME} @ ${env.GIT_COMMIT?.take(7)}"
        }
        failure {
            echo "Pipeline FAILED for ${env.BRANCH_NAME} @ ${env.GIT_COMMIT?.take(7)}"
        }
        always {
            // Clean workspace to avoid disk buildup
            cleanWs()
        }
    }
}
