#!groovy
/*
 * 20241212 Vitor Neves
 * CI/CD on NIF Validator
 */

/* groovylint-disable-next-line CompileStatic */
pipeline {
    agent any
    environment {
        HOME = "${env.WORKSPACE}"
    }
    stages {
        stage('Create docker environment') {

            agent {
                docker {
                    image 'python:3.11-slim'
                    reuseNode true
                }
            }
            steps {
                sh"""
                export PIP_DISABLE_PIP_VERSION_CHECK=1
                export PYTHONDONTWRITEBYTECODE=1
                export PYTHONUNBUFFERED=1
                export PATH="$HOME/.local/bin:${PATH}"
                # Install python libraries
                pip install --user -r requirements.txt
                pip install --user -r requirements-test.txt
                """
            }
        }

        stage('Unit Tests') {
            agent {
                docker {
                    image 'python:3.11-slim'
                    reuseNode true
                }
            }
            steps {
                sh 'python3 -m pytest --junitxml results.xml tests/'
            }
            post {
                always {
                    archiveArtifacts artifacts: 'results.xml', fingerprint: true
                    junit 'results.xml'
                }
            }
        }

        stage('Coverage report') {
            agent {
                docker {
                    image 'python:3.11-slim'
                    reuseNode true
                }
            }
            steps {
                sh'''
                python3 -m coverage run --source=. --omit=tests/* -m pytest tests
                python3 -m coverage report -m
                python3 -m coverage html
                '''
            }
            post {
                always {
                    publishHTML(target:[
                        allowMissing: false,
                        alwaysLinkToLastBuild: false,
                        keepAll: true,
                        reportDir: 'htmlcov',
                        reportFiles: 'index.html',
                        reportName: 'Coverage Report'
                    ])
                }
            }
        }
        stage('Quality Analysis') {
            parallel {
                stage('Integration Tests') {
                    agent any
                    steps {
                        echo 'Integration tests'
                    }
                }
                stage('User Interface tests') {
                    agent any
                    steps {
                        echo 'UI tests'
                    }
                }
                stage('PEP8 Verification') {
                    agent {
                        docker {
                            image 'python:3.11-slim'
                            reuseNode true
                        }
                    }
                    steps {
                        sh 'python3 -m flake8 . --exclude site-packages --exit-zero'
                    }
                }
                stage('Cyclomatic complexity analysis') {
                    agent {
                        docker {
                            image 'python:3.11-slim'
                            reuseNode true
                        }
                    }
                    steps {
                         sh 'python3 -m radon cc . -a -s --exclude site-packages'
                    }
                }
            }

        }

        stage('Deploy') {
            steps {
                withCredentials([usernamePassword(credentialsId: 'docker-hub',
                    passwordVariable:'passwd', usernameVariable:'username')]) {
                    sh"""
                    docker build -t ${username}/nif-validator .
                    docker login -u ${username} -p ${passwd}
                    docker push ${username}/nif-validator
                    """
                    } 
            }
        }
    }

    post {
        success {
            echo 'Pipeline completed successfully!'
        }
        failure {
            mail to: 'vitor@milv.casa',
            subject: "Failed pipeline: ${currentBuild.fullDisplayName}"
            body: 'You realy mess things up'
        }
    }
}