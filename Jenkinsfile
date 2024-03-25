pipeline {
  agent any
  tools {
        jdk 'jdk17'
        nodejs 'node16'
    }
    environment {
        SCANNER_HOME = tool 'sonar-scanner'
    }

  stages {
        stage('clean workspace'){
            steps{
                cleanWs()
            }
        }
        stage('Checkout from Git') {
            steps {
                git branch: 'main', url: 'https://github.com/Veeresh2708/myspringbootproject.git'
            }
        }
            stage('Build Artifact') {
                steps {
                sh "mvn clean package -DskipTests=true"
                archive 'target/*.jar' //so that they can be downloaded later
                }
        }
        stage('Unit Tests') {
            steps {
              sh "mvn test jacoco:report" 
            }
            post {
                always {
                junit 'target/surefire-reports/*.xml'
                jacoco execPattern: 'target/jacoco.exec'
                }
            }
        }
        stage('Mutation Tests - PIT') {
           steps {
              sh "mvn org.pitest:pitest-maven:mutationCoverage"
            }
            post {
                always {
                    pitmutation mutationStatsFile: '**/target/pit-reports/**/mutations.xml'
                }  
            }
        }
        stage('Trivy FS Scanning') {
            steps {
              sh 'trivy fs . > trivyfs.txt'
            }
        }
        stage('Vulnerability Scan - Docker ') {
            steps {
                sh "mvn dependency-check:check"
            }
            post {
                always {
                    dependencyCheckPublisher pattern: 'target/dependency-check-report.xml'
                }
            }
        }
        stage('Docker Build and Push') {
            steps {
                script{
                   withDockerRegistry(credentialsId: 'docker-hub', toolName: 'docker'){
                   //withCredentials([usernamePassword(credentialsId: 'docker-hub', passwordVariable: 'dockerHubPassword', usernameVariable: 'dockerHubUser')]) {
                   sh 'printenv'
                   sh 'docker build -t springboot_app:""$GIT_COMMIT"" .'
                   sh 'docker tag springboot_app:""$GIT_COMMIT"" veereshvanga/macho1:$BUILD_NUMBER'
                   sh 'docker push veereshvanga/macho1:""$BUILD_NUMBER""'
                   }
               }
            }
        }
        stage('Trivy Scanning') {
            steps {
              sh 'trivy image veereshvanga/macho1:""$BUILD_NUMBER"" > trivyimage.txt'
            }
        }
    }
}
