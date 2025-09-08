pipeline {
    agent any
    stages {
        stage('Test SSH Connection') {
            steps {
                sshagent(['ec2-ssh-key']) {
                    sh "ssh -o StrictHostKeyChecking=no ubuntu@3.88.18.183 'hostname && whoami'"
                }
            }
        }
    }
}
