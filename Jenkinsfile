node {

    agent any
    
    // def mvnHome
    stage('Preparation') { // for display purposes
        steps {
            // Get some code from a GitHub repository
            echo "git 'https://github.com/TheWeatherCompany/analytics-pipeline-insinkerator.git'"
        }
        // Get the Maven tool.
        // ** NOTE: This 'M3' Maven tool must be configured
        // **       in the global configuration.           
        // mvnHome = tool 'M3'
    }
    stage('Tagging') { // We do want to tag for Development
        steps {
            echo "stage tagging"

        }
    }
    stage('Build') {
        steps {
            echo "stage Build"

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





}