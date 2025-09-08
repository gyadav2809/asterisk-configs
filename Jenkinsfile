pipeline {
    agent any

    environment {
        EC2_HOST = "3.88.18.183"       // Asterisk EC2 IP
        EC2_USER = "ubuntu"            // EC2 username
        EC2_KEY  = "ec2-ssh-key"       // Jenkins credential ID for SSH key
        CONFIG_DIR = "/etc/asterisk"   // Directory on Asterisk EC2
    }

    stages {
        stage('Checkout Configs') {
            steps {
                git branch: 'main', url: 'https://github.com/gyadav2809/asterisk-configs.git'
            }
        }

        stage('Deploy to EC2') {
            steps {
                sshagent([EC2_KEY]) {
                    sh '''
                        ssh -o StrictHostKeyChecking=no ${EC2_USER}@${EC2_HOST} "
                            set -e

                            # Create backup
                            BACKUP_DIR=${CONFIG_DIR}-backup-$(date +%F-%H%M)
                            echo 'Backing up old configs to' $BACKUP_DIR
                            sudo cp -r ${CONFIG_DIR} $BACKUP_DIR

                            # Deploy new configs
                            echo 'Deploying new configs...'
                            sudo rsync -avz --delete ./ ${CONFIG_DIR}/

                            # Validate configuration (basic check)
                            echo 'Validating Asterisk configs...'
                            sudo asterisk -rx 'core show help' >/dev/null

                            # Reload Asterisk
                            echo 'Reloading Asterisk...'
                            sudo systemctl reload asterisk

                            echo 'Deployment complete.'
                        "
                    '''
                }
            }
        }
    }

    post {
        failure {
            echo "Deployment failed. You can manually restore backup from /etc/asterisk-backup-<timestamp>"
        }
        success {
            echo "CI/CD deployment to Asterisk EC2 succeeded."
        }
    }
}
