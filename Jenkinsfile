pipeline {
  agent {
    kubernetes {
      yaml '''
        apiVersion: v1
        kind: Pod
        spec:
          containers:
          - name: git
            image: alpine/git:latest
            command:
            - cat
            tty: true
          - name: docker
            image: docker:latest
            command:
            - cat
            tty: true
            volumeMounts:
             - mountPath: /var/run/docker.sock
               name: docker-sock
          volumes:
          - name: docker-sock
            hostPath:
              path: /var/run/docker.sock    
        '''
    }
  }

				environment {

				DOCKER_HUB_CREDENTIALS = credentials('DEVSDS_DOCKERHUB')

				}

 stages {
    stage('Clone') {
      steps {
        container('git') {
          git branch: 'master', changelog: false, poll: false, url: 'https://github.com/kalpitrcc/MLOPS.git'
        }
      }
    }   
    stage('Build-Docker-Image') {
      steps {
        container('docker') {
          sh 'docker build -t devsds/modeltraining:$BUILD_NUMBER .'
        }
      }
    }
    stage('Login-Into-Docker') {
      steps {
        container('docker') {
          sh 'echo $DOCKER_HUB_CREDENTIALS_PSW | docker login -u $DOCKER_HUB_CREDENTIALS_USR --password-stdin'
      
      }
    }
    }
     stage('Push-Image-to-DockerHub') {
      steps {
        container('docker') {
          sh 'docker push devsds/modeltraining:$BUILD_NUMBER'
      }
    }
   }
   stage('Pre-processing-test') {
      agent {
        kubernetes {
          yaml """
        apiVersion: v1
        kind: Pod
        spec:
          containers:
          - name: modeltraining
            image: "devsds/modeltraining:${BUILD_NUMBER}"
            command:
            - cat
            tty: true
        """.stripIndent()
        }
      }
      
      steps {
        container('modeltraining') {
	  sh 'python3 --version'
	  sh 'python --version'	
          sh 'chmod +x preprocessing.py'
	  sh 'pwd'
	  sh 'python3 preprocessing.py'
	  sh 'ls -ltr'
	  sh 'date'
        }
      }
    }
  }
    post {
      always {
        container('docker') {
          sh 'docker images'
        }
      }
    }
}
