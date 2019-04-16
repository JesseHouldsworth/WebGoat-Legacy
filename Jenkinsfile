import groovy.json.*

node () {
   def mvnHome, commitId
    
   stage('Preparation') { // for display purposes
      // Get some code from a GitHub repository
      // git 'git@github.com:CMYanko/struts2-showcase-demo.git'
      checkout scm
      
      
      // Get the Maven tool.
      // ** NOTE: This 'M3' Maven tool must be configured
      // **       in the global configuration.           
      // mvnHome = tool 'M3'
      
      // sh 'git rev-parse HEAD > commit'
      // commitId = readFile('commit').trim()
      // sh "echo my commitid ${commitId}"

   }
   stage('Build') {
      sh 'mvn clean install'     
   }

   
   stage('Lifecycle Evaluation'){
    // postGitHub commitId, 'pending', 'analysis', 'Nexus Lifecycle Analysis is running'

      def policyEvaluationResult = nexusPolicyEvaluation failBuildOnNetworkError: false, iqApplication: manualApplication('struts2-rce'), iqStage: 'stage-release', jobCredentialsId: ''
    /*  if (currentBuild.result == 'FAILURE'){
        postGitHub commitId, 'failure', 'analysis', 'Nexus Lifecycle Analysis failed',"${policyEvaluationResult.applicationCompositionReportUrl}"
        return
      } else {
        postGitHub commitId, 'success', 'analysis', 'Nexus Lifecycle Analysis succeeded',"${policyEvaluationResult.applicationCompositionReportUrl}"
      } */
   }

   
   stage('Build Docker Image'){
    sh 'pwd'
    sh 'ls -l'
    sh 'docker build -t webgoat:latest .'
   }
   
   stage('Deploy Docker Image'){
    sh 'docker run -d -p 9080:8080 webgoat:latest'
   }
   
   stage('Done testing?'){
    input 'Done?'
   }
   
   stage('Kill running webgoat container'){
    sh 'docker ps -a | awk \'{ print $1,$2 }\' | grep webgoat | awk \'{print $1 }\' | xargs -I {} docker kill {}'
   }
   
}

def postGitHub(commitId, state, context, description, target_url="http://localhost:8080") {
         def payload = JsonOutput.toJson(
           state: state,
           context: context,
           description: description,
           target_url: target_url
          )
        sh "curl -H \"Authorization: token ${gitHubApiToken}\" --request POST --data '${payload}' https://api.github.com/repos/${project}/statuses/${commitId}"
}
