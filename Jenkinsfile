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
                curl -fsSL https://deb.nodesource.com/setup_22.x -o nodesource_setup.sh
                sudo -E bash nodesource_setup.sh
                sudo apt-get install -y nodejs
                npm -v
                wget -qO - https://aquasecurity.github.io/trivy-repo/deb/public.key | sudo apt-key add -
                echo deb https://aquasecurity.github.io/trivy-repo/deb $(lsb_release -sc) main | sudo tee -a /etc/apt/sources.list.d/trivy.list
                sudo apt-get update
                sudo apt-get install trivy
                mkdir reports/
                cd server
                docker build . -t nettu-meet-server:latest -f Dockerfile
                trivy image --format cyclonedx --output sbom_server.json nettu-meet-server:latest
                #trivy sbom sbom_server.json
                npm build
                cd ../frontend
                npm help
                #docker build . -t nettu-meet-frontend:latest -f docker/Dockerfile
                #trivy image --format cyclonedx --output sbom_frontend.json nettu-meet-frontend:latest
                #trivy sbom sbom_frontend.json
                '''
              }
              archiveArtifacts artifacts: '*.json', allowEmptyArchive: true
          }
      }
  }
}
