pipeline {
    agent any

    // Provision Node.js (and a global yarn) via the NodeJS plugin.
    // 'Node-26.4.0' is configured in Manage Jenkins -> Tools.
    tools {
        nodejs 'Node-26.4.0'
    }

    options {
        timestamps()
        disableConcurrentBuilds()
    }

    stages {
        stage('Backend: Build & Test') {
            steps {
                dir('backend') {
                    sh './gradlew --no-daemon clean build'
                }
            }
            post {
                always {
                    junit testResults: 'backend/build/test-results/test/*.xml',
                          allowEmptyResults: true
                }
            }
        }

        stage('Frontend: Install, Test & Build') {
            steps {
                dir('frontend') {
                    sh 'yarn install --immutable'
                    sh 'yarn test:ci'
                    sh 'yarn build'
                }
            }
            post {
                always {
                    junit testResults: 'frontend/test-results/junit.xml',
                          allowEmptyResults: true
                }
            }
        }
    }

    post {
        success {
            archiveArtifacts artifacts: 'backend/build/libs/*.jar, frontend/dist/**',
                             fingerprint: true
        }
    }
}
