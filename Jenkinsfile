pipeline 
{
  agent
  {
      label 'maven'
  }
         stages
        {
            stage('BUILD')
            {
                steps
                {
                    echo 'building'
                    sh 'mvn clean package'
                }
            }
            
            
            
                  stage('Create Container Image') 
                  {
                    steps 
                    {
                         echo 'Create Container Image..'
        
                        script 
                        {

                          openshift.withCluster() { 
                               openshift.withProject("calculate") {
  
                             def buildConfigExists = openshift.selector("bc", "frontend-java").exists() 
    
                          if(!buildConfigExists){ 
                          openshift.newBuild("--name=frontend-java", "--docker-image=registry.redhat.io/jboss-eap-7/eap74-openjdk8-openshift-rhel7", "--binary") 
                             } 
    
                       openshift.selector("bc", "frontend-java").startBuild("--from-file=target/jb-hello-world-maven-0.2.0-SNAPSHOT.jar", "--follow") 
                                   
                               } }

                             }
                      }
                  }
        
          stage('Deploy') {
      steps {
        echo 'Deploying....'
        script {

          openshift.withCluster() { 
            openshift.withProject("calculate") { 
              def deployment = openshift.selector("dc", "frontend-java") 
     
              if(!deployment.exists()){ 
                 openshift.newApp('frontend-java', "--as-deployment-config").narrow('svc').expose() 
                  } 
    
                      timeout(5) { 
                           openshift.selector("dc", "frontend-java").related('pods').untilEach(1) { 
                           return (it.object().status.phase == "Running") 
                                 } 
                            } 
                         } 
                      }
        }
         }
                }
}

}
