pipeline {
    agent any

    environment {
        JENKINS_HOME_DIR = "/var/lib/jenkins"   // Change if different
        BACKUP_DIR = "/tmp/jenkins_backup"
        GIT_REPO = "git@github.com:pavi426/jenkins-backup-repo"
        BACKUP_FILE = "jenkins-backup-${new Date().format('yyyyMMdd-HHmm')}.tar.gz"
    } 



    stages {
        stage('Clean Old Backup') {
            steps {
                sh '''
                    rm -rf $BACKUP_DIR
                    mkdir -p $BACKUP_DIR
                '''
            }
        }

        stage('Create Backup Archive') {
            steps {
                sh '''
                    tar -czf $BACKUP_DIR/$BACKUP_FILE -C $JENKINS_HOME_DIR .
                '''
            }
        }

        stage('Push to GitHub') {
            steps {
                sh '''
                    cd $BACKUP_DIR
                    git init
                    git remote add origin $GIT_REPO || true
                    git checkout -b main || git checkout main
                    git add .
                    git commit -m "Backup taken on $(date)"
                    git push origin main --force
                '''
            }
        }
    }
}
