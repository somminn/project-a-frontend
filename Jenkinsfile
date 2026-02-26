pipeline {
  agent {
    kubernetes {
      defaultContainer 'jnlp'
      yaml """
apiVersion: v1
kind: Pod
spec:
  serviceAccountName: jenkins
  containers:
    - name: kaniko
      image: gcr.io/kaniko-project/executor:latest
      command: ["cat"]
      tty: true
      volumeMounts:
        - name: docker-config
          mountPath: /kaniko/.docker
    - name: git
      image: alpine/git:2.45.2
      command: ["cat"]
      tty: true
    - name: yq
      image: mikefarah/yq:4.44.3
      command: ["cat"]
      tty: true
  volumes:
    - name: docker-config
      secret:
        secretName: dockerhub-config
"""
    }
  }

  options {
    disableConcurrentBuilds()
  }

  environment {
    DOCKER_IMAGE   = "cowmin/project-a-frontend"
    GITOPS_REPO    = "git@github.com:somminn/project-a-gitops.git"
    GITOPS_FILE    = "app/frontend/kustomization.yml"
    GIT_USER_NAME  = "jenkins"
    GIT_USER_EMAIL = "jenkins@local"
  }

  stages {
    stage('Checkout App Repo') {
      steps {
        checkout scm
        script {
          env.IMAGE_TAG = sh(script: "git rev-parse HEAD", returnStdout: true).trim()
          echo "IMAGE_TAG=${env.IMAGE_TAG}"
        }
      }
    }

    stage('Build & Push (Kaniko)') {
      steps {
        container('kaniko') {
          sh '''
            set -eu
            /kaniko/executor \
              --context="${WORKSPACE}" \
              --dockerfile="${WORKSPACE}/Dockerfile" \
              --destination="${DOCKER_IMAGE}:${IMAGE_TAG}" \
              --cache=true \
              --cache-ttl=168h
          '''
        }
      }
    }

    stage('Update GitOps Tag') {
      steps {
        container('git') {
          withCredentials([sshUserPrivateKey(credentialsId: 'GITOPS_DEPLOY_KEY', keyFileVariable: 'SSH_KEY')]) {
            sh '''
              set -eu
              mkdir -p ~/.ssh
              cp "$SSH_KEY" ~/.ssh/id_rsa
              chmod 600 ~/.ssh/id_rsa
              ssh-keyscan -t rsa github.com >> ~/.ssh/known_hosts

              rm -rf gitops
              git clone "${GITOPS_REPO}" gitops
            '''
          }
        }

        container('yq') {
          sh '''
            set -eu
            cd gitops

            yq -i '
              (.images[] | select(.name == strenv(DOCKER_IMAGE)) | .newTag) = strenv(IMAGE_TAG)
            ' "${GITOPS_FILE}"

            yq '.images' "${GITOPS_FILE}"
          '''
        }

        container('git') {
          withCredentials([sshUserPrivateKey(credentialsId: 'gitops-deploy-key', keyFileVariable: 'SSH_KEY')]) {
            sh '''
              set -eu
              cd gitops
              git config user.name "${GIT_USER_NAME}"
              git config user.email "${GIT_USER_EMAIL}"

              git add "${GITOPS_FILE}"
              git diff --cached --quiet && echo "No changes" && exit 0

              git commit -m "chore(gitops): update frontend image tag to ${IMAGE_TAG}"
              GIT_SSH_COMMAND='ssh -i ~/.ssh/id_rsa -o StrictHostKeyChecking=yes' git push
            '''
          }
        }
      }
    }
  }
}