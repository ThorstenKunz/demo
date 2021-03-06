def label = "worker-${UUID.randomUUID().toString()}"

podTemplate(label: label, containers: [
  containerTemplate(name: 'maven', image: 'maven:3.5.2-jdk-8', command: 'cat', ttyEnabled: true),
  containerTemplate(name: 'docker', image: 'docker', command: 'cat', ttyEnabled: true),
  containerTemplate(name: 'kubectl', image: 'lachlanevenson/k8s-kubectl:v1.8.8', command: 'cat', ttyEnabled: true),
  containerTemplate(name: 'helm', image: 'lachlanevenson/k8s-helm:latest', command: 'cat', ttyEnabled: true)
],
volumes: [
  hostPathVolume(mountPath: '/var/run/docker.sock', hostPath: '/var/run/docker.sock'),
  nfsVolume(mountPath: '/home/jenkins/workspace/', serverAddress: '10.10.18.20', serverPath: '/k8s_prod/jenkins-agent', readOnly: false),
]) {
  node(label) {
    def myRepo = checkout scm
    def gitCommit = myRepo.GIT_COMMIT
    def gitBranch = myRepo.GIT_BRANCH
    def shortGitCommit = "${gitCommit[0..10]}"
    def previousGitCommit = sh(script: "git rev-parse ${gitCommit}~", returnStdout: true)
 
    stage('Test') {
      try {
        container('maven') {
          sh """
            pwd
            echo "GIT_BRANCH=${gitBranch}" >> /etc/environment
            echo "GIT_COMMIT=${gitCommit}" >> /etc/environment
            mvn -q -B test
            """
        }
        junit '**/target/surefire-reports/**/*.xml'
      }
      catch (exc) {
        println "Failed to test - ${currentBuild.fullDisplayName}"
        throw(exc)
      }
    }
    stage('Build') {
      container('maven') {
        sh "mvn -q -B package"
      }
    }
    stage('Create Docker images') {
      container('docker') {
        sh "docker build -t namespace/my-image:${gitCommit} ."
//        withCredentials([[$class: 'UsernamePasswordMultiBinding',
//          credentialsId: 'dockerhub',
//          usernameVariable: 'DOCKER_HUB_USER',
//          passwordVariable: 'DOCKER_HUB_PASSWORD']]) {
//          sh """
//            docker login -u ${DOCKER_HUB_USER} -p ${DOCKER_HUB_PASSWORD}
//            docker build -t namespace/my-image:${gitCommit} .
//            docker push namespace/my-image:${gitCommit}
//            """
//        }
      }
    }
    stage('Run kubectl') {
      container('kubectl') {
        sh "kubectl get pods"
      }
    }
    stage('Run helm') {
      container('helm') {
        sh "helm list"
      }
    }
  }
}
