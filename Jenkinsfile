pipeline {
    agent any
    options {
        // Running builds concurrently could cause a race condition with
        // building the Docker image.
        disableConcurrentBuilds()
    }
    triggers {
        cron('0 * * * *')
    }
    stages {
        // Run the build in the against the dev branch to check for compile errors
        stage('Run Integration Tests') {
            when {
                anyOf {
                    branch 'testing/behave'
                    branch 'dev'
                    branch 'master'
                    triggeredBy 'TimerTrigger'
                    changeRequest target: 'dev'
                }
            }
            steps {
                sh 'docker build --no-cache --target voigt_kampff -t mycroft-core:latest .'
                sh 'docker run \
                    -v "$HOME/voigtmycroft:/root/.mycroft" \
                    --device /dev/snd \
                    -e PULSE_SERVER=unix:${XDG_RUNTIME_DIR}/pulse/native \
                    -v ${XDG_RUNTIME_DIR}/pulse/native:${XDG_RUNTIME_DIR}/pulse/native \
                    -v ~/.config/pulse/cookie:/root/.config/pulse/cookie \
                    mycroft-core:latest'
            }
        }
        
    }
    post {
        always('Important stuff') {
            sh 'mv "$HOME/voigtmycroft/behave.html" ./'
            publishHTML (target: [
              allowMissing: false,
              alwaysLinkToLastBuild: false,
              keepAll: true,
              reportDir: '.',
              reportFiles: 'behave.html',
              reportName: "Behave Report"
            ])
            echo 'Cleaning up docker containers and images'
            sh 'docker container prune --force --filter "until=3h"'
            sh 'docker image prune --force'
        }
    }
}
