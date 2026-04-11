pipeline {
    agent any
    
    options {
        ansiColor('xterm')
    }

    stages {
        stage('build') {
            agent {
                docker {
                    image 'node:22-alpine'
                }
            }
            steps {
                sh 'npm ci'
                sh 'npm run build'
                stash name: 'workspace', includes: '**/*'
            }
        }

        stage('test') {
            parallel {
                stage('unit tests') {
                    agent {
                        docker {
                            image 'node:22-alpine'
                        }
                    }
                    steps {
                        sh 'npm ci'
                        sh 'npx vitest run --reporter=verbose'
                    }
                }

                stage('integration tests') {
                    agent {
                        docker {
                            image 'mcr.microsoft.com/playwright:v1.54.2-jammy'
                            reuseNode true
                        }
                    }
                    steps {
                        unstash 'workspace'
                        sh 'npm ci'
                        sh 'npx playwright test --reporter=html'
                    }
                }
            }
        }

        stage('deploy') {
            agent {
                docker {
                    image 'alpine'
                }
            }
            steps {
                echo 'Mock deployment was successful!'
            }
        }

        stage('e2e') {
            agent {
                docker {
                    image 'mcr.microsoft.com/playwright:v1.54.2-jammy'
                }
            }
            environment {
                E2E_BASE_URL = 'http://spanish-cards.netlify.app/'
            }
            steps {
                unstash 'workspace'
                sh 'npm ci'
                sh 'npx playwright install --with-deps'
                sh 'npm run test:e2e'
            }
        }
    }

    post {
        always {
            archiveArtifacts artifacts: 'playwright-report/**', allowEmptyArchive: true
        }
    }
}