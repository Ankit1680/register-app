pipeline {
    // agent { label 'Jenkins-Agent' }
    agent any

    tools {
        jdk 'java17'
        maven 'maven3'
    }

    parameters {
        string(name: 'REPO_NAME', description: 'Name of the GitHub repository')
        string(name: 'REPO_URL', description: 'GitHub repository URL')
        string(name: 'ORG_NAME', description: 'GitHub organization or user name')
        string(name: 'LANGUAGE', description: 'Primary programming language')
        string(name: 'RELEASE', defaultValue: '1.0.0', description: 'Release version')
    }

    environment {
        BUILD_VERSION = "${params.RELEASE}-${env.BUILD_NUMBER}"
        APP_NAME = "${params.REPO_NAME}"
    }

    stages {

        stage('Cleanup Workspace') {
            steps {
                cleanWs()
            }
        }

        stage('Checkout from SCM') {
            steps {
                git branch: 'main', credentialsId: 'github-cred', url: "${params.REPO_URL}"
            }
        }

        stage('Build Application') {
            when {
                expression {
                    params.LANGUAGE.toLowerCase() == 'java'
                }
            }
            steps {
                sh 'mvn clean package'
            }
        }

        stage('Test Application') {
            when {
                expression {
                    params.LANGUAGE.toLowerCase() == 'java'
                }
            }
            steps {
                sh 'mvn test'
            }
        }

        stage('Post-Build Info') {
            steps {
                echo "Build version: ${BUILD_VERSION}"
                echo "Repo: ${params.REPO_URL}"
                echo "Org: ${params.ORG_NAME}"
                echo "Tech Stack: ${params.LANGUAGE}"
            }
        }
    }

    post {
        failure {
            echo "Build failed for ${APP_NAME}"
        }
        success {
            echo "Build succeeded for ${APP_NAME}"
        }
    }
}
