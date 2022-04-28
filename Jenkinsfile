pipeline {
   agent any
   environment {
         configFile = 'config.json'
   }
   
   tools { 
      maven "maven"
   }
   
    parameters {
        string(name: 'host', description: 'API Manager Host Name', defaultValue: 'forrester.demo.axway.com')
        string(name: 'stage', description: 'Targer Environment to Deploy', defaultValue: 'staging')
        string(name: 'returnCodeMapping', description: 'Swagger Promote CLI return code mapping', defaultValue: '10:0')
        string(name: 'appName', description: 'API Builder Appname')
       
    }

   stages {
      stage('Import API to Axway API Manager') {
         steps {
            
            
            withCredentials([usernamePassword(credentialsId: 'ars', usernameVariable: 'username', passwordVariable: 'password')])  {
               script {
                  sh 'axway acs login "${username} ${password}"'
                  domainName = sh 'axway acs list "${appName}" | grep URL: | grep us.axway.com | cut -c 20-200'
                  echo "Domain name ${domainName}"
                  def props = readJSON file: configFile
                  def backend = props.get("backendBasepath")
                  def newUrl =  backend.replace("localhost", domainName)
                  echo "New URL ${newUrl}"
                  props.put("backendBasepath",newUrl)
                  writeJSON file: configFile, json: props
                  
               }
            }
            
            script{
               def props = readJSON file: configFile
               if(stage.equals("preprod")){
                  def list = new ArrayList()
                  list.add("prod")
                  def tags = props.get("tags")
                  tags.accumulate("nextStage", list)
                  writeJSON file: configFile, json: props
               }
            }
            
            
            withCredentials([usernamePassword(credentialsId: "${stage}", usernameVariable: 'username', passwordVariable: 'password')])  {
               sh 'mvn clean exec:java -Dexec.args="-h ${host} -u ${username} -p ${password} -c  ${configFile} -force  -returnCodeMapping ${returnCodeMapping}"'
            }
     
         }
      }
   }
   post {
        always {
            echo 'Logout ARS'
            sh 'axway acs logout'
        }
   }
}
