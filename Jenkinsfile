pipeline {
    
    environment {
imagename = "tsaha/spring-hello"
registryCredential = 'hub.docker'
dockerImage = ''
}
    
  agent {
    kubernetes {
      label 'springBoot'
      defaultContainer 'jnlp'
      yaml """
apiVersion: v1
kind: Pod
metadata:
labels:
  component: ci
spec:
  # Use service account that can deploy to all namespaces
  serviceAccountName: jenkins
  containers:
  - name: maven
    image: maven:latest
    command:
    - cat
    tty: true
    volumeMounts:
    - mountPath: /root/.m2
      name: m2
    
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
    - name: m2
      hostPath:
        path: /tmp/m2    
    
"""
}
   }
  stages {
    stage('Get a Maven Project'){
        steps{
                git branch: 'master', url: 'https://github.com/spring-guides/gs-spring-boot-docker.git'
        }
    }
    stage('Maven Build') {
      steps {
        container('maven') {
          sh """
                        cd ./complete
                        mvn clean install -DskipTests
                        mkdir -p target/dependency && (cd target/dependency; jar -xf ../*.jar)
                                                """
        }
      }
    }
    
    stage('Docker Build') {
      steps {
        container('docker') {
        withCredentials([[$class: 'UsernamePasswordMultiBinding',
          credentialsId: 'hub.docker',
          usernameVariable: 'DOCKER_HUB_USER',
          passwordVariable: 'DOCKER_HUB_PASSWORD']]) {
          sh """
            docker login -u ${DOCKER_HUB_USER} -p ${DOCKER_HUB_PASSWORD}
            cd ./complete
            docker build  -t $imagename:$BUILD_NUMBER .
            docker push  $imagename:$BUILD_NUMBER
            """
        }
      }
        
      }
    }
  }
}
