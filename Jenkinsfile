pipeline {
    agent any
    environment {
        SSH_CREDENTIALS_ID = 'bc013f38-40d9-4731-8ed1-23c56055cc0f' // Replace with your SSH credential ID from you jenkin dashboard
    }
    stages {
        stage('Checkout') {
            steps {
                // Pull the latest code from the Git repository
                git branch: 'master', url: 'https://github.com/novrian6/api_go_jenkins_demo.git'
            }
        }
        stage('Build') {
            steps {
                // Build the Go application
                sh 'go build -o hello-world-api'
            }
        }
        stage('Test') {
            steps {
                // Run Go unit tests
                sh 'go test -v ./...'
            }
        }
        stage('Deploy') {
            steps {
                sshagent(credentials: [SSH_CREDENTIALS_ID]) {
                    script {
                        // Commands to be executed on the remote server
                        def remoteCommands = '''
                        # Stop existing Go binary process if running
                        pkill hello-world-api || true
                        # Create target directory
                        mkdir -p ~/go_apps/production
                        # Remove old binary if exists
                        rm -f ~/go_apps/production/hello-world-api
                        # Copy the new binary
                        scp -o StrictHostKeyChecking=no -i ${SSH_AUTH_SOCK} hello-world-api nn@172.16.137.133:~/go_apps/production/
                        # Change directory and make the binary executable
                        chmod +x ~/go_apps/production/hello-world-api
                        cd ~/go_apps/production
                        # Run the new binary in the background
                        nohup ./hello-world-api > /dev/null 2>&1 &
                        '''
                        // Execute commands on the remote server
                        sh "ssh -o StrictHostKeyChecking=no nn@172.16.137.133 '${remoteCommands}'"
                    }
                }
            }
        }
    }
    post {
        always {
            cleanWs()
        }
    }
}
