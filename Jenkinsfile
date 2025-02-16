pipeline {
    agent any

    environment {
        NEXUS_URL = "http://52.23.170.163:8081/repository/maven-repo/"
        NEXUS_CREDENTIALS_ID = "nexus-credential"
        SONARQUBE_URL = "http://<IP_SONARQUBE>:9000"
        SONARQUBE_TOKEN = "<TOKEN_SONARQUBE>"
    }

    stages {
        stage('Checkout Code') {
            steps {
                  echo "Checking out code from SCM..."
                checkout scm
            }
        }

        stage('Build & Test') {
            steps {
                sh 'mvn clean package'
            }
        }

      //  stage('Code Analysis with SonarQube') {
        //    steps {
          //      withSonarQubeEnv('SonarQube') {
            //        sh 'mvn sonar:sonar -Dsonar.host.url=$SONARQUBE_URL -Dsonar.login=$SONARQUBE_TOKEN'
              //  }
           // }
       // }

        stage('Upload to Nexus') {
            steps {
                          nexusArtifactUploader(
            nexusVersion: 'nexus3',
            protocol: 'http',
            nexusUrl: '52.23.170.163:8081',
            repository: 'maven-releases',
            credentialsId: NEXUS_CREDENTIALS_ID,
            groupId: 'com.example',
            version: '1.0.0-SNAPSHOT',
            artifacts: [
                [file: 'target/java-getting-started-1.0.0-SNAPSHOT.jar', type: 'jar']
            ]
        )

            }
        }
    }
}

