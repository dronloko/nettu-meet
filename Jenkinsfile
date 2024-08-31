pipeline {
  agent any
    stages {
      stage ('semgrep') {
        steps {
          script {
            catchError(buildResult: 'SUCCESS', stageResult: 'FAILURE') {
              sh '''
              echo "Running semgrep..."
              apk add python3 py3-pip py3-virtualenv
              python3 -m venv venv
              . venv/bin/activate
              pip install semgrep
              mkdir reports/
              semgrep ci --config=auto --json > reports/semgrep.json
              '''
            }
            archiveArtifacts artifacts: 'reports/*', allowEmptyArchive: true
          }
        }
      }
      stage ('trivy') {
          steps {
            script {
                sh '''
                cd server/
                docker build . -t nettu-meet:latest
                apk add trivy
                trivy image --format json --severity HIGH,CRITICAL,WARNING nettu-meet:latest
                '''
              }
              archiveArtifacts artifacts: 'reports/*', allowEmptyArchive: true
          }
    }
  }
}
