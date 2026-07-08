pipeline {
    agent any

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
                    // Any yarn launcher delegates to the Yarn 4.17.0 release
                    // committed in .yarn/releases (via yarnPath in .yarnrc.yml)
                    sh 'command -v yarn >/dev/null 2>&1 || npm install -g yarn'
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
