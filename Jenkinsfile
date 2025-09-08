pipeline {
    agent any
    stages {
        stage('Test SSH Connection') {
            steps {
                sshagent(['ec2-ssh-key']) {
                    sh "ssh -o StrictHostKeyChecking=no ubuntu@<ASTERISK_EC2_IP> 'hostname && whoami'"
                }
            }
        }
    }
}
