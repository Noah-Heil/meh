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
        // Takes the configs it has and fill the values into th place where they are needed to be used
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

pipeline {
    agent any
    stages {
        stage('Tagging') { // for display purposes
            // Get some code from a GitHub repository
            steps {
                script {
                    sshagent(credentials: ['f62c4435-f490-4659-8afc-510efd848445']) {
                        def repository = "git@" + env.GIT_URL.replaceFirst(".+://", "").replaceFirst("/", ":")

                        sh("git remote set-url origin https://github.com/Noah-Heil/meh.git")
                        sh("git tag --force build-master")
                        sh("git push --force origin build-master")
                    }
                }
                // script {
                //     env.BRANCH_NAME = "master"// BRANCH_NAME is predefined in multibranch pipeline job
                //     env.J_GIT_CONFIG = "true"
                //     env.J_USERNAME = "Jenkins CI"
                //     env.J_EMAIL = "jenkins-ci@example.com"
                //     env.J_CREDS_IDS = 'f62c4435-f490-4659-8afc-510efd848445' // Use credentials id from Jenkins
                //     echo "git 'https://github.com/TheWeatherCompany/analytics-pipeline-insinkerator.git'"
                //     repositoryCommiterEmail = 'ci@example.com'
                //     repositoryCommiterUsername = 'examle.com'

                //     // f62c4435-f490-4659-8afc-510efd848445


                //     checkout scm

                //     sh "echo done"
                //     echo env.BRANCH_NAME
                //     // if (env.BRANCH_NAME == 'master') {

                        // echo "inside"

                        // sh("git config user.email ${repositoryCommiterEmail}")
                        // sh("git config user.name '${repositoryCommiterUsername}'")

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
                echo "stage Build"
                echo "sbt clean assembly"
            }
        }
        stage('Upload To S3') {
            steps {
                echo "stage Upload To S3"

            }
            // Run the maven build
            // withEnv(["MVN_HOME=$mvnHome"]) {
                // if (isUnix()) {
                    // sh '"$MVN_HOME/bin/mvn" -Dmaven.test.failure.ignore clean package'
                // } else {
                    // bat(/"%MVN_HOME%\bin\mvn" -Dmaven.test.failure.ignore clean package/)
                // }
            // }
        }
        stage('Results') {
            steps {
                echo "stage results"

            }
            // junit '**/target/surefire-reports/TEST-*.xml'
            // archiveArtifacts 'target/*.jar'
        }
    }
}