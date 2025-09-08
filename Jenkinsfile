pipeline {
    agent any

    environment {
        EC2_HOST = "3.88.18.183"       // Your EC2 Public IP
        EC2_USER = "ubuntu"            // EC2 Username
        EC2_KEY  = "ec2-ssh-key"       // Jenkins credential ID
    }

    stages {
        stage('Checkout Configs') {
            steps {
                git branch: 'main', url: 'https://github.com/gyadav2809/asterisk-configs.git'
            }
        }

        stage('Deploy to EC2') {
            steps {
                sshagent (credentials: ["${EC2_KEY}"]) {
                    sh """
                    ssh -o StrictHostKeyChecking=no ${EC2_USER}@${EC2_HOST} '
                        set -e
                        # Backup existing configs
                        sudo cp -r /etc/asterisk /etc/asterisk-backup-$(date +%F-%H%M)
                        
                        # Sync new configs
                        sudo rsync -avz --delete ./ /etc/asterisk/
                        
                        # Reload asterisk
                        sudo systemctl reload asterisk
                    '
                    """
                }
            }
        }
    }
}

