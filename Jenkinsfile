podTemplate(label: 'maven-ose', cloud: 'openshift', containers: [
  containerTemplate(name: 'maven', image: "registry.access.redhat.com/openshift3/jenkins-slave-maven-rhel7"),
],
volumes: [secretVolume(secretName: 'jenkins-nepemail-token-va3uy', mountPath: '/root/jenkins'),
          persistentVolumeClaim(claimName: 'jenkins', mountPath: '/home/jenkins/.m2')]) {

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
                oc login --certificate-authority=/var/run/secrets/kubernetes.io/serviceaccount/ca.crt --token=\$(cat /root/jenkins/token) https://openshift.rhc-lab.iad.redhat.com:8443
                oc project nepemail-int
                oc start-build nepemail --from-file='./target/gs-spring-boot-docker-0.1.0.jar' -n nepemail-int --wait --follow
                """
            }
        }
    }
}
