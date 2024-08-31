pipeline {
  agent any
    stages {
      /*stage ('semgrep') {
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
              semgrep scan --config=auto . --exclude=venv --json > reports/semgrep.json
              '''
            }
            archiveArtifacts artifacts: 'reports/*', allowEmptyArchive: true
          }
        }
      }*/
      stage ('trivy') {
          agent { label "dind" }
          steps {
            script {
                sh '''
                sudo apt-get update
                sudo apt-get install ca-certificates curl
                sudo install -m 0755 -d /etc/apt/keyrings
                sudo curl -fsSL https://download.docker.com/linux/ubuntu/gpg -o /etc/apt/keyrings/docker.asc
                sudo chmod a+r /etc/apt/keyrings/docker.asc
                sudo apt-get install docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin
                cd server/
                docker build . -t nettu-meet:latest
                sudo apt install trivy
                mkdir reports/
                trivy image --format json --severity HIGH,CRITICAL,WARNING nettu-meet:latest > reports/trivy.json
                '''
              }
              archiveArtifacts artifacts: 'reports/*', allowEmptyArchive: true
          }
      }
  }
}
