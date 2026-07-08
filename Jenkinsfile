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
                    // Single sh block: the PATH export must survive until the yarn calls.
                    // Any yarn launcher delegates to the Yarn 4.17.0 release
                    // committed in .yarn/releases (via yarnPath in .yarnrc.yml).
                    sh '''
                        # Provision Node.js into the workspace when the agent has none
                        # (e.g. the stock jenkins/jenkins Docker image). Cached in
                        # .node/ across builds.
                        NODE_VERSION=24.18.0
                        OS=$(uname | tr '[:upper:]' '[:lower:]')
                        case "$(uname -m)" in
                            aarch64|arm64) ARCH=arm64 ;;
                            *)             ARCH=x64   ;;
                        esac
                        DIST="node-v${NODE_VERSION}-${OS}-${ARCH}"
                        if [ -d ".node/${DIST}/bin" ]; then
                            export PATH="$PWD/.node/${DIST}/bin:$PATH"
                        elif ! command -v npm >/dev/null 2>&1; then
                            echo "Node.js not found on agent - downloading ${DIST}"
                            mkdir -p .node
                            curl -fsSL "https://nodejs.org/dist/v${NODE_VERSION}/${DIST}.tar.gz" | tar -xz -C .node
                            export PATH="$PWD/.node/${DIST}/bin:$PATH"
                        fi
                        node --version
                        # Yarn launcher: global install if possible, npx as fallback
                        YARN=yarn
                        if ! command -v yarn >/dev/null 2>&1; then
                            npm install -g yarn >/dev/null 2>&1 || true
                            command -v yarn >/dev/null 2>&1 || YARN="npx --yes yarn"
                        fi
                        $YARN install --immutable
                        $YARN test:ci
                        $YARN build
                    '''
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
