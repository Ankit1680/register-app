pipeline {
    agent { label 'Jenkins-Agent' }

    tools {
        jdk 'java17'
        maven 'maven3'
    }

    parameters {
        string(name: 'APP_NAME', description: 'Name of the application')
        string(name: 'REPO_LINK', description: 'GitHub repository URL')
        string(name: 'RELEASE', defaultValue: '1.0.0', description: 'Release version')
        string(name: 'ORG_NAME', description: 'GitHub organization name')
        string(name: 'TECH_STACK', description: 'Primary tech stack/language')
    }

    environment {
        BUILD_VERSION = "${params.RELEASE}-${env.BUILD_NUMBER}"
    }

    stages {

        stage('Cleanup Workspace') {
            steps {
                cleanWs()
            }
        }

        stage('Checkout from SCM') {
            steps {
                git branch: 'main', credentialsId: 'github-cred', url: "${params.REPO_LINK}"
            }
        }

        stage('Build Application') {
            steps {
                sh 'mvn clean package'
            }
        }

        stage('Test Application') {
            steps {
                sh 'mvn test'
            }
        }

        stage('Post-Build Info') {
            steps {
                echo "✅ Build version: ${BUILD_VERSION}"
                echo "📦 Repo: ${params.REPO_LINK}"
                echo "👤 Org: ${params.ORG_NAME}"
                echo "📌 Tech Stack: ${params.TECH_STACK}"
            }
        }
    }

    post {
        failure {
            echo "❌ Build failed for ${params.APP_NAME}"
        }
        success {
            echo "✅ Build succeeded for ${params.APP_NAME}"
        }
    }
}
