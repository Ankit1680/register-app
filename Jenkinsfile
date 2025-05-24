pipeline {
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
        string(name: 'GIT_TOKEN_ID', description: 'Jenkins credentials ID for GitHub access')
    }

    environment {
        BUILD_VERSION = "${params.RELEASE}-${env.BUILD_NUMBER}"
        APP_NAME = "${params.REPO_NAME}"
    }

    stages {
        stage('Cleanup') {
            steps {
                cleanWs()
            }
        }

        // stage('Checkout from SCM') {
        //     steps {
        //         git branch: 'main', credentialsId: 'github-cred', url: "${params.REPO_URL}"
        //     }
        // }

        stage('Checkout from SCM') {
            steps {
                withCredentials([string(credentialsId: "${params.GIT_TOKEN_ID}", variable: 'GIT_TOKEN')]) {
                    script {
                        def repoWithToken = params.REPO_URL.replace('https://', "https://${GIT_TOKEN}@")
                        git branch: 'master', url: repoWithToken
                    }
                }
            }
        }



        stage('Determine Final Tech Stack') {
            steps {
                script {
                    def detectedStack = ''
                    if (fileExists('pom.xml')) {
                        detectedStack = 'java'
                    } else if (fileExists('package-lock.json') || fileExists('package.json')) {
                        detectedStack = 'nodejs'
                    } else if (fileExists('app.py') || fileExists('requirements.txt')) {
                        detectedStack = 'python'
                    } else {
                        error "Unable to detect a valid tech stack from repo files"
                    }

                    env.FINAL_TECH_STACK = detectedStack
                    echo "Confirmed Tech Stack: ${env.FINAL_TECH_STACK}"
                }
            }
        }

        stage('Trigger Final Pipeline') {
            steps {
                script {
                    def finalJobName = ''
                    if (env.FINAL_TECH_STACK == 'java') {
                        finalJobName = 'springboot-pipeline'
                    } else if (env.FINAL_TECH_STACK == 'nodejs') {
                        finalJobName = 'node-pipeline'
                    } else if (env.FINAL_TECH_STACK == 'python') {
                        finalJobName = 'python-pipeline'
                    } else {
                        error "No matching pipeline for detected tech stack: ${env.FINAL_TECH_STACK}"
                    }

                    echo "Triggering job: ${finalJobName}"

                    build job: finalJobName, wait: false, parameters: [
                        string(name: 'REPO_NAME', value: params.REPO_NAME),
                        string(name: 'REPO_URL', value: params.REPO_URL),
                        string(name: 'ORG_NAME', value: params.ORG_NAME),
                        string(name: 'LANGUAGE', value: env.FINAL_TECH_STACK),
                        string(name: 'RELEASE', value: params.RELEASE),
                        string(name: 'GIT_TOKEN_ID', value: params.GIT_TOKEN_ID)
                    ]
                }
            }
        }
    }

    post {
        failure {
            echo "Dispatcher failed for ${APP_NAME}"
        }
        success {
            echo "Dispatcher succeeded for ${APP_NAME}"
        }
    }
}
