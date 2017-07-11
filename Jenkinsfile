node (env.SLAVE) {
       def student = 'ndolya'
       env.PATH=env.PATH +":/home/student/groovy-2.4.12/bin:/home/student/gradle/gradle-4.0.1/bin"

 stage('Preparation (Checking out)') {
       git([url: 'https://github.com/MNT-Lab/mntlab-pipeline.git', branch: "${student}"])
         }

 stage('Building code') {
              sh "gradle build"
           }
 stage('Testing') {

    parallel (
     'firstTest': { sh "gradle cucumber" },
     'secondTest' : { sh "gradle jacocoTestReport" },
     'thirdTest' : {  sh "gradle test" }
      )
 }

 stage ('Triggering job and fetching artefact after finishing') {

 build job: "EPBYMINW1969/MNTLAB-${student}-child1-build-job", parameters: [string(name: 'BRANCH_NAME', value: "${student}")]
 echo WORKSPACE
step([$class: 'CopyArtifact',
      filter: "${student}_dsl_script.tar.gz",
      projectName: "EPBYMINW1969/MNTLAB-${student}-child1-build-job" ])
      }

  stage ('Packaging and Publishing results') {

       sh "tar -xf ${student}_dsl_script.tar.gz jobs.groovy"
       sh "tar -czf pipeline-${student}-${BUILD_NUMBER}.tar.gz jobs.groovy Jenkinsfile -C build/libs gradle-simple.jar"
       archiveArtifacts "pipeline-${student}-${BUILD_NUMBER}.tar.gz"
       sh "groovy pull-push.groovy -p push -a pipeline-${student}-${BUILD_NUMBER}.tar.gz"
  }

  stage ('Asking for manual approval') {
        input 'Do you approve deployment?'
  }

  stage ('Deployment') {
     sh "java -jar build/libs/gradle-simple.jar"
  }

  stage ('Sending status') {
         echo "ALL STEPS SUCCESS"
  }
}
