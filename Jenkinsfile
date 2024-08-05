pipeline {
    agent any
    environment {
        GH_TOKEN = credentials('git-pat')
        HARDCODED_RECIPIENT = 'atiq.khan@gmail.com'
    }
    parameters {
        string(name: 'SERVICE', defaultValue: '', description: 'Name of the Service for Merge Request')
        string(name: 'QA_INITIATOR', defaultValue: '', description: 'Name of the QA who is sending merge request')
        string(name: 'QA_APPROVER', defaultValue: '', description: 'Name of the QA who approves merge request')
        string(name: 'EMAIL_RECIPIENTS', defaultValue: '', description: 'Comma-separated list of additional email recipients')
    }
    stages {
        stage('Fetch, Merge, and Push') {
            steps {
                script {
                    // Configure Git to use the token for authentication
                    sh '''
                        #!/bin/bash
                        git config --global credential.helper store
                        echo "https://${GH_TOKEN}:@github.com" > ~/.git-credentials
                    '''
                    // Clone the repository if it does not exist
                    sh """
                        #!/bin/bash
                        if [ ! -d ".git" ]; then
                            git clone https://github.com/khan-atiq/${params.SERVICE}.git .
                        fi
                    """
                    // Perform the Git operations
                    sh '''
                        #!/bin/bash
                        git fetch origin
                        git checkout pre-prod
                        git pull origin pre-prod
                        if git merge origin/qa; then
                            echo "Merge successful. Ready for approval."
                        else
                            echo "Merge conflict detected. Please resolve conflicts manually."
                            exit 1
                        fi
                    '''
                }
            }
        }
        stage('Approval') {
            steps {
                script {
                    // Ensure the build initiator is not the approver
                    def qaInitiator = "${params.QA_INITIATOR}"
                    def qaApprover = "${params.QA_APPROVER}"
                    if (qaInitiator != qaApprover) {
                        input message: 'Please approve the merge to the pre-prod branch.',
                              ok: 'Approve',
                              submitter: "${qaApprover}"
                    } else {
                        error "The merge initiator and approver cannot be the same person."
                    }
                }
            }
        }
        stage('Push Changes') {
            steps {
                script {
                    // Push changes after approval
                    sh '''
                        #!/bin/bash
                        git push origin pre-prod
                    '''
                }
            }
        }
    }
    post {
        success {
            emailext (
                subject: "Build Successful: ${env.JOB_NAME} #${env.BUILD_NUMBER}",
                body: """\
                The build ${env.JOB_NAME} #${env.BUILD_NUMBER} was successful.
                
                Check console output at ${env.BUILD_URL} to see the build details.
                """,
                to: "${env.HARDCODED_RECIPIENT}, ${params.EMAIL_RECIPIENTS}"
            )
        }
        failure {
            emailext (
                subject: "Build Failed: ${env.JOB_NAME} #${env.BUILD_NUMBER}",
                body: """\
                The build ${env.JOB_NAME} #${env.BUILD_NUMBER} failed.
                
                Check console output at ${env.BUILD_URL} to see the build details.
                """,
                to: "${env.HARDCODED_RECIPIENT}, ${params.EMAIL_RECIPIENTS}"
            )
        }
    }
}
