pipeline{
    agent any

    tools{
        jdk 'jdk17'
        maven 'maven3'
    }

    environment{
        SCANNER_HOME = tool 'sonar-scanner'
    }

    stages{
        stage('Git Checkout'){
            steps{
                git branch: 'main', url: 'https://github.com/devops-maestro17/Board-Game-App.git' 
            }
        }

        stage('Compile source code'){
            steps{
                sh "mvn compile"
            }
        }

        stage('Run Tests'){
            steps{
                sh "mvn test"
            }
        }

        stage('Trivy File system scan'){
            steps{
                sh "trivy fs --format table -o trivy-fs-report.html ."
            }
        }

        stage('Sonarqube analysis'){
            steps{
                withSonarQubeEnv('sonar-server'){
                    sh '''
                    $SCANNER_HOME/bin/sonar-scanner -Dsonar.projectName=BoardGame -Dsonar.projectKey=BoardGame \
                    -Dsonar.java.binaries=.
                    '''
                }
            }
        }
        
        stage("Quality gate"){
           steps {
                script {
                    waitForQualityGate abortPipeline: false, credentialsId: 'sonar-cred' 
                }
            } 
        }

        stage('Build the code'){
            steps{
                sh "mvn package"
            }
        }

        stage("Publish artifact to Nexus"){
            steps{
                withMaven(globalMavenSettingsConfig: 'maven-settings', jdk: 'jdk17', maven: 'maven3', mavenSettingsConfig: '', traceability: true) {
                    sh "mvn deploy"
                }
            }
        }

        stage("Build and Tag Docker Image"){
            steps{
                script{
                    withDockerRegistry(credentialsId: 'docker-cred', toolName: 'docker') {
                        sh "docker build -t containerizeops/boardgame:1.0 ."
                    }
                }
            }
        }

        stage('Docker Image Scan') {
            steps {
                sh "trivy image --format table -o trivy-image-report.html containerizeops/boardgame:1.0 "
            }
        }

        stage('Push Docker Image') {
            steps {
                script {
                    withDockerRegistry(credentialsId: 'docker-cred', toolName: 'docker') {
                        sh "docker push containerizeops/boardgame:1.0"
                    }
                }
            }
        }

        stage("Update Image Tag in Manifests"){
            environment {
                GIT_REPO_NAME = "Board-Game-App"
                GIT_USER_NAME = "devops-maestro17"
            }
            steps {
                withCredentials([string(credentialsId: 'github', variable: 'GITHUB_TOKEN)]) {
                    sh '''
                        git config user.email "rajdeep_deogharia@outlook.com"
                        git config user.name "devops-maestro17"
                        BUILD_NUMBER=${BUILD_NUMBER}
                        sed -i "s/replaceImageTag/${BUILD_NUMBER}/g" Board-Game-App/deployment-service.yaml
                        git add Board-Game-App/deployment-service.yaml
                        git commit -m "Update deployment image to version ${BUILD_NUMBER}"
                        git push https://${GITHUB_TOKEN}@github.com/${GIT_USER_NAME}/${GIT_REPO_NAME} HEAD:main
                    '''
                }
            }
        }

    }
}