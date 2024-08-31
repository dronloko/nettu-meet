pipeline {
  agent any
  environment {
    DEPTRACK_URL="https://s410-exam.cyber-ed.space:8081"
    DEPTRACK_API_KEY="odt_SfCq7Csub3peq7Y6lSlQy5Ngp9sSYpJl"
    DOJO_URL="https://s410-exam.cyber-ed.space:8083"
    DOJO_API_TOKEN="c5b50032ffd2e0aa02e2ff56ac23f0e350af75b4"    
  }
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
              stash name: 'reports/semgrep.json', includes: "semgrep-report"
            }
          }
        }
      }*/
      stage ('trivy') {
          agent { label "dind" }
          steps {
            script {
                sh '''
                wget -qO - https://aquasecurity.github.io/trivy-repo/deb/public.key | sudo apt-key add -
                echo deb https://aquasecurity.github.io/trivy-repo/deb $(lsb_release -sc) main | sudo tee -a /etc/apt/sources.list.d/trivy.list
                sudo apt update
                sudo apt install trivy
                mkdir reports/
                cd server
                trivy fs --format cyclonedx -o ../reports/sbom.json package-lock.json
                #docker build . -t nettu-meet:latest -f Dockerfile
                #trivy image --format cyclonedx -o ../reports/sbom.json nettu-meet-server:latest
                trivy sbom -f json -o ../reports/trivy.json ../reports/sbom.json
                '''
                archiveArtifacts artifacts: 'reports/*', allowEmptyArchive: true
                stash includes: 'reports/sbom.json', name: 'sbom'
                stash includes: 'reports/trivy.json', name: 'trivy-report'
              }
          }
      }
      stage ('deptrack') {
        agent { label "dind" }
        steps {
          unstash "sbom"
          sh '''
          ls reports/sbom.json
          cd reports/
          response_code=$(curl -k --silent --output /dev/null --write-out %{http_code} \
          -X "POST" "${DEPTRACK_URL}/api/v1/bom" \
          -H "Content-Type:multipart/form-data" -H "X-Api-Key:${DEPTRACK_API_KEY}" \
          -F "autoCreate=true" -F "projectName=dronloko" -F "projectVersion=1.0" -F "bom=@sbom.json")
          echo $response_code
          '''
        }
      }
      /*stage ('owasp zap') {
        agent { label "dind" }
        steps {
          sh '''
          mkdir reports/
          sudo apt update
          sudo apt install -y wget openjdk-11-jre
          wget -q https://github.com/zaproxy/zaproxy/releases/download/v2.15.0/ZAP_2.15.0_Linux.tar.gz
          tar xzf ZAP_2.15.0_Linux.tar.gz        
          ZAP_2.15.0/zap.sh -cmd -addonupdate -addoninstall wappalyzer -addoninstall pscanrulesBeta
          ZAP_2.15.0/zap.sh -cmd -quickurl https://s410-exam.cyber-ed.space:8084 -quickout $(pwd)/owaspzap.json
          cp $(pwd)/owaspzap.json reports/
          '''
          archiveArtifacts artifacts: 'reports/*', allowEmptyArchive: true
          stash includes: 'reports/owaspzap.json', name: 'owaspzap-report'
        }
      }*/
      stage ('defect dojo') {
        steps {
          //unstash "semgrep-report"
          unstash "trivy-report"
          //unstash "owaspzap-report
          sh '''
          cd reports/
          #curl -X "POST" -kL "${DOJO_URL}/api/v2/import-scan/" -H "accept: application/json" -H "Authorization: Token ${DOJO_API_TOKEN}" -H "Content-Type: multipart/form-data" -F "active=true" -F "verified=true" -F "deduplication_on_engagement=true" -F "minimum_severity=High" -F "scan_date=2024-08-31" -F "engagement_end_date=2024-08-31" -F "group_by=component_name" -F "tags=" -F "product_name=dronloko" -F "file=@semgrep.json;type=application/json" -F "auto_create_context=true" -F "scan_type=Semgrep JSON Report" -F "engagement=1"
          #curl -X "POST" -kL "${DOJO_URL}/api/v2/import-scan/" -H "accept: application/json" -H "Authorization: Token ${DOJO_API_TOKEN}" -H "Content-Type: multipart/form-data" -F "active=true" -F "verified=true" -F "deduplication_on_engagement=true" -F "minimum_severity=High" -F "scan_date=2024-08-31" -F "engagement_end_date=2024-08-31" -F "group_by=component_name" -F "tags=" -F "product_name=dronloko" -F "file=@trivy.json;type=application/json" -F "auto_create_context=true" -F "scan_type=Trivy Scan" -F "engagement=2"
          #curl -X "POST" -kL "${DOJO_URL}/api/v2/import-scan/" -H "accept: application/json" -H "Authorization: Token ${DOJO_API_TOKEN}" -H "Content-Type: multipart/form-data" -F "active=true" -F "verified=true" -F "deduplication_on_engagement=true" -F "minimum_severity=High" -F "scan_date=2024-08-31" -F "engagement_end_date=2024-08-31" -F "group_by=component_name" -F "tags=" -F "product_name=dronloko" -F "file=@owaspzap.json;type=application/json" -F "auto_create_context=true" -F "scan_type=ZAP Scan" -F "engagement=3"
          curl -X POST ^
            -H Authorization:"Token ${DOJO_API_TOKEN}" ^
            -F scan_type="Trivy Scan" ^
            -F file=@trivy.json;type=application/json ^
            -F engagement=2 ^
            -H Content-Type:multipart/form-data ^
            -H accept:application/json ^
            ${DOJO_URL}/api/v2/import-scan/
          '''
        }
      }      
  }
}
