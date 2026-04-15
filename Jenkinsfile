pipeline {
    agent none

    environment {
        SONARQUBE_ENV = 'sonarserver'
        PROJECT_KEY   = 'go-project'
        PROJECT_NAME  = 'go-project'
        SCANNER_HOME  = tool 'sonarqube8.0'
    }

    stages {

        stage('Build & Test') {
            agent {
                docker {
                    image 'golang:1.23-bookworm'
                    args '''-u root \
                            -e HOME=/tmp \
                            -e GOCACHE=/tmp/go-cache \
                            -e GOPATH=/tmp/go'''
                }
            }
            steps {
                sh '''
                    git config --global --add safe.directory ${WORKSPACE}
                    go version
                    go mod download
                    go build -v ./...
                    go test ./... -v -coverprofile=coverage.out
                '''
                stash includes: 'coverage.out, **/*.go, go.mod, go.sum', name: 'go-artifacts'
            }
        }

        stage('SonarQube Analysis') {
            agent { label 'built-in' }
            steps {
                unstash 'go-artifacts'
                withSonarQubeEnv("${SONARQUBE_ENV}") {
                    sh """
                        ${SCANNER_HOME}/bin/sonar-scanner \
                          -Dsonar.projectKey=${PROJECT_KEY} \
                          -Dsonar.projectName=${PROJECT_NAME} \
                          -Dsonar.sources=. \
                          -Dsonar.go.coverage.reportPaths=coverage.out
                    """
                }
            }
        }

        stage('Quality Gate') {
            agent { label 'built-in' }
            steps {
                timeout(time: 20, unit: 'MINUTES') {
                    waitForQualityGate abortPipeline: true
                }
            }
        }
    }

    post {
        success {
            echo 'Pipeline sukses'
        }
        failure {
            echo 'Pipeline gagal'
        }
    }
}
