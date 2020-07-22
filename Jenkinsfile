pipeline {
  agent any
  stages {
    stage('Build') {
      steps {
        echo 'Running build automation'
        sh './gradlew build --no-daemon'
        archiveArtifacts artifacts: 'dist/trainSchedule.zip'
      }
    }
    stage('DeployToStaging') {
      when {
        branch 'without-docker'
      }
      steps {
        withCredentials([usernamePassword(credentialsId: 'deployment_login', usernameVariable: 'USERNAME', passwordVariable: 'USERPASS')]) {
          sshPublisher(
            failOnError: true,
            continueOnError: false,
            publishers: [
              sshPublisherDesc(
                configName: 'staging',
                sshCredentials: [
                  username: "$USERNAME",
                  encryptedPassphrase: "$USERPASS"
                ],
                transfers: [
                  sshTransfer(
                    sourceFiles: 'dist/trainSchedule.zip',
                    removePrefix: 'dist/',
                    remoteDirectory: '/tmp',
                    execCommand: 'rm -rf /opt/train-schedule/* && unzip /tmp/trainSchedule.zip -d /opt/train-schedule'
                  )
                ]
              )
            ]
          )
        }
      }
    }
    stage('DeployToProduction') {
      when {
        branch 'without-docker'
      }
      steps {
        input 'Does the staging environment look OK?'
        milestone(1)
        withCredentials([usernamePassword(credentialsId: 'deployment_login', usernameVariable: 'USERNAME', passwordVariable: 'USERPASS')]) {
          sshPublisher(
            failOnError: true,
            continueOnError: false,
            publishers: [
              sshPublisherDesc(
                configName: 'production',
                sshCredentials: [
                  username: "$USERNAME",
                  encryptedPassphrase: "$USERPASS"
                ],
                transfers: [
                  sshTransfer(
                    sourceFiles: 'dist/trainSchedule.zip',
                    removePrefix: 'dist/',
                    remoteDirectory: '/tmp',
                    execCommand: 'rm -rf /opt/train-schedule/* && unzip /tmp/trainSchedule.zip -d /opt/train-schedule'
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
