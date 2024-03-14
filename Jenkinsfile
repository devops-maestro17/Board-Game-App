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

    }
}