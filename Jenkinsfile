pipeline{
    agent any 
        tools{
            maven 'maven1'
            jdk 'jdk11'
    }
    environment{
        REGISTRY_CREDENTIALS = credentials('docker-cred')
        DOCKER_IMAGE = "murthyvella/myimage:${BUILD_NUMBER}"
        //defining update image 
         GIT_REPO_NAME = "Production-Grade-Cloud-Native-CICD" 
         GIT_USER_NAME = "murthyvella"
    }
    stages{
        stage('code'){
            steps{
                checkout scmGit(branches: [[name: '*/main']], extensions: [], userRemoteConfigs: [[url: 'https://github.com/murthyvella/Production-Grade-Cloud-Native-CICD']])
            }
        }
        stage('build&test'){
            steps{
                sh ''' java -version 
                      mvn clean package
                    '''
            }
        }
        stage('security analysis'){
            environment {
                SONAR_HOME = 'http://18.205.239.211:8000/'
            }
            steps{
                withCredentials([string(credentialsId: 'sonarqube_scanner', variable: 'sonarqube_scanner')]) {
                sh ''' 
                mvn clean package org.sonarsource.scanner.maven:sonar-maven-plugin:sonar \
               -Dsonar.projectKey=ultimate-ci-cd \
               -Dsonar.host.url=$SONAR_HOME \
               -Dsonar.login=$sonarqube_scanner
                '''
                }
            }
        }
        stage('build and push'){
                 steps {
              script {
                 sh ' docker build -t ${DOCKER_IMAGE} .'
                 def dockerImage = docker.image("${DOCKER_IMAGE}")
                  docker.withRegistry('https://index.docker.io/v1/', "docker-cred") {
                  dockerImage.push()
                }
             }
                
            }
        }
        stage('image update'){
            steps{
                withCredentials([string(credentialsId: 'GITHUB-TOKEN', variable: 'GIT_TOKEN')]) {
                 sh '''
                    git config user.email "murthyvella.xyz@gmail.com"
                    git config user.name "murthyvella"
                    BUILD_NUMBER=${BUILD_NUMBER}
                    sed -i "s/replaceTag/${BUILD_NUMBER}/g" deployment-service.yaml
                    git add deployment-service.yaml
                    git commit -m "Update deployment image to version ${BUILD_NUMBER}"
                    git push https://${GIT_TOKEN}@github.com/${GIT_USER_NAME}/${GIT_REPO_NAME} HEAD:main '''
                }
            }
         }
    }
}
