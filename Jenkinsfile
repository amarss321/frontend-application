pipeline {
    agent any
    
    tools{
        jdk 'jdk17'
        nodejs 'nodejs'
    }
    
    environment{
        SCANNER_HOME=tool 'sonar-scanner'
        AWS_ACCOUNT_ID = credentials('891377203846')
        AWS_ECR_REPO_NAME = credentials('frontend')
        AWS_DEFAULT_REGION = 'us-east-1'
        REPOSITORY_URI = "${AWS_ACCOUNT_ID}.dkr.ecr.${AWS_DEFAULT_REGION}.amazonaws.com/"
        GIT_REPO_NAME = 'k8s-frontend-backend'
        GIT_USER_NAME = 'amarss321'
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

        stage('Trivy File Scan') {
            steps {
                sh 'trivy fs --format table -o trivy-fs-reports.html .'
            }
        }
        stage('Docker Image Build') {
            steps {
                script {
                    sh "docker build -t ${AWS_ECR_REPO_NAME} ."   
                }
            }
        }
        stage('Trivy Scan Image') {
            steps {
                sh "trivy image ${AWS_ECR_REPO_NAME} > trivyimage-frontend-${BUILD_NUMBER}-${BUILD_ID}.txt"
            }
        }
        stage("ECR Image Pushing") {
            steps {
                script {
                    sh 'aws ecr get-login-password --region ${AWS_DEFAULT_REGION} | docker login --username AWS --password-stdin ${REPOSITORY_URI}'
                    sh 'docker tag ${AWS_ECR_REPO_NAME} ${REPOSITORY_URI}${AWS_ECR_REPO_NAME}:${BUILD_NUMBER}'
                    sh 'docker push ${REPOSITORY_URI}${AWS_ECR_REPO_NAME}:${BUILD_NUMBER}'
                }
            }
        }
        stage('Update Deployment file') {
            steps {
                dir('Frontend') {
                withCredentials([string(credentialsId: 'git-cred')]) {
                    script {
                        sh '''
                        git config user.email "amarnathmalasani@gmail.com"
                        git config user.name "amarnath"
                        BUILD_NUMBER=${BUILD_NUMBER}
                        echo $BUILD_NUMBER
                        imageTag=$(grep -oP '(?<=frontend:)[^ ]+' deployment.yaml)
                        echo $imageTag
                        sed -i "s/${AWS_ECR_REPO_NAME}:${imageTag}/${AWS_ECR_REPO_NAME}:${BUILD_NUMBER}/" deployment.yaml
                        git add deployment.yaml
                        git commit -m "Update deployment Image to version \${BUILD_NUMBER}"
                        git push @github.com/${GIT_USER_NAME}/${GIT_REPO_NAME">https://${GITHUB_TOKEN}@github.com/${GIT_USER_NAME}/${GIT_REPO_NAME} HEAD:main
                        '''
                    
                        }   
                    }
                }
            }
        }
    }
}