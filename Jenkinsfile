pipeline {
  agent any

  tools {
    jdk 'jdk21'
    maven 'maven3'
  }

  parameters {
    booleanParam(
      name: 'DEPLOY_TO_VM_TOMCAT',
      defaultValue: false,
      description: 'Deploy WAR to VM-based Tomcat (8080)'
    )
    booleanParam(
      name: 'DEPLOY_TO_DOCKER',
      defaultValue: false,
      description: 'Build and deploy to Docker container using direct Docker commands (8082)'
    )
    booleanParam(
      name: 'DEPLOY_WITH_ANSIBLE',
      defaultValue: true,
      description: 'Deploy to Docker using Ansible automation (8082) - RECOMMENDED'
    )
  }

  environment {
    // VM Tomcat deploy settings
    DEPLOY_DIR = '/opt/jenkins-deploy'
    WAR_NAME   = 'ABCtechnologies-1.0.war'
    APP_URL_VM = 'http://localhost:8080/ABCtechnologies-1.0/'

    // Docker deploy settings (direct)
    DOCKER_IMAGE     = "igp1-app:${BUILD_NUMBER}"
    DOCKER_CONTAINER = 'igp1-app'
    DOCKER_HOST_PORT = '8082'
    DOCKER_CONT_PORT = '8080'
    APP_URL_DOCKER   = "http://localhost:${DOCKER_HOST_PORT}/"
    
    // Ansible settings
    ANSIBLE_PLAYBOOK = '/opt/ansible-igp1/deploy-docker.yml'
    ANSIBLE_INVENTORY = '/opt/ansible-igp1/inventory.ini'
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
    // VM Tomcat Deploy (existing)
    // -------------------------
    stage('Stage WAR for Deploy') {
      when { expression { params.DEPLOY_TO_VM_TOMCAT } }
      steps {
        sh """
          mkdir -p "${DEPLOY_DIR}"
          cp -f "target/${WAR_NAME}" "${DEPLOY_DIR}/${WAR_NAME}"
        """
      }
    }

    stage('Deploy to VM Tomcat') {
      when { expression { params.DEPLOY_TO_VM_TOMCAT } }
      steps {
        sh 'sudo -n /usr/local/bin/deploy_igp1.sh'
      }
    }

    stage('Health Check - VM Tomcat') {
      when { expression { params.DEPLOY_TO_VM_TOMCAT } }
      steps {
        sh """
          for i in \$(seq 1 12); do
            CODE=\$(curl -s -o /dev/null -w "%{http_code}" "${APP_URL_VM}" || true)
            if [ "\$CODE" = "200" ]; then
              echo "VM Tomcat health check OK: ${APP_URL_VM}"
              exit 0
            fi
            echo "VM Tomcat not ready yet (HTTP \$CODE). Waiting..."
            sleep 5
          done
          echo "VM Tomcat health check FAILED: ${APP_URL_VM}"
          exit 1
        """
      }
    }

    // -------------------------
    // Docker Direct Deploy (Phase 3)
    // -------------------------
    stage('Docker - Build Image') {
      when { expression { params.DEPLOY_TO_DOCKER } }
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
      when { expression { params.DEPLOY_TO_DOCKER } }
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

    stage('Health Check - Docker') {
      when { expression { params.DEPLOY_TO_DOCKER } }
      steps {
        sh """
          for i in \$(seq 1 18); do
            CODE=\$(curl -s -o /dev/null -w "%{http_code}" "${APP_URL_DOCKER}" || true)
            if [ "\$CODE" = "200" ]; then
              echo "Docker health check OK: ${APP_URL_DOCKER}"
              exit 0
            fi
            echo "Docker app not ready yet (HTTP \$CODE). Waiting..."
            sleep 5
          done

          echo "Docker health check FAILED: ${APP_URL_DOCKER}"
          echo "Last 200 lines of container logs:"
          docker logs --tail 200 "${DOCKER_CONTAINER}" || true
          exit 1
        """
      }
    }

    // -------------------------
    // Ansible Deploy (Phase 4 - NEW!)
    // -------------------------
    stage('Deploy with Ansible') {
      when { expression { params.DEPLOY_WITH_ANSIBLE } }
      steps {
        script {
          echo "Deploying using Ansible playbook..."
          sh """
            ansible-playbook ${ANSIBLE_PLAYBOOK} \\
              -i ${ANSIBLE_INVENTORY} \\
              -e "build_number=${BUILD_NUMBER}" \\
              -e "project_directory=${WORKSPACE}"
          """
        }
      }
    }
  }

  post {
    always {
      junit 'target/surefire-reports/*.xml'
    }
    success {
      echo "✓ Build and deployment successful!"
      script {
        if (params.DEPLOY_WITH_ANSIBLE) {
          echo "✓ Deployed via Ansible automation"
        } else if (params.DEPLOY_TO_DOCKER) {
          echo "✓ Deployed via direct Docker commands"
        } else if (params.DEPLOY_TO_VM_TOMCAT) {
          echo "✓ Deployed to VM Tomcat"
        }
      }
    }
    failure {
      echo "✗ Build or deployment failed. Check logs above."
    }
  }
}
