podTemplate(label: 'maven-ose', cloud: 'openshift', containers: [
  containerTemplate(name: 'maven', image: "registry.access.redhat.com/openshift3/jenkins-slave-maven-rhel7", ttyEnabled: true, command: 'cat'),
],
volumes: [configMapVolume(configMapName: 'jenkins-maven-settings', mountPath: '/etc/maven')
          secretVolume(secretName: 'jenkins-nepemail-token-bfxfb', mountPath: '/etc/jenkins'),
          persistentVolumeClaim(claimName: 'maven-local-repo', mountPath: '/etc/.m2repo')]) {

    node('maven-ose') {
        container(name: 'maven', cloud: 'openshift') {
            def WORKSPACE = pwd()
            //def mvnHome = tool 'maven'
            
            stage('Checkout') {
                checkout scm
            }
            
            stage('Maven Build') {
                sh """
                mvn clean package -DskipTests
                """
            }

            stage ('Binary Build') {
                sh """
                set +x
                oc login --certificate-authority=/var/run/secrets/kubernetes.io/serviceaccount/ca.crt --token=\$(cat /etc/jenkins/token) https://openshift.rhc-lab.iad.redhat.com:8443
                oc project nepemail-int
                oc start-build nepemail --from-file='./target/gs-spring-boot-docker-0.1.0.jar' -n nepemail-int --wait --follow
                """
            }
        }
    }
}
