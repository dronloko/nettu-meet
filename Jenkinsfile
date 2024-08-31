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
                sudo apt-get install wget apt-transport-https gnupg lsb-release
                wget -qO - https://aquasecurity.github.io/trivy-repo/deb/public.key | sudo apt-key add -
                echo deb https://aquasecurity.github.io/trivy-repo/deb $(lsb_release -sc) main | sudo tee -a /etc/apt/sources.list.d/trivy.list
                sudo apt-get update
                sudo apt-get install trivy
                mkdir reports/
                cd server
                docker build . -t nettu-meet-server:latest -f Dockerfile
                trivy image --format cyclonedx --output reports/sbom_frontend.json nettu-meet-server:latest
                trivy sbom reports/sbom_server.json
                cd ../frontend
                docker build . -t nettu-meet-frontend:latest -f docker/Dockerfile
                trivy image --format cyclonedx --output reports/sbom_frontend.json nettu-meet-frontend:latest
                trivy sbom reports/sbom_frontend.json
                '''
              }
              archiveArtifacts artifacts: 'reports/*', allowEmptyArchive: true
          }
      }
  }
}
