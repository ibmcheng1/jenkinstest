def gitCommit
def volumes = [ hostPathVolume(hostPath: '/var/run/docker.sock', mountPath: '/var/run/docker.sock') ]
volumes += secretVolume(secretName: 'jenkins-docker-sec', mountPath: '/jenkins_docker_sec')
podTemplate(label: 'icp-liberty-build-jenkinstest', slaveConnectTimeout: 600,
    containers: [
        containerTemplate(name: 'jnlp', image: 'mycluster.icp:8500/default/jenkins/jnlp-slave:3.23-1'),
        containerTemplate(name: 'maven', image: 'mycluster.icp:8500/default/maven:3.5.4-jdk-8', ttyEnabled: true, command: 'cat'),
        containerTemplate(name: 'docker', image: 'mycluster.icp:8500/default/docker:17.12', ttyEnabled: true, command: 'cat'),
        containerTemplate(name: 'kubectl', image: 'mycluster.icp:8500/default/ibmcom/k8s-kubectl:v1.8.3', ttyEnabled: true, command: 'cat'),       
    ],
    volumes: volumes
)
{
    node ('icp-liberty-build-jenkinstest') {
         //def gitCommit
        stage ('Extract') {
          checkout scm
          gitCommit = sh(script: 'git rev-parse --short HEAD', returnStdout: true).trim()
          echo "checked out git commit ${gitCommit}"
        }
        stage ('maven build') {
          container('maven') {
            sh '''
            echo "maven build ... "
            # mvn clean test install -Dhttp.proxyHost=172.21.254.254 -Dhttp.proxyPort=3128 -Dhttps.proxyHost=172.21.254.254 -Dhttps.proxyPort=3128 -Dhttp.nonProxyHosts=kubernetes.default           
            # mvn clean install -Dhttp.proxyHost=172.21.254.254 -Dhttp.proxyPort=3128 -Dhttps.proxyHost=172.21.254.254 -Dhttps.proxyPort=3128 -Dhttp.nonProxyHosts=kubernetes.default
            '''
          }
        }
          stage ('docker') {
          container('docker') {
            def imageTag = "mycluster.icp:8500/default/jenkinstest:${gitCommit}"
            echo "imageTag ${imageTag}"
          }
        }       

        stage ('deploy') {
          container('kubectl') {
            def imageTag = null
            imageTag = "fa84914"
            sh """
            #!/bin/bash
            pwd
            ls -l
            echo "Describe deployment"
            kubectl describe deployment jenkinstest-deployment --namespace jenkinstest
            echo "finished"
            """
          }
        }
    }
}
