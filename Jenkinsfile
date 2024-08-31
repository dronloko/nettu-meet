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
              archiveArtifacts artifacts: 'reports/*', allowEmptyArchive: true
              stash name: 'reports/semgrep.json', includes: "semgrep"
            }
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
                trivy fs --format cyclondx -o ../reports/sbom.json package-lock.json
                #docker build . -t nettu-meet:latest -f Dockerfile
                #trivy image --format cyclonedx -o ../reports/sbom.json nettu-meet-server:latest
                trivy sbom -f json -o ../reports/trivy.json ../reports/sbom.json
                '''
                archiveArtifacts artifacts: 'reports/*', allowEmptyArchive: true
                stash includes: 'reports/sbom*.json', name: 'sbom'
              }
          }
      }
      stage ('deptrack') {
        agent { label "dind" }
        steps {
          unstash "sbom"
          sh '''
          ls -R .
          '''
        }
      }
      stage ('owasp zap') {
        agent { label "dind" }
        steps {
          sh '''
          mkdir reports/
          curl -L -o ZAP_2.15.0_Linux.tar.gz https://github.com/zaproxy/zaproxy/releases/download/v2.15.0/ZAP_2.15.0_Linux.tar.gz
          tar xzf ZAP_2.15.0_Linux.tar.gz          
          ZAP_2.15.0/zap.sh -cmd -quickurl https://s410-exam.cyber-ed.space:8084 -quickout reports/owaspzap.json
          '''
          archiveArtifacts artifacts: 'reports/*', allowEmptyArchive: true
          stash includes: 'reports/owaspzap.json', name: 'owaspzap'
        }
      }
  }
}
