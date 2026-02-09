pipeline {
  agent any

  tools {
    jdk 'jdk21'
    maven 'maven3'
  }

  parameters {
    choice(
      name: 'DEPLOYMENT_METHOD',
      choices: ['kubernetes', 'ansible', 'docker-direct', 'vm-tomcat', 'none'],
      description: 'Choose deployment method: kubernetes (Phase 5), ansible (Phase 4), docker-direct, vm-tomcat, or none'
    )
  }

  environment {
    // VM Tomcat deploy settings
    DEPLOY_DIR = '/opt/jenkins-deploy'
    WAR_NAME   = 'ABCtechnologies-1.0.war'
    APP_URL_VM = 'http://localhost:8080/ABCtechnologies-1.0/'

    // Docker deploy settings
    DOCKER_IMAGE     = "igp1-app:${BUILD_NUMBER}"
    DOCKER_CONTAINER = 'igp1-app'
    DOCKER_HOST_PORT = '8082'
    DOCKER_CONT_PORT = '8080'
    APP_URL_DOCKER   = "http://localhost:${DOCKER_HOST_PORT}/"

    // Ansible settings
    ANSIBLE_PLAYBOOK_DOCKER = '/opt/ansible-igp1/deploy-docker.yml'
    ANSIBLE_PLAYBOOK_K8S    = '/opt/ansible-igp1/deploy-k8s.yml'
    ANSIBLE_INVENTORY       = '/opt/ansible-igp1/inventory.ini'
    
    // Kubernetes settings
    K8S_NODEPORT = '30080'
    APP_URL_K8S  = "http://localhost:${K8S_NODEPORT}/"
  }

  stages {
    stage('Checkout') {
      steps {
        checkout([$class: 'GitSCM',
          branches: [[name: '*/main']],
          userRemoteConfigs: [[
            url: 'git@github.com:Gats-bey/Industrial-Grade-Project-1.git',
            credentialsId: 'igp1-github-ssh'
          ]]
        ])
      }
    }

    stage('Build & Test') {
      steps {
        sh 'mvn -B clean test'
      }
    }

    stage('Package WAR') {
      steps {
        sh 'mvn -B package -DskipTests'
      }
    }

    stage('Archive Artifact') {
      steps {
        archiveArtifacts artifacts: 'target/*.war', fingerprint: true
      }
    }

    // -------------------------
    // Deployment Method Selection
    // -------------------------
    stage('Display Deployment Method') {
      steps {
        script {
          echo "═══════════════════════════════════════════════"
          echo "Selected Deployment Method: ${params.DEPLOYMENT_METHOD}"
          echo "═══════════════════════════════════════════════"
        }
      }
    }

    // -------------------------
    // VM Tomcat Deploy
    // -------------------------
    stage('Stage WAR for Deploy') {
      when { expression { params.DEPLOYMENT_METHOD == 'vm-tomcat' } }
      steps {
        sh """
          mkdir -p "${DEPLOY_DIR}"
          cp -f "target/${WAR_NAME}" "${DEPLOY_DIR}/${WAR_NAME}"
        """
      }
    }

    stage('Deploy to VM Tomcat') {
      when { expression { params.DEPLOYMENT_METHOD == 'vm-tomcat' } }
      steps {
        sh 'sudo -n /usr/local/bin/deploy_igp1.sh'
      }
    }

    stage('Health Check - VM Tomcat') {
      when { expression { params.DEPLOYMENT_METHOD == 'vm-tomcat' } }
      steps {
        sh """
          for i in \$(seq 1 12); do
            CODE=\$(curl -s -o /dev/null -w "%{http_code}" "${APP_URL_VM}" || true)
            if [ "\$CODE" = "200" ]; then
              echo "✓ VM Tomcat health check OK: ${APP_URL_VM}"
              exit 0
            fi
            echo "VM Tomcat not ready yet (HTTP \$CODE). Waiting..."
            sleep 5
          done
          echo "✗ VM Tomcat health check FAILED: ${APP_URL_VM}"
          exit 1
        """
      }
    }

    // -------------------------
    // Docker Direct Deploy
    // -------------------------
    stage('Docker - Build Image') {
      when { expression { params.DEPLOYMENT_METHOD == 'docker-direct' } }
      steps {
        sh """
          test -f Dockerfile
          test -f target/${WAR_NAME}

          echo "Building image: ${DOCKER_IMAGE}"
          docker build -t "${DOCKER_IMAGE}" .
        """
      }
    }

    stage('Docker - Run Container') {
      when { expression { params.DEPLOYMENT_METHOD == 'docker-direct' } }
      steps {
        sh """
          echo "Stopping/removing any existing container named ${DOCKER_CONTAINER}..."
          docker rm -f "${DOCKER_CONTAINER}" >/dev/null 2>&1 || true

          echo "Starting container ${DOCKER_CONTAINER} on port ${DOCKER_HOST_PORT} -> ${DOCKER_CONT_PORT}"
          docker run -d --name "${DOCKER_CONTAINER}" \\
            -p "${DOCKER_HOST_PORT}:${DOCKER_CONT_PORT}" \\
            "${DOCKER_IMAGE}"

          docker ps --filter "name=${DOCKER_CONTAINER}"
        """
      }
    }

    stage('Health Check - Docker Direct') {
      when { expression { params.DEPLOYMENT_METHOD == 'docker-direct' } }
      steps {
        sh """
          for i in \$(seq 1 18); do
            CODE=\$(curl -s -o /dev/null -w "%{http_code}" "${APP_URL_DOCKER}" || true)
            if [ "\$CODE" = "200" ]; then
              echo "✓ Docker health check OK: ${APP_URL_DOCKER}"
              exit 0
            fi
            echo "Docker app not ready yet (HTTP \$CODE). Waiting..."
            sleep 5
          done

          echo "✗ Docker health check FAILED: ${APP_URL_DOCKER}"
          echo "Last 200 lines of container logs:"
          docker logs --tail 200 "${DOCKER_CONTAINER}" || true
          exit 1
        """
      }
    }

    // -------------------------
    // Ansible Docker Deploy
    // -------------------------
    stage('Deploy with Ansible - Docker') {
      when { expression { params.DEPLOYMENT_METHOD == 'ansible' } }
      steps {
        script {
          echo "═══════════════════════════════════════════════"
          echo "Deploying to Docker using Ansible automation"
          echo "═══════════════════════════════════════════════"
        }
        sh """
          ansible-playbook "${ANSIBLE_PLAYBOOK_DOCKER}" \\
            -i "${ANSIBLE_INVENTORY}" \\
            -e "build_number=${BUILD_NUMBER}" \\
            -e "project_directory=${WORKSPACE}"
        """
      }
    }

    stage('Health Check - Ansible Docker') {
      when { expression { params.DEPLOYMENT_METHOD == 'ansible' } }
      steps {
        sh """
          echo "[INFO] Checking app health after Ansible deployment: ${APP_URL_DOCKER}"

          for i in \$(seq 1 18); do
            CODE=\$(curl -s -o /dev/null -w "%{http_code}" "${APP_URL_DOCKER}" || true)
            echo "[INFO] Attempt \$i: HTTP \$CODE"

            if [ "\$CODE" = "200" ]; then
              echo "✓ Ansible Docker deployment health check OK: ${APP_URL_DOCKER}"
              exit 0
            fi

            sleep 5
          done

          echo "✗ Ansible Docker deployment health check FAILED: ${APP_URL_DOCKER}"
          echo "Last 200 lines of container logs:"
          docker logs --tail 200 "${DOCKER_CONTAINER}" || true
          exit 1
        """
      }
    }

    // -------------------------
    // Kubernetes Deploy (Phase 5 - NEW!)
    // -------------------------
    stage('Deploy to Kubernetes with Ansible') {
      when { expression { params.DEPLOYMENT_METHOD == 'kubernetes' } }
      steps {
        script {
          echo "═══════════════════════════════════════════════"
          echo "Deploying to Kubernetes using Ansible"
          echo "Phase 5: Container Orchestration"
          echo "═══════════════════════════════════════════════"
        }
        sh """
          ansible-playbook "${ANSIBLE_PLAYBOOK_K8S}" \\
            -i "${ANSIBLE_INVENTORY}" \\
            -e "build_number=${BUILD_NUMBER}" \\
            -e "project_directory=${WORKSPACE}"
        """
      }
    }

    stage('Health Check - Kubernetes') {
      when { expression { params.DEPLOYMENT_METHOD == 'kubernetes' } }
      steps {
        sh """
          echo "[INFO] Checking Kubernetes deployment health: ${APP_URL_K8S}"

          for i in \$(seq 1 30); do
            CODE=\$(curl -s -o /dev/null -w "%{http_code}" "${APP_URL_K8S}" || true)
            echo "[INFO] Attempt \$i: HTTP \$CODE"

            if [ "\$CODE" = "200" ]; then
              echo "✓ Kubernetes deployment health check OK: ${APP_URL_K8S}"
              kubectl get pods -l app=igp1-app
              exit 0
            fi

            sleep 5
          done

          echo "✗ Kubernetes deployment health check FAILED: ${APP_URL_K8S}"
          echo "Pod status:"
          kubectl get pods -l app=igp1-app
          echo "Pod logs:"
          kubectl logs -l app=igp1-app --tail=100
          exit 1
        """
      }
    }
  }

  post {
    always {
      junit 'target/surefire-reports/*.xml'
    }
    success {
      script {
        echo "═══════════════════════════════════════════════"
        echo "✓ BUILD AND DEPLOYMENT SUCCESSFUL!"
        echo "═══════════════════════════════════════════════"
        if (params.DEPLOYMENT_METHOD == 'kubernetes') {
          echo "✓ Deployed to Kubernetes (Phase 5)"
          echo "✓ Application URL: ${APP_URL_K8S}"
          echo "✓ Access via NodePort 30080"
        } else if (params.DEPLOYMENT_METHOD == 'ansible') {
          echo "✓ Deployed via Ansible automation to Docker"
          echo "✓ Application URL: ${APP_URL_DOCKER}"
        } else if (params.DEPLOYMENT_METHOD == 'docker-direct') {
          echo "✓ Deployed via direct Docker commands"
          echo "✓ Application URL: ${APP_URL_DOCKER}"
        } else if (params.DEPLOYMENT_METHOD == 'vm-tomcat') {
          echo "✓ Deployed to VM Tomcat"
          echo "✓ Application URL: ${APP_URL_VM}"
        } else {
          echo "✓ Build completed successfully (no deployment)"
        }
        echo "═══════════════════════════════════════════════"
      }
    }
    failure {
      echo "✗ Build or deployment failed. Check logs above."
    }
  }
}
