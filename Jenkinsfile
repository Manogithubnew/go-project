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
       NEXUS_USER = 'admin'
       NEXUS_PASS = 'admin@123'
       RELEASE_REPO = 'docker_nexus'
       CENTRAL_REPO = 'vpro-maven-central'
       NEXUS_IP = '192.168.29.68'
       NEXUS_PORT = '8081'
       NEXUS_GRP_REPO = 'vpro-maven-group'
       NEXUS_LOGIN = 'nexuslogin'
       SONARSERVER = 'sonarserver'
       SONARSCANNER = 'sonarscanner'
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
       stage ("Upload Artifact") {
            steps {
                nexusArtifactUploader(
                  nexusVersion: 'nexus3',
                  protocol: 'http',
                  nexusUrl: "${NEXUS_IP}:${NEXUS_PORT}",  
                  groupId: 'QA',
                  version: "${env.BUILD_ID}-${env.BUILD_TIMESTAMP}",
                  repository: "${RELEASE_REPO}",
                  credentialsId: "${NEXUS_LOGIN}", 
                  artifacts: [
                    [artifactId: 'vproapp',
                     classifier: '',
                     file: 'target/vprofile-v2.war',
                     type: 'war']
                  ]
                )
            }
        }
    }
}
