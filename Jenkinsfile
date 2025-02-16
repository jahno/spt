pipeline {
    agent any

    environment {
        NEXUS_URL = "http://52.23.170.163:8081/repository/maven-repo/"
        NEXUS_CREDENTIALS_ID = "nexus-credential"
        WINDOWS_SERVER = "54.90.149.12"
        WAR_FILE = "java-getting-started-1.0.0-SNAPSHOT.war"
        WINDOWS_DEST = "C:/Users/Administrator/Desktop/${WAR_FILE}"
    }

    stages {
        stage('Checkout Code') {
            steps {
                echo "🔍 Vérification du code source..."
                checkout scm
            }
        }

        stage('Build & Test') {
            steps {
                echo "🏗️ Compilation du projet..."
                sh 'mvn clean package'
            }
        }

        stage('Upload to Nexus') {
            steps {
                echo "🚀 Upload du WAR vers Nexus..."
                nexusArtifactUploader(
                    nexusVersion: 'nexus3',
                    protocol: 'http',
                    nexusUrl: '52.23.170.163:8081',
                    repository: 'maven-repo',
                    credentialsId: NEXUS_CREDENTIALS_ID,
                    groupId: 'com.example',
                    version: '1.0.0-SNAPSHOT',
                    artifacts: [
                        [artifactId: 'java-getting-started', file: "target/${WAR_FILE}", classifier: '', type: 'war']
                    ]
                )
            }
        }

        stage('Deploy to Windows') {
            steps {
                script {
                    echo "🚀 Déploiement du WAR sur Windows..."

                    // 🔒 Utilisation sécurisée des credentials Nexus
                    withCredentials([usernamePassword(credentialsId: 'nexus-credential', usernameVariable: 'NEXUS_USER', passwordVariable: 'NEXUS_PASS')]) {
                        echo "🔍 Récupération de la dernière version du WAR depuis Nexus..."
                        sh """
                            latest_war=\$(curl -s -u "$NEXUS_USER:$NEXUS_PASS" "$NEXUS_URL/com/example/java-getting-started/1.0.0-SNAPSHOT/maven-metadata.xml" | grep -oP '(?<=<value>).*?(?=</value>)' | sort -V | tail -1)
                            echo "✅ Dernière version détectée: \$latest_war"

                            if [ -z "\$latest_war" ]; then
                                echo "❌ ERREUR: Impossible de détecter la version du WAR !" && exit 1
                            fi

                            curl -u "$NEXUS_USER:$NEXUS_PASS" -o "${env.WORKSPACE}/${WAR_FILE}" "$NEXUS_URL/com/example/java-getting-started/1.0.0-SNAPSHOT/java-getting-started-\$latest_war.war"

                            echo "✅ Fichier téléchargé :"
                            ls -lh "${env.WORKSPACE}/${WAR_FILE}"
                        """
                    }

                    // 🔒 Utilisation sécurisée du mot de passe Windows SSH
                    withCredentials([string(credentialsId: 'windows-ssh-password', variable: 'SSH_PASSWORD')]) {
                        echo "📤 Transfert du fichier vers Windows..."
                        sh """
                            sshpass -p "$SSH_PASSWORD" scp -P 22 "${env.WORKSPACE}/${WAR_FILE}" Administrator@$WINDOWS_SERVER:"${WINDOWS_DEST}"
                        """

                        // Vérifier que le fichier est bien présent sur Windows
                        echo "📂 Vérification de la présence du fichier sur Windows..."
                        sshCommand remote: [
                            name: 'WindowsServer',
                            host: WINDOWS_SERVER,
                            user: 'Administrator',
                            password: SSH_PASSWORD,
                            allowAnyHosts: true,
                            port: 22
                        ], command: "dir C:\\Users\\Administrator\\Desktop\\"

                        // Redémarrer Tomcat après le déploiement (optionnel)
                        // echo "🔄 Redémarrage de Tomcat..."
                        // sshCommand remote: [
                        //     name: 'WindowsServer',
                        //     host: WINDOWS_SERVER,
                        //     user: 'Administrator',
                        //     password: SSH_PASSWORD,
                        //     allowAnyHosts: true,
                        //     port: 22
                        // ], command: "C:\\Tomcat\\bin\\shutdown.bat && C:\\Tomcat\\bin\\startup.bat"
                    }
                }
            }
        }
    }
}

