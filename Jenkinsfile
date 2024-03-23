pipeline {
    agent any
    tools{
        jdk 'jdk17'
        maven 'maven3'
    }
    environment{
        SONAR_SCANNER = tool 'sonar-scanner'
        NEW_IMAGE = ''
        DOCKER_IMAGE = "utsab12312/boardgame:$BUILD_NUMBER"
        REPO_URL = 'https://github.com/utsab818/boardGame.git'
        DEPLOYMENT_FILE = 'k8s/deployment-service.yaml'
    }

    stages {
        stage('Git checkout'){
            steps{
                git branch: 'main', url: 'https://github.com/utsab818/boardGame.git'
            }
        }
        stage('Maven compile'){
            steps{
                sh 'mvn compile'
            }
        }
        stage('Maven test'){
            steps{
                sh 'mvn test'
            }
        }
        stage('Trivy Filesystem Scan'){
            steps{
                sh 'trivy fs --format table -o trivy-fs-report.html .'
            }
        }
        stage('Sonarqube Analysis'){
            steps{
                withSonarQubeEnv('sonar') {
                    sh '''
                        $SCANNER_HOME/bin/sonar-scanner -Dsonar.projectName=BoardGame \
                        -Dsonar.projectKey=BoardGame -Dsonar.java.binaries=.
                    '''
                }
            }
        }
        stage('Quality Gate'){
            steps{
                script{
                    waitForQualityGate abortPipeline: false, credentialsId: 'sonar-token'
                }
            }
        }
        stage('Maven build'){
            steps{
                sh 'mvn package'
            }
        }
        stage('Publish to Nexus'){
            steps{
                withMaven(globalMavenSettingsConfig: 'global-settings', jdk: 'jdk17', maven: 'maven3', mavenSettingsConfig: '', traceability: true) {
                    sh "mvn deploy"
                }
            }
        }
        stage('Delete previous docker image'){
            steps {
                script {
                    def currentBuildNumber = currentBuild.number
                    def prevBuildNumber = currentBuildNumber.toInteger() - 1
                    def imageExists = sh(script: "docker images -q utsab12312/boardgame:$prevBuildNumber", returnStatus: true) == 0
                    if (imageExists) {
                        try {
                            sh "docker rmi utsab12312/boardgame:$prevBuildNumber"
                        } catch (Exception e) {
                            echo "Failed to remove Docker image: $e"
                        }
                    } else {
                        echo "No previous Docker image found for build number $prevBuildNumber. Skipping deletion."
                    }
                }
            }
        }
        stage('Build and tag Docker image'){
            steps{
                script{
                    // DOCKER_IMAGE = "utsab12312/boardgame:$BUILD_NUMBER"
                    sh "docker build -t $DOCKER_IMAGE ."
                }
            }
        }  
        stage('Trivy Dockerfile Scan'){
            steps{
                sh "trivy image --format table -o trivy-image-report.html $DOCKER_IMAGE"
            }
        }

        stage('Push to dockerhub'){
            steps{
                script{
                    withDockerRegistry(credentialsId: 'dockerhub_pass', toolName: 'docker'){
                        sh "docker push $DOCKER_IMAGE"
                    }
                }
            }
        }

        // for updating the kubernetes manifest with new image
        stage('Clone repository'){
            steps{
                checkout([$class: 'GitSCM', branches: [[name: '*/main']], userRemoteConfigs: [[url: REPO_URL]]])
            }
        }
        stage('Update Docker image'){
            steps{
                script{
                    //fetch the new image from dockerhub
                    // NEW_IMAGE = sh(script: "docker pull $DOCKER_IMAGE && docker inspect --format='{{.RepoTags}}' $DOCKER_IMAGE | awk '{print $1}'", returnStdout: true).trim()
                    // sed -i "pattern|changePattern" filename
                    sh "sed -i \"s|image:.*|image: $DOCKER_IMAGE|\" $DEPLOYMENT_FILE"
                    echo "Updated Docker image to $DOCKER_IMAGE in $DEPLOYMENT_FILE"
                }
            }
        }
        stage('Push Changes'){
            steps{
                script{
                    withCredentials([string(credentialsId: 'git-cred', variable: 'GITHUB_TOKEN')]) {
                        sh '''
                            git config user.email "lolgang08@gmail.com"
                            git config user.name "utsab818" 
                            git add ${DEPLOYMENT_FILE}
                            git commit -m "Updated deployment image to version ${BUILD_NUMBER}"
                            git push https://${GITHUB_TOKEN}@github.com/utsab818/boardGame.git HEAD:main
                        '''
                    } 
                }
            }
        }
    }
}
