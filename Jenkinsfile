pipeline {
    agent any

    environment {
        NEXUS_URL = "http://52.23.170.163:8081/repository/maven-repo/"
        NEXUS_CREDENTIALS_ID = "nexus-credential"
        SONARQUBE_URL = "http://<IP_SONARQUBE>:9000"
        SONARQUBE_TOKEN = "<TOKEN_SONARQUBE>"
        WINDOWS_SERVER = "54.90.149.12"  
        SSH_CREDENTIALS_ID = "windows-ssh" 
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

        // stage('Code Analysis with SonarQube') {
        //     steps {
        //         withSonarQubeEnv('SonarQube') {
        //             sh 'mvn sonar:sonar -Dsonar.host.url=$SONARQUBE_URL -Dsonar.login=$SONARQUBE_TOKEN'
        //         }
        //     }
        // }

        stage('Upload to Nexus') {
            steps {
                nexusArtifactUploader(
                    nexusVersion: 'nexus3',
                    protocol: 'http',
                    nexusUrl: '52.23.170.163:8081',
                    repository: 'maven-repo',
                    credentialsId: NEXUS_CREDENTIALS_ID,
                    groupId: 'com.example',
                    version: '1.0.0-SNAPSHOT',
                    artifacts: [
                        [artifactId: 'java-getting-started', file: 'target/java-getting-started-1.0.0-SNAPSHOT.war', classifier: '', type: 'war']
                    ]
                )
            }
        }

        stage('Deploy to Windows') {
            steps {
                script {
                     def warFile = 'java-getting-started-1.0.0-SNAPSHOT.war'

                    // Télécharger le .war depuis Nexus
                     // Récupérer la dernière version du .war
                    sh """
                        latest_war=\$(curl -s -u admin:admin "http://52.23.170.163:8081/repository/maven-repo/com/example/java-getting-started/1.0.0-SNAPSHOT/maven-metadata.xml" | grep -oP '(?<=<value>).*?(?=</value>)' | tail -1)
                        echo "Dernière version détectée: \$latest_war"

                        curl -u admin:admin -o ${warFile} \
                        http://52.23.170.163:8081/repository/maven-repo/com/example/java-getting-started/1.0.0-SNAPSHOT/java-getting-started-\$latest_war.war
                    """

                    // Copie test sur le bureau de l'admin
                    sshPut remote: [
                        name:'WindowsServer',
                        host: WINDOWS_SERVER,
                        user: 'Administrator',
                        password: 'UL64DOE3YK5vc@8387lRgd9xS%k%8bP6',
                        port: 22
                    ], from: warFile, into: "C:\\Users\\Administrator\\Desktop\\${warFile}"

                    // Copier le .war sur le serveur Windows via SSH
                    // sshPut remote: [
                    //     host: WINDOWS_SERVER,
                    //     credentialsId: 'windows-ssh',
                    //     port: 22
                    // ], from: warFile, into: "C:\\Tomcat\\webapps\\${warFile}"

                    // Redémarrer Tomcat via SSH
                    // sshCommand remote: [
                    //     host: WINDOWS_SERVER,
                    //     credentialsId: 'windows-ssh',
                    //     port: 22
                    // ], command: "C:\\Tomcat\\bin\\shutdown.bat && C:\\Tomcat\\bin\\startup.bat"
                }
            }
        }  
    }  
} 
