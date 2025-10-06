pipeline {
    agent any

    environment {
        JENKINS_HOME_DIR = "/var/lib/jenkins"
        BACKUP_DIR = "/tmp/jenkins_backup"
        GIT_REPO = "git@github.com:pavi426/jenkins-backup-repo"
        // This is executed on the controller, use with caution regarding timezones
        BACKUP_FILE = "jenkins-backup-${new Date().format('yyyyMMdd-HHmm')}.tar.gz" 
    }

    stages {
        stage('Clean Old Backup') {
            steps {
                sh 'rm -rf $BACKUP_DIR && mkdir -p $BACKUP_DIR'
            }
        }

        stage('Create Backup Archive') {
            steps {
                // The -C $JENKINS_HOME_DIR . is correct for archiving contents
                sh "tar -czf $BACKUP_DIR/$BACKUP_FILE -C $JENKINS_HOME_DIR ."
            }
        }

        stage('Push to GitHub') {
            // 🚨 CRITICAL FIX: Use sshagent for authentication 🚨
            sshagent(credentials: ['github-ssh-key']) { 
                steps {
                    sh '''
                        cd $BACKUP_DIR
                        git init
                        
                        # Set user/email to avoid warnings/errors
                        # git config user.email "jenkins@backup.com"
                        # git config user.name "Jenkins Backup Job"
                        
                        # Add remote (the || true handles re-runs)
                        git remote add origin $GIT_REPO || true
                        
                        # Use -b to create the branch on the first run, or checkout otherwise
                        git checkout -b main || git checkout main
                        
                        # Add the backup file only
                        git add $BACKUP_FILE
                        
                        git commit -m "Backup taken on $(date)"
                        
                        # Push will now use the SSH key loaded by sshagent
                        git push origin main --force
                    '''
                }
            }
        }
    }
}
