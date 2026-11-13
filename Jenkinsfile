// Check GitHub every 2 minutes for new commits
properties([pipelineTriggers([pollSCM('H/2 * * * *')])])

node {
    stage('Preparation') {
        // Check out the repo, so Jenkins knows which commit was built (needed for Poll SCM)
        checkout scm
        catchError(buildResult: 'SUCCESS') {
            sh 'docker stop samplerunning'
            sh 'docker rm samplerunning'
        }
    }
    stage('Build') {
        build 'BuildSampleApp'
    }
    stage('Results') {
        build 'TestSampleApp'
    }
    // Deploy the same app to production (dmz010) with Docker over SSH
    stage('Deploy to production') {
        sshagent(['dmz010-deploy']) {
            // Trust the host key of dmz010
            sh 'mkdir -p ~/.ssh && ssh-keyscan 192.0.2.10 >> ~/.ssh/known_hosts'
            withEnv(['DOCKER_HOST=ssh://jenkins@192.0.2.10']) {
                sh 'docker rm -f samplerunning || true'
                sh 'rm -rf tempdir'
                sh 'bash sample-app.sh'
            }
        }
    }
}