   def appName='TestApplication'
   def deployName = 'Test'
   def dataFormat ='json'
   def configFile ='config/**/Component/*.json'
   def target ='deployable'
   def namePath ="jenkinsTest/${currentBuild.number}"
   def commit =true
   def validate =true
   def convertPath =true
   def changesetNumber=""
   def snapshotName=''

   def fullFileName="${appName}-${deployName}-${currentBuild.number}.${dataFormat}"
   def exporterName ='testExporter' 
   
   
   
   
pipeline {
   agent any
   stages {
       stage('Clone repository') {               
          steps{
               // checkout scm
               git branch: 'master', url: 'https://github.com/iamkumaramit/samplejava.git'
          }
       }     
       stage('Upload Configuration file'){
           steps{
               script{
                   sh "echo uploading configuration file ${configFile}"
                   echo "name path ::::: ${namePath}"
                   changesetNumber = snDevOpsConfigUpload(
                            applicationName: "${appName}",
                            deployableName: "${deployName}",
                            dataFormat: "${dataFormat}",
                            configFile: "${configFile}",
                            target:"${target}",
                            namePath: "${namePath}",
                            autoCommit:"false",
                            autoValidate:"${validate}",
                            changesetNumber:"${changesetNumber}",
                            convertPath:"${convertPath}"
                            )
                            
                    //upload to test
//                      snDevOpsConfigUpload(
//                             applicationName: "${appName}",
//                             deployableName: "test",
//                             dataFormat: "${dataFormat}",
//                             configFile: "${configFile}",
//                             target:"${target}",
//                             namePath: "${namePath}",
//                             autoCommit:"false",
//                             autoValidate:"${validate}",
//                             changesetNumber:"${changesetNumber}",
//                             convertPath:"${convertPath}"
//                     )
                    
                     //upload to PreProd
//                             snDevOpsConfigUpload(
//                             applicationName: "${appName}",
//                             deployableName: "preprod",
//                             dataFormat: "${dataFormat}",
//                             configFile: "${configFile}",
//                             target:"${target}",
//                             namePath: "${namePath}",
//                             autoCommit:"${commit}",
//                             autoValidate:"${validate}",
//                             changesetNumber:"${changesetNumber}",
//                             convertPath:"${convertPath}"
//                     )
                   echo "Upload result $changesetNumber"
               }
           }
       }
       stage("Register change set to pipeline"){
           steps{
               script{
                   echo "Change set registration for ${changesetNumber}"
                   changeSetRegResult = snDevOpsConfigRegisterChangeSet(
                   changesetNumber:"${changesetNumber}"
                   )
                   echo "change set registration set result ${changeSetRegResult}"
               }
           }
       }
       stage("Get snapshots created"){
           steps{
               echo "Triggering Get snapshots for applicationName:${appName},deployableName:${deployName},changeSetId:${changesetNumber}"
               script{
                   changeSetResults = snDevOpsConfigGetSnapshots(
                         applicationName: "${appName}",
                         changesetNumber: "${changesetNumber}",
                         deployableName: "${deployName}"
                         ,outputFormat:"xml"
                         )
                   echo "ChangeSet Result : ${changeSetResults}"
                   def changeSetResultsObject = readJSON text: changeSetResults
                        changeSetResultsObject.each {
                           if(it.validation == "passed"){
                               echo "validation passed for snapshot : ${it.name}"
                               snapshotName = it.name
                           }else{
                               echo "Snapshot failed to get validated : ${it.name}" ;
                               assert it.validation == "passed"
                           }
                       }
                 if (!snapshotName?.trim()){
                   error "No snapshot found to proceed" ;
                 }
                 echo "Snapshot Name : ${snapshotName} "  
               }
           }
       }
       stage('Publish the snapshot'){
           steps{
               script{
                   echo "Step to publish snapshot applicationName:${appName},deployableName:${deployName} snapshotName:${snapshotName}"
                   publishSnapshotResults = snDevOpsConfigPublish(
                   applicationName:"${appName}",
                   deployableName:"${deployName}",
                   snapshotName: "${snapshotName}"
                   )
                   echo " Publish result for applicationName:${appName},deployableName:${deployName} snapshotName:${snapshotName} is ${publishSnapshotResults}"
               }
           }
       }
       stage('Initiate ChangeRequest'){
           steps{
               echo "Devops Change trigger change request"
               snDevOpsChange()
           }

       }
       stage('Export Snapshots from Service Now') {
           steps{
               script{
                   echo "Exporting for App: ${appName} Deployable; ${deployName} Exporter name ${exporterName} "
                   echo "Configfile exporter file name ${fullFileName}"
                   sh  'echo "<<<<<<<<<export file is starting >>>>>>>>"'
                   response = snDevOpsConfigExport(
                  applicationName: "${appName}", 
                  snapshotName: "${snapshotName}", 
                  deployableName: "${deployName}", 
                  exporterFormat: "${dataFormat}", 
                  fileName:"${fullFileName}", 
                  exporterName: "${exporterName}"
                  )
                   echo " RESPONSE FROM EXPORT : ${response}"
               }
           }
       }
       stage("Deploying to Prod"){
           steps{
               echo "Reading config from file name ${fullFileName}"
               echo " ++++++++++++ BEGIN OF File Content ***************"
               sh "cat ${fullFileName}"
               echo " ++++++++++++ END OF File content ***************"
               echo "deploy finished successfully."
           }
       }
   }
   post{
        always{
            echo ">>>>>Displaying Test results"
              junit '**/*.xml'
        }
    }
}
