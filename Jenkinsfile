pipeline {
    agent {
        docker {
           
            image 'maven:3-openjdk-8' 
            args '-v /root/.m2:/root/.m2' 
        }
    }
    stages {
        stage('Print environment') { 
            steps {
                echo "${DATASOURCE_USER}"
                echo "${JWT_SECRET}"
 //               echo "$(JWT_SECRET)"
            }
        }
        stage('Build') { 
            steps {
                 withEnv(['DATASOURCE_PASSWORD=${DATASOURCE_PASSWORD} '])
                 withEnv(['DATASOURCE_URL=${DATASOURCE_URL} '])
                 withEnv(['DATASOURCE_USER=${DATASOURCE_USER} '])
                 withEnv(['JWT_SECRET=${JWT_SECRET} '])
                echo 'Running build automation'
                sh 'mvn clean package' 
            }
        }
    stage('DeployToStaging') {
            when {
                branch 'main'
            }
            steps {
                withCredentials([usernamePassword(credentialsId: 'backend', usernameVariable: 'USERNAME', passwordVariable: 'USERPASS')]) {
                    sshPublisher(
                        failOnError: true,
                        continueOnError: false,
                        publishers: [
                            sshPublisherDesc(
                                configName: 'backend_root',
                                sshCredentials: [
                                    username: "$USERNAME",
                                    encryptedPassphrase: "$USERPASS"
                                ], 
                                transfers: [
                                    sshTransfer(
                                        sourceFiles: 'target/TeachUA-1.0.war',
                                        removePrefix: 'target/',
                                        remoteDirectory: '/home/teachua/www/back',
                                         execCommand: 'sudo mv /home/teachua/www/back/TeachUA-1.0.war /home/teachua/www/back/dev.war' 
                                        //                                        execCommand: 'sudo /usr/bin/systemctl stop train-schedule && rm -rf /opt/train-schedule/* && unzip /tmp/trainSchedule.zip -d /opt/train-schedule && sudo /usr/bin/systemctl start train-schedule'
                                    )
                                ]
                            )
                        ]
                    )
                }
            }
        }
    }
}
