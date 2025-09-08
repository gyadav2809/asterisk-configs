pipeline {
    agent any

    environment {
        EC2_HOST   = "3.88.18.183"       // Asterisk EC2 IP
        EC2_USER   = "ubuntu"            // EC2 username
        EC2_KEY    = "ec2-ssh-key"       // Jenkins credential ID for SSH key
        CONFIG_DIR = "/etc/asterisk"     // Directory on Asterisk EC2
    }

    stages {
        stage('Checkout Configs') {
            steps {
                git branch: 'main', url: 'https://github.com/gyadav2809/asterisk-configs.git'
            }
        }

        stage('Backup Old Configs') {
            steps {
                sshagent([EC2_KEY]) {
                    sh """
                        ssh -o StrictHostKeyChecking=no ${EC2_USER}@${EC2_HOST} '
                            set -e
                            BACKUP_DIR=${CONFIG_DIR}-backup-\$(date +%F-%H%M)
                            echo "Backing up existing configs to \$BACKUP_DIR"
                            sudo cp -r ${CONFIG_DIR} \$BACKUP_DIR
                        '
                    """
                }
            }
        }

        stage('Deploy New Configs') {
            steps {
                sshagent([EC2_KEY]) {
                    sh """
                        ssh -o StrictHostKeyChecking=no ${EC2_USER}@${EC2_HOST} '
                            set -e
                            echo "Deploying new configs from GitHub..."
                            sudo rsync -avz --delete ./ ${CONFIG_DIR}/
                            sudo chown -R ubuntu:ubuntu ${CONFIG_DIR}
                            echo "Reloading Asterisk..."
                            sudo asterisk -rx "core reload"
                        '
                    """
                }
            }
        }

        stage('Verify Deployment') {
            steps {
                sshagent([EC2_KEY]) {
                    sh """
                        ssh -o StrictHostKeyChecking=no ${EC2_USER}@${EC2_HOST} '
                            set -e
                            echo "Checking Asterisk status..."
                            sudo systemctl status asterisk | head -n 10
                            echo "Listing /etc/asterisk contents..."
                            ls -l ${CONFIG_DIR}
                        '
                    """
                }
            }
        }
    }

    post {
        success {
            echo "CI/CD deployment succeeded. Old configs are backed up."
        }
        failure {
            echo "Deployment failed! You can restore backup manually from /etc/asterisk-backup-<timestamp>"
        }
    }
}
