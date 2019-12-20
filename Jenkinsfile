// So here is the general flow:
// You have a master branch which corresponds to current production code (and you dif between versions via git tags)
// You also have a develop branch for ongoing features and bug fixes
// 
// The below branches are more ephemeral in nature:
// For adding features you branch from current develop and work on whatever your adding until it is ready to be merged back into develop
// After merging to develop you'll create a release branch which will only receive bug fixes but no new features
    // Once the release branch is stable it will be merged into master and a new git tag will be created
// If there is the need for an emergency fix (to master) then a hotfix branch is created and subsuquently merged into master (and additional git tag created)

// Need to talk to devs about how they do their versioning when they


// Changing to demo

// Here is the current method we use for Bar deployment:
// 
// **** Preparation ****
// 
// ~~~ Step 1 ~~~
// Merge/PR is apporved
//
// ~~~ Step 2 ~~~
// Create Git Tag (For source and config repos) corresponding to version you are attempting to deploy
// 
// ~~~ Step 3 ~~~
// Checkout Tag in source & Build using SBT (command: clean assembly) 
// There is no artifactory sillyness
// 
// ~~~ Step 4 ~~~
// Navigate to config repo and utilize a `something-To-s3.sh` script to:
    // Perform Config Rewriting
        // Takes the configs it has and fill the values into the place where they are needed to be used
        // Then it will take the files with the now filled in config values along with the executable we just build and will upload them to S3
            // (/twc-emr-env-auto-prod/) --> spark submit scripts
            // (/twc-spark-jobs/bin/[app-name])
// 
// 
// Once all the above steps are done we are ready to deploy the EMR Launch Scripts to the HDFS Namenode (Yes the one you are thinking about)
// 
// 
// **** Deployment ****
// 
// 
// ~~~ Step 5 ~~~
// Log into HDFS Namenode as hdfs user
// 
// ~~~ Step 6 ~~~
// Trigger S3 Pull Jobs to:
    // (Path Is: /opt/sparkjobs/bin/[app-name])
    // Sync up all of the config files and newly built binaries we just uploaded to S3 in during Step 4
// 
// ~~~ Step 7 ~~~
// At this point we should be deployed and everything should be working as expected
// 
// There is a lot of stuff about this which should change...however we will work with this first and then move forward from here...
// Gitflow

// Setting this up for use in "Trigger hdfs Pull Jobs" stage for remote access:
// Dear Future me... Please realize this might not work because the namenode is very restrictive on where it allows
// connections from (you need to be connected to both cisco vpn and or atleast the F5 big-ip edge client vpn as well...)
// so this could very easily cause issues when attempting to connect.
// TODO: Make sure to abstract these into job parameters
// For details see DOCs : https://github.com/jenkinsci/ssh-steps-plugin  
  def remote = [:]
  remote.name = 'abastion4x00'
  remote.host = 'abastion4x00.analytics.weather.com'
  remote.user = 'noah.heil'
  remote.password = 'password'
  remote.allowAnyHosts = true
  remote.pty = true

pipeline {
    agent any
    environment {
       String result = '0.0.0';
    }
    options {
        ansiColor('xterm')
        // Use profile information from ~/.aws/config
	    withAWS(profile:'myProfile') 
        // You could also do this: 
        // withAWS(credentials:'IDofAwsCredentials') { do something } // Use Jenkins AWS credentials information (AWS Access Key: AccessKeyId, AWS Secret Key: SecretAccessKey)
        // Or this:
        // withAWS(credentials:'IDofSystemCredentials') { do something } // Use Jenkins UsernamePassword credentials information (Username: AccessKeyId, Password: SecretAccessKey):
    }
    stages {
        // stage('Preparation') {
        //     steps {
        //         echo "Build Preparation"
        //         checkout scm 
        //     }
        // }
        stage("Tag commit with build id") {
            steps {
                script {
                    withCredentials([usernamePassword(credentialsId: '4fda8056-07ba-43b3-a1eb-f8e6cd8e44a6', usernameVariable: 'GIT_USERNAME', passwordVariable: 'GIT_PASSWORD')]) {

                        // use BUILD_ID for tag
                        def tag = BUILD_ID

                        // configure the git credentials, these are cached in RAM for several minutes to use
                        // this is required until https://issues.jenkins-ci.org/browse/JENKINS-28335 is resolved upstream
                        sh "echo 'protocol=https\nhost=https://github.com/Noah-Heil/meh\nusername=${GIT_USERNAME}\npassword=${GIT_PASSWORD}\n\n' | git credential approve "

                        sh "git tag -a ${tag} -m 'Noah-Heil tagging'"
                        sh "git remote set-url --push origin https://${GIT_USERNAME}:${GIT_PASSWORD}@github.com/Noah-Heil/meh.git"
                        sh "git push --tags"
                    }
                }
            }
        }
        stage('Auto tagging') { 
            steps {
                script {
                    // version=\$(git describe --tags)
                    // withCredentials([usernamePassword(credentialsId: '4fda8056-07ba-43b3-a1eb-f8e6cd8e44a6', passwordVariable: 'password_name', usernameVariable: 'git_username')]) { 
                    //     sh 'git config --local credential.helper "!p() { echo username=\\$git_username; echo password=\\$password_name; }; p"'
                    //     sh 'git tag -m "" ${VERSION_NUMBER}'
                    // }

                    // withCredentials([usernamePassword(credentialsId: 'github', usernameVariable: 'git_username', passwordVariable: 'password_name')]) {
                    //     sh 'git push origin ${VERSION_NUMBER}'
                    // }

                    sh """ 
                    version=\$(git describe --tags `git rev-list -—tags —-max-count=1`)
                    
                    #Version to get the latest tag 
                    A="\$(echo \$version|cut -d '.' -f1)"
                    B="\$(echo \$version|cut -d '.' -f2)"
                    C="\$(echo \$version|cut -d '.' -f3)"
                     if [ \$C -gt 8 ]
                     then 
                    if [ \$B -gt 8 ]
                     then
                     A=\$((A+1))
                     B=0 C=0 
                    else
                     B=\$((B+1))
                     C=0
                     fi
                    else
                     C=\$((C+1))
                     fi
                    echo "A[\$A.\$B.\$C]">outFile """
                    nextVersion = readFile 'outFile' 
                    echo "we will tag '${nextVersion}'" 
                    result =nextVersion.substring(nextVersion.indexOf("[")+1,nextVersion.indexOf("]"))
                    echo "we will tag '${result}'"
                    withCredentials([usernamePassword(credentialsId: '4fda8056-07ba-43b3-a1eb-f8e6cd8e44a6', passwordVariable: 'password_name', usernameVariable: 'git_username')]) { 
                        sh """ 
                        curl -d '{
                        "tag_name": "${result}",
                        "target_commitish": "release",
                        "name": "${result}",
                        "body": "Release of version ${result}",
                        "draft": false, "prerelease": false}' -X POST https://api.github.com/repos/Noah-Heil/meh/releases"""
                    }
                    // withCredentials([usernamePassword(credentialsId: '4fda8056-07ba-43b3-a1eb-f8e6cd8e44a6', passwordVariable: 'password_name', usernameVariable: 'git_username')]) { 
                    //     sh """ 
                    //     curl -d '{
                    //     "tag_name": "${result}",
                    //     "target_commitish": "release",
                    //     "name": "${result}",
                    //     "body": "Release of version ${result}",
                    //     "draft": false, "prerelease": false}' -X POST https://api.github.com/repos/Noah-Heil/meh/releases?access_token=$password_name"""
                    // }
                }
            }
        }
        stage('Tagging') { // for display purposes
            // Get some code from a GitHub repository
            steps {
                script {
                        repositoryCommiterEmail = 'nceheil@gmail.com'
                        repositoryCommiterUsername = 'Noah-Heil'
                        checkout scm
                        sh "git remote set-url origin git@github.com:..."

                        // deletes current snapshot tag
                        sh "git tag -d snapshot || true"
                        // tags current changeset
                        sh "git tag -a snapshot -m \"passed CI\""
                        // deletes tag on remote in order not to fail pushing the new one
                        sh "git push origin :refs/tags/snapshot"
                        // pushes the tags
                        sh "git push --tags"
                }
                // Keeping for Nostalgia
                // script {
                //     env.BRANCH_NAME = "master"// BRANCH_NAME is predefined in multibranch pipeline job
                //     env.J_GIT_CONFIG = "true"
                //     env.J_USERNAME = "Jenkins CI"
                //     env.J_EMAIL = "jenkins-ci@example.com"
                //     env.J_CREDS_IDS = 'f62c4435-f490-4659-8afc-510efd848445' // Use credentials id from Jenkins
                //     echo "git 'https://github.com/TheWeatherCompany/analytics-pipeline-insinkerator.git'"
                //     repositoryCommiterEmail = 'ci@example.com'
                //     repositoryCommiterUsername = 'example.com'

                //     // f62c4435-f490-4659-8afc-510efd848445



                //     sh "echo done"
                //     echo env.BRANCH_NAME
                //     // if (env.BRANCH_NAME == 'master') {

                        // echo "inside"


                        // sh "git remote set-url origin git@github.com:..."

                        // // deletes current snapshot tag
                        // sh "git tag -d snapshot || true"
                        // // tags current changeset
                        // sh "git tag -a snapshot -m \"passed CI\""
                        // // deletes tag on remote in order not to fail pushing the new one
                        // sh "git push origin :refs/tags/snapshot"
                        // // pushes the tags
                        // sh "git push --tags"
                    // }
                // }
            }
        }
        stage('Build') { // We do want to tag for Development
            steps {
                echo "Building Tag"


                // TODO: Checkout potentially uselful section commented out below in this same step
                // echo "checkout scm: [$class: 'GitSCM', userRemoteConfigs: [[url: repoURL, credentialsId: credential]], branches: [[name: tag-version]]],poll: false"
                echo "sbt clean assembly"
                // https://stackoverflow.com/questions/53210867/jenkins-pipeline-with-a-sbt-project

                // Store Binary 
                // https://stackoverflow.com/questions/36843215/how-can-i-use-the-jenkins-copy-artifacts-plugin-from-within-the-pipelines-jenki
                echo "Store Binary"
                echo "sh 'mkdir archive'"
                echo "sh 'echo test > archive/test.txt'"
                echo "zip zipFile: 'test.zip', archive: false, dir: 'archive'"
                echo "archiveArtifacts artifacts: 'test.zip', fingerprint: true"

                // Potentially useful:
                // stage('Checkout') {
                //     withEnv([
                //         "LAST_SUCCESSFUL_EXTENSIONS_REPO_BUILD=${ sh ( script: "curl <JENKINS_URL>/job/extensionsrepo/job/${BRANCH_NAME}/lastSuccessfulBuild/buildNumber", returnStdout: true ) }",
                //         "LAST_SUCCESSFUL_SHARED_REPO_BUILD=${ sh ( script: "curl <JENKINS_URL>/job/sharedrepo/job/${BRANCH_NAME}/lastSuccessfulBuild/buildNumber", returnStdout: true ) }"
                //     ]) {
                //         sh 'echo ${BRANCH_NAME}_${LAST_SUCCESSFUL_EXTENSIONS_REPO_BUILD}'
                //         sh 'echo ${BRANCH_NAME}_${LAST_SUCCESSFUL_SHARED_REPO_BUILD}'
                //         checkout scm
                //         checkout([
                //             $class: 'GitSCM',
                //             branches: [[name: 'refs/tags/${BRANCH_NAME}_${LAST_SUCCESSFUL_EXTENSIONS_REPO_BUILD}']],
                //             doGenerateSubmoduleConfigurations: false,
                //             extensions: [[
                //                 $class: 'RelativeTargetDirectory', 
                //                 relativeTargetDir: 'app/extensions'
                //             ]],
                //             submoduleCfg: [],
                //             userRemoteConfigs: [[
                //                 credentialsId: 'ssh-key',
                //                 url: '<GIT_URL>'
                //             ]]
                //         ])
                //         checkout([
                //             $class: 'GitSCM',
                //             branches: [[name: 'refs/tags/${BRANCH_NAME}_${LAST_SUCCESSFUL_SHARED_REPO_BUILD}']],
                //             doGenerateSubmoduleConfigurations: false,
                //             extensions: [[
                //                 $class: 'RelativeTargetDirectory', 
                //                 relativeTargetDir: 'app/shared'
                //             ]],
                //             submoduleCfg: [],
                //             userRemoteConfigs: [[
                //                 credentialsId: 'ssh-key',
                //                 url: '<GIT_URL>'
                //             ]]
                //         ])
                //     }
                // }
            }
        }
        stage('Config Management') {
            // Get configs from GitHub
            // https://stackoverflow.com/questions/40224272/using-a-jenkins-pipeline-to-checkout-multiple-git-repos-into-same-job
            // checkout([  
            //             $class: 'GitSCM', 
            //             branches: [[name: 'refs/heads/analytics-ingest-bar']], 
            //             doGenerateSubmoduleConfigurations: false, 
            //             extensions: [[$class: 'RelativeTargetDirectory', relativeTargetDir: 'config']], 
            //             submoduleCfg: [], 
            //             userRemoteConfigs: [[credentialsId: 'REPLACE_ME', url: 'git@github.com:TheWeatherCompany/analytics-pipeline-data_ingest.git']]
            //         ])
            steps {
                // TODO: This script should eventually be removed and just done inside of jenkins
                // TODO: Check if the following README section is valid: https://github.com/TheWeatherCompany/analytics-pipeline-data_ingest/tree/analytics-ingest-bar#initial-deployment-setup
                echo "sh '. config/utilities/analytics-pipeline-data_ingest-to-s3.sh'"
            }
        }
        stage('Upload To S3') {
            steps {
                // Documentation for using Jenkins AWS Pipeline Plugin: https://github.com/jenkinsci/pipeline-aws-plugin
                // Where you can find these:
                // s3Upload(pathStyleAccessEnabled: true, payloadSigningEnabled: true, file:'file.txt', bucket:'my-bucket', path:'path/to/target/file.txt')
                // s3Copy(pathStyleAccessEnabled: true, fromBucket:'my-bucket', fromPath:'path/to/source/file.txt', toBucket:'other-bucket', toPath:'path/to/destination/file.txt')
                // s3Delete(pathStyleAccessEnabled: true, bucket:'my-bucket', path:'path/to/source/file.txt')
                // s3Download(pathStyleAccessEnabled: true, file:'file.txt', bucket:'my-bucket', path:'path/to/source/file.txt', force:true)
                // exists = s3DoesObjectExist(pathStyleAccessEnabled: true, bucket:'my-bucket', path:'path/to/source/file.txt')
                // files = s3FindFiles(pathStyleAccessEnabled: true, bucket:'my-bucket')
                // 

                // Retrieve Binary
                // https://stackoverflow.com/questions/36843215/how-can-i-use-the-jenkins-copy-artifacts-plugin-from-within-the-pipelines-jenki
                echo "Retrieve Binary"
                echo "copyArtifacts filter: 'test.zip', fingerprintArtifacts: true, projectName: '${JOB_NAME}', selector: specific('${BUILD_NUMBER}')"
                echo "unzip zipFile: 'test.zip', dir: './archive_new'"
                echo "sh 'cat archive_new/test.txt'"

                // Make sure you have your credentials are setup in the options section above (which is above the stages section but below the environment section)
                // Otherwise this will not function as expected
                echo "stage Upload To S3"
                echo "s3Upload(pathStyleAccessEnabled: true, payloadSigningEnabled: true, file:'test.txt', bucket:'my-bucket', path:'path/to/target/test.txt')"
            }
        }
        stage('Trigger hdfs Pull Jobs') {
            steps {
                // Dear Future me... Please realize the ssh connection might not work because the namenode is very restrictive on where it allows
                // connections from (you need to be connected to both cisco vpn and or atleast the F5 big-ip edge client vpn as well...)
                // so this could very easily cause issues when attempting to connect.

                // ~~~ Step 6 ~~~
                // Trigger S3 Pull Jobs to:
                    // (Path Is: /opt/sparkjobs/bin/[app-name])
                    // Sync up all of the config files and newly built binaries we just uploaded to S3 in during Step 4
                // 
                echo 'sshCommand remote: remote, command: "ssh -t hdfsname4b01 sudo su - hdfs -c ls /opt/sparkjobs/bin/"'
            }
        }
        stage('Results') {
            steps {
                echo "stage results"
                echo "we should be done now"
            }
        }
    }
}