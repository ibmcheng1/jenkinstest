def gitCommit
def volumes = [ hostPathVolume(hostPath: '/var/run/docker.sock', mountPath: '/var/run/docker.sock') ]
volumes += secretVolume(secretName: 'jenkins-docker-sec', mountPath: '/jenkins_docker_sec')
podTemplate(label: 'icp-liberty-build-jenkinstest', slaveConnectTimeout: 600,
    containers: [
        containerTemplate(name: 'jnlp', image: 'mycluster.icp:8500/default/jenkins/jnlp-slave:3.23-1'),
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
 
    }
}
