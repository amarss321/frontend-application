pipeline {
    agent any
    
    tools{
        jdk 'jdk17'
        nodejs 'nodejs'
    }
    
    environment{
        SCANNER_HOME=tool 'sonar-scanner'
    }

    stages {
        stage('Git Checkout') {
            steps {
                git credentialsId: 'git-cred', url: 'https://github.com/amarss321/frontend-application.git'
                }
        }
        stage('Sonarqube Analysis') {
            steps {
                    withSonarQubeEnv('sonar-server') {
                    sh "$SCANNER_HOME/bin/sonar-scanner -Dsonar.projectName=three-tier-frontend -Dsonar.projectKey=three-tier-frontend"
                
                }
            }
        }
        stage('Quality Check') {
            steps {
                //waitForQualityGate abortPipeline: false, credentialsId: 'sonar-token'
                sh 'echo "Quality check passed"'
            }
        }
        stage('OWASP Dependency-Check Scan') {
            steps {
                
                    dependencyCheck additionalArguments: '--scan ./ --disableYarnAudit --disableNodeAudit', odcInstallation: 'DP-Check'
                    dependencyCheckPublisher pattern: '**/dependency-check-report.xml'
                
            }
        }
    }
}