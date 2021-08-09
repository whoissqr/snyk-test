pipeline {
    agent any

    environment {
      PATH = "/snyk:$PATH"
      snyk_api_token = credentials('snyk-api-secret')
    }

    stages {
        stage('Checkout') {
            steps {
                git branch: 'main', changelog: false, poll: false, url: 'https://github.com/whoissqr/snyk-test'
            }
        }
        
        stage('Auth') {
            steps {
                sh script: "snyk-alpine config set api=$snyk_api_token"
                sh script: "snyk-alpine config set org=snyk-jenkins", label: "set SNYK ORG"
            }
        }

        stage('SNYK SCA') {
            steps {             
                sh script: "npm install --no-audit --no-fund", label: "install dep"
                snykSecurity additionalArguments: '''--all-projects --remote-repo-url=https://github.com/whoissqr/snyk-test''', \
                    failOnIssues: false, \
                    snykInstallation: 'snyk-cli-jenkins', \
                    snykTokenId: 'snyk-api-token'
            }
        }
        stage('SNYK IaC') {
            steps {
                catchError(buildResult: 'SUCCESS', stageResult: 'SUCCESS') {
                    sh script: "snyk-alpine iac test ec2.yml --org=snyk-jenkins", label: "scan EC2 config"
                }
            }
        }
        stage('SNYK Container') {
            steps {
            sh script: "docker build -t hello-tomcat .", label: "Docker build"
            sh script: "snyk-alpine container monitor hello-tomcat --file=Dockerfile --project-name=snyk-container", label: "scan image"
            sh script: "snyk-alpine container monitor docker-archive:ngrok/ngrok.tar --project-name=snyk-container-tar", label: "scan image"
            }
        }
        
        stage('SNYK docker tar') {
            steps {
            sh script: "snyk-alpine container monitor docker-archive:ngrok/ngrok.tar --project-name=snyk-container-tar", label: "scan img tar"
            }
        }
    }
}
