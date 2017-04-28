OC_LOG_LEVEL=3

node ('maven') {

    def WORKSPACE = pwd()
   
    def ENV_INT  = 'nepemail-int'
    def ENV_UAT  = 'nepemail-uat'
    def ENV_PROD = 'nepemail-prod'

    def APP = 'nepemail'
    def ALL_APPS = [APP]

    stage 'Checkout'
        checkout scm

    stage 'Maven Build'

        // build the project
	// mvnHome = tool 'maven'
           sh """
             set -x 
	     mvn -gs /etc/maven/settings.xml clean install -DskipTests
	   """
    
    stage "Binary Build"
        // NOTE: in theory this shouldn't be needed but run into issue where -n flag on commands isn't
        // being adhered to so this is the work around
        sh """
            set -x
            oc login --certificate-authority=/var/run/secrets/kubernetes.io/serviceaccount/ca.crt --token=\$(cat /etc/jenkins/token) https://openshift.rhc-lab.iad.redhat.com:8443

            oc project ${ENV_INT}
            oc start-build ${APP} --from-file='./target/gs-spring-boot-docker-0.1.0.jar' -n ${ENV_INT} --wait --follow
        """

    stage "Integration Deploy"
        // NOTE: in theory this shouldn't be needed but run into issue where -n flag on commands isn't
        // being adhered to so this is the work around
        sh """
            set -x
            oc project ${ENV_INT}
        """

        deploy(APP, ENV_INT, 'latest')


}

/* Deploys the given application names to the given namspace.
 *
 * @param app       [String]  Application names to deploy. Names must map to DeploymentConfigs
 *                           and ImageStreams of the same name.
 * @param namespace [String] Namespace to deploy the applicaitons in.
 * @param version   [String] Version of the applications (ImageStreams) to deploy.
 */
def deploy(app, namespace, version) {
    deploys = [:]
    deploys["${appName}"] = {
        sh """
            set -x
    
            # get the latest image reference from the image stream
            newImageReference=\$(oc get is ${app} -n ${namespace} -o jsonpath="{.status.tags[?(@.tag==\\"${version}\\")].items[0].dockerImageReference}")
            
            # update the DeploymnetConfig to deploy the latest version
            oc patch dc/${app} -n ${namespace} --loglevel=${OC_LOG_LEVEL} -p "
              {\\"spec\\":
                {\\"template\\":
                  {\\"spec\\":
                    {\\"containers\\":
                      [{
                        \\"name\\":\\"${appName}\\",
                        \\"image\\":\\"\${newImageReference}\\"
                      }]
                    }
                  }
                }
              }"
    
            # deploy the latest image
            oc deploy ${app} -n ${namespace} --latest --follow --loglevel=${OC_LOG_LEVEL}
         
        """
    }

    parallel deploys
}

/* Promotes an image from one namespace to another by tagging the image in the new namespace.
 *
 * @param imageStreamNames [Array]  ImageStream names to promote
 * @param srcNamespace     [String] Source namspace to promote the ImageStreams from.
 * @param destNamespace    [String] Destination namespace to promote the ImageStreams to.
 * @param version          [String] Version of the ImageStreams to promote.
 */
def promote(imageStreamNames, srcNamespace, destNamespace, version) {
    for ( is in imageStreamNames ) {
        def imageStream = is //if you use 'an' below, it will equal the last value in each parallel run

        sh """
          set -x
          oc tag ${srcNamespace}/${imageStream}:${version} ${destNamespace}/${imageStream}:${version} --alias=false --loglevel=${OC_LOG_LEVEL}
        """
    }
}


