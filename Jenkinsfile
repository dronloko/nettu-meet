pipeline {
  agent any
  environment {
    DEPTRACK_URL="https://s410-exam.cyber-ed.space:8081"
    DEPTRACK_API_KEY="odt_SfCq7Csub3peq7Y6lSlQy5Ngp9sSYpJl"
    DOJO_URL="https://s410-exam.cyber-ed.space:8083"
    DOJO_API_TOKEN="c5b50032ffd2e0aa02e2ff56ac23f0e350af75b4"    
  }
    stages {
      stage ('semgrep') {
        agent { label "alpine" }
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
              archiveArtifacts artifacts: 'reports/semgrep.json', allowEmptyArchive: true
              stash includes: 'reports/semgrep.json', name: 'semgrep-report'
            }
          }
        }
      }
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
      stage ('owasp zap') {
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
      }
      /*stage ('defect dojo') {
        steps {
          //unstash "semgrep-report"
          unstash "trivy-report"
          //unstash "owaspzap-report
          sh '''
          cd reports/
          #curl -X "POST" -kL "${DOJO_URL}/api/v2/import-scan/" -H "accept: application/json" -H "Authorization: Token ${DOJO_API_TOKEN}" -H "Content-Type: multipart/form-data" -F "active=true" -F "verified=true" -F "deduplication_on_engagement=true" -F "minimum_severity=High" -F "scan_date=2024-08-31" -F "engagement_end_date=2024-08-31" -F "group_by=component_name" -F "tags=" -F "product_name=dronloko" -F "file=@semgrep.json;type=application/json" -F "auto_create_context=true" -F "scan_type=Semgrep JSON Report" -F "engagement=1"
          #curl -X "POST" -kL "${DOJO_URL}/api/v2/import-scan/" -H "accept: application/json" -H "Authorization: Token ${DOJO_API_TOKEN}" -H "Content-Type: multipart/form-data" -F "active=true" -F "verified=true" -F "deduplication_on_engagement=true" -F "minimum_severity=High" -F "scan_date=2024-08-31" -F "engagement_end_date=2024-08-31" -F "group_by=component_name" -F "tags=" -F "product_name=dronloko" -F "file=@trivy.json;type=application/json" -F "auto_create_context=true" -F "scan_type=Trivy Scan" -F "engagement=2"
          #curl -X "POST" -kL "${DOJO_URL}/api/v2/import-scan/" -H "accept: application/json" -H "Authorization: Token ${DOJO_API_TOKEN}" -H "Content-Type: multipart/form-data" -F "active=true" -F "verified=true" -F "deduplication_on_engagement=true" -F "minimum_severity=High" -F "scan_date=2024-08-31" -F "engagement_end_date=2024-08-31" -F "group_by=component_name" -F "tags=" -F "product_name=dronloko" -F "file=@owaspzap.json;type=application/json" -F "auto_create_context=true" -F "scan_type=ZAP Scan" -F "engagement=3"
          curl -k -X POST "${DOJO_URL}/api/v2/import-scan/" -H Authorization:"Token ${DOJO_API_TOKEN}" -H "Content-Type:multipart/form-data" -H "accept:application/json" -F scan_type="Trivy Scan" -F "file=@trivy.json;type=application/json" -F "engagement=2" -F "product_name=dronloko"
          '''
        }
      }*/
      stage ('quality gate') {
          steps {
            unstash "semgrep-report"
            unstash "trivy-report"
            unstash "owaspzap-report"
            
            sh '''
              cd reports/
              e=$(cat semgrep.json | jq | grep -iE '"severity": "ERROR"' | wc -l)
              w=$(cat semgrep.json | jq | grep -iE '"severity": "WARNING"' | wc -l)
              echo "Semgrep: Found $e errors"
              echo "Semgrep: Found $e warnings"
              if [ $e -ge 2 ] || [ $w -gt 10 ]; then
                echo "Semgrep QualityGate failed"
                #exit "Semgrep QualityGate failed"
              fi
              c=$(cat trivy.json | jq | grep -iE "\"severity\": \"CRITICAL" | wc -l )
              h=$(cat trivy.json | jq | grep -iE "\"severity\": \"HIGH" | wc -l)
              m=$(cat trivy.json | jq | grep -iE "\"severity\": \"MEDIUM" | wc -l)
              l=$(cat trivy.json | jq | grep -iE "\"severity\": \"LOW" | wc -l)
              echo "trivy: Found $c critical severity findings"
              echo "trivy: Found $h high severity findings"
              echo "trivy: Found $m medium severity findings"
              echo "trivy: Found $l low severity findings"
              if [ $c -ge 1 ] || [ $h -ge 5 ] || [ $m -ge 10] || [ $l -ge 15]; then
                echo "Trivy QualityGate failed"
                #exit("Trivy QualityGate failed")
              fi
              c=$(cat owaspzap.json | jq | grep -E "\"riskdesc\": \"Critical" | wc -l )
              h=$(cat owaspzap.json | jq | grep -E "\"riskdesc\": \"High" | wc -l)
              m=$(cat owaspzap.json | jq | grep -E "\"riskdesc\": \"Medium" | wc -l)
              l=$(cat owaspzap.json | jq | grep -E "\"riskdesc\": \"Low" | wc -l)
              if [ $c -ge 1 ] || [ $h -ge 10 ] || [ $m -ge 9 ] || [ $l -ge 7 ]; then
                echo "OWASP ZAP QualityGate failed"
                #exit("OWASP ZAP QualityGate failed")
              fi
            '''
          }
      }
  }
}
