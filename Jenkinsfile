 pipeline {
   agent any


   tools {
      go '1.23.2'
   }
   environment {
       DOCKERHUB_CREDENTIALS = credentials('doocker-hub-credential')
       DOCKER_IMAGE = 'mrthcldock/tektondemo'
       GITHUB_CREDENTIALS = 'github'
       SONAR_TOKEN = credentials('sonartoken')
       SNAP_REPO = 'vprofile-snapshot'
       NEXUS_USERNAME = 'admin'
       NEXUS_PASSWORD = 'admin@123'
       RELEASE_REPO = 'docker_nexus'
       CENTRAL_REPO = 'vpro-maven-central'
       NEXUS_IP = '192.168.29.68'
       NEXUS_PORT = '8081'
       NEXUS_GRP_REPO = 'vpro-maven-group'
       SONARSERVER = 'sonarserver'
       SONARSCANNER = 'sonarscanner'
       NEXUS_CREDS = credentials('nexuslogin')
       NEXUS_DOCKER_REPO = 'localhost:8081'
   }


   stages{
       stage('Checkout'){
           steps{
               echo "checking out repo"
               git url: 'https://github.com/Manogithubnew/go-project.git', branch: 'main',
               credentialsId: "${GITHUB_CREDENTIALS}"
           }
       }
       stage ('Sonar Analysis') {
            environment {
                scannerHome = tool "${SONARSCANNER}" 
            }
            steps {
              withSonarQubeEnv("${SONARSERVER}") {
                 sh '''${scannerHome}/bin/sonar-scanner -Dsonar.projectKey=vprofile \
                     -Dsonar.projectName=vprofile-repo \
                     -Dsonar.projectVersion=1.0 \
                     -Dsonar.sources=/var/lib/jenkins/workspace/mrt-ci-pipeline-java/ \
                     -Dsonar.java.binaries=target/test-classes/com/visualpathit/account/controllerTest/ \
                     -Dsonar.exclusions=**/*.js,**/*.ts,**/*.css,**/*.jps \
                     -Dsonar.junit.reportsPath=target/surefire-reports/ \
                     -Dsonar.jacoco.reportsPath=target/jacoco.exec \
                     -Dsonar.java.checkstyle.reportPaths=target/checkstyle-result.xml'''
               }
           }
       }
       stage('Run Docker Build'){
           steps{
               script{
                    echo "starting docker build"
                    sh "docker build -t ${DOCKER_IMAGE}:${env.BUILD_ID} ."
                    echo "docker built successfully"
               }
           }
       }
       
       stage('Push to Nexus') {
           steps {
               script {
                   docker.withRegistry('http://192.168.29.68:8081/repository/docker-nexus/', 'nexuslogin') {
                   }
            
                   sh "echo '${nexusCreds.password}' | docker login -u ${nexusCreds.username} --password-stdin http://192.168.29.68:8085/repository/docker-nexus"
                   sh 'docker tag ${DOCKER_IMAGE}:${env.BUILD_ID} http://192.168.29.68:8085/repository/docker-nexus:latest'
                   sh 'docker push http://192.168.29.68:8085/repository/docker-nexus:latest'
               }
           }
       }
   }
}
