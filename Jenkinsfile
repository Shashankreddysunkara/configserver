pipeline {
  agent {
    kubernetes {
        defaultContainer 'gradle'
        yaml '''
        apiVersion: v1
        kind: Pod
        metadata:
          labels:
            cicd: gradle-docker
        spec:
          containers:
          - name: gradle
            image: gradle:7.2.0-jdk11
            command:
            - cat
            tty: true
            resources:
              limits:
                cpu: 2
                memory: 4Gi
              requests:
                cpu: 2
                memory: 4Gi
            volumeMounts:
            - name: build-cache
              mountPath: /tmp
          - name: docker
            image: docker:19.03.1-dind
            securityContext:
              privileged: true
          volumes:
          - name: build-cache
            hostPath:
            path: /tmp
        '''
    }
  }
  options {
    timestamps()
    ansiColor('xterm')
    buildDiscarder(logRotator(numToKeepStr: '20', artifactNumToKeepStr: '20'))
    timeout(time: 60, unit: 'MINUTES')
    disableConcurrentBuilds()
  }
  environment {
    GITHUB = credentials('github-http-user-token')
    SONAR_TOKEN = credentials('sonarcloud-token')
  }
  stages {
    stage('Git Checkout') {
      steps {
        git url: 'https://github.com/ggnanasekaran77/configserver.git', branch: 'develop'
      }      
    }

    stage('App Build') {
      steps {
        sh 'mkdir -p /tmp/cache'  
        sh "gradle clean build -g /tmp/cache jacocoTestReport sonarqube -Dsonar.branch.name=${env.GIT_BRANCH}"
      }
    }

    stage('Docker Build') {
      steps {
        container ('docker') {
          script {
            docker.withRegistry('', 'docker-hub-token') {
              def customImage = docker.build("ggnanasekaran77/configserver:${GIT_COMMIT}")
              customImage.push()
            }
          }
        }
      }
    }

    stage('Deploy') {
      steps {
        sh "git clone https://$GITHUB_USR:$GITHUB_PSW@github.com/ggnanasekaran77/kube-demoapp.git"
        dir ("kube-demoapp") {
          sh """
            git config --global user.name 'Gnanasekaran Gajendiran'
            git config --global user.email 'ggnanasekaran77@gmail.com'
            sed -i "s#ggnanasekaran77/configserver:.*#ggnanasekaran77/configserver:${env.GIT_COMMIT}#" kube/deployment.yaml
            cat kube/deployment.yaml
            git commit -am 'Publish new version' && git push || echo 'no changes'
          """
        }
      }
    }
  }
  post {
    always {
      script {
        publishToElastic("multibranchPipeline", "argocd", "configserver", "configserver", "stg")
      }
      cleanWs()
      dir("${env.WORKSPACE}@tmp") {
        deleteDir()
      }
      dir("${env.WORKSPACE}@script") {
        deleteDir()
      }
      dir("${env.WORKSPACE}@script@tmp") {
        deleteDir()
      }
    }
  }
}
