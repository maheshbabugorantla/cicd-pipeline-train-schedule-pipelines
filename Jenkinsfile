pipeline {
    agent any
    stages {
        stage("Build") {
            steps {
                echo "Running Build Automation"
                sh './gradlew build --no-daemon'
                archiveArtifacts artifacts: 'dist/trainSchedule.zip' 
            }
        }
        stage("DeployToStaging") {
            when {
                branch "master"
            }
            steps {
                withCredentials([userNamePassword(credentialsId: 'webserver_login', usernameVariable: 'USERNAME', passwordVariable: 'USERPASS')]) {
                    sshPublisher(
                        failOnError: true,
                        continueOnError: false,
                        publishers: [
                            sshPublisherDesc(
                                configName: 'staging',
                                sshCredentials: [
                                    username: '$USERNAME',
                                    encryptedPassphrase: "$USERPASS"
                                ],
                                transfers: [
                                    sshTransfer(
                                        sourceFiles: 'dist/trainSchedule.zip',
                                        removePrefix: 'dist/',
                                        remoteDirectory: '/tmp',
                                        execCommand: 'sudo /usr/bin/systemctl stop train-schedule && \
                                                      rm -rf /opt/train-schedule/* && \
                                                      unzip /tmp/trainSchedule.zip -d /opt/train-schedule/ && \
                                                      sudo /usr/bin/systemctl start train-schedule'
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