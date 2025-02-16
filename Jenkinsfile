pipeline {
    agent any

    environment {
        NEXUS_URL = "http://52.23.170.163:8081/repository/maven-repo/"
        NEXUS_CREDENTIALS_ID = "nexus-credential"
        WINDOWS_SERVER = "54.90.149.12"
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

                    // Récupérer la dernière version du .war
                    sh """
                        latest_war=\$(curl -s -u admin:admin "$NEXUS_URL/com/example/java-getting-started/1.0.0-SNAPSHOT/maven-metadata.xml" | grep -oP '(?<=<value>).*?(?=</value>)' | sort -V | tail -1)
                        echo "Dernière version détectée: \$latest_war"

                        curl -u admin:admin -o ${warFile} "$NEXUS_URL/com/example/java-getting-started/1.0.0-SNAPSHOT/java-getting-started-\$latest_war.war"

                        echo "Fichier téléchargé, vérification de la taille:"
                        ls -lh ${warFile}
                    """

                    // 🔒 Utilisation sécurisée du mot de passe depuis les credentials
                    withCredentials([string(credentialsId: 'windows-ssh-password', variable: 'SSH_PASSWORD')]) {
                        sh """
                            sshpass -p "$SSH_PASSWORD" scp -P 22 ${warFile} Administrator@$WINDOWS_SERVER:"C:/Users/Administrator/Desktop/"
                        """


                           // Vérifier si le fichier est bien présent sur Windows
                    sshCommand remote: [
                        name: 'WindowsServer',
                        host: WINDOWS_SERVER,
                        user: 'Administrator',
                        password: SSH_PASSWORD,
                        allowAnyHosts: true,
                        port: 22
                     ], command: "dir C:\\Users\\Administrator\\Desktop\\"


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
}

