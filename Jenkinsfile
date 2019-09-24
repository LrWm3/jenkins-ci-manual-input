pipeline {
  // We want to use agents per stage to avoid blocking our build agents
  // while we are waiting for user input.
  agent none
  // The question mark naming convention is helpful to show you which
  //  approval stage belongs to which work stage.
  stages {
    stage("Release?") {
      parallel {
        stage('Staging') {
          // Unlike staging, Staging parallel is conditionally executed
          // Execute if this is master
          // Execute if this is feature branch
          stages('Staging') {
            stage('Staging: Release?') {
              // Don't allocate an agent because we don't want to block our
              // slaves while waiting for user input.
              agent none
              options {
                // Optionally, let's add a timeout that we don't allow ancient
                // builds to be released.
                timeout time: 3, unit: 'DAYS' 
              }
              // I don't know if before-agent is even possible here...
              // We might need separate jobs; one for manual input another for automatic deployment...
              //when {
              //  // Evaluate the 'when' directive before allocating the agent.
              //  beforeAgent true
                  // TODO - when branch name is not master
              //}
              steps {
                // The input statement has to go to a script block because we
                // want to assign the result to an environment variable. As we 
                // want to stay as declarative as possible, we put nothing but
                // this into the script block.
                script {
                  // Assign the 'DO_STAGING_RELEASE' environment variable that is going
                  //  to be used in the next stage.
                  env.DO_STAGING_RELEASE = "yes"
          
                }

                // In case you approved multiple pipeline runs in parallel, this
                // milestone would kill the older runs and prevent deploying
                // older releases over newer ones.

                // TODO : milestones and locks dont work inside a 'parallel' block sadly: 
                //  see: https://issues.jenkins-ci.org/browse/JENKINS-47179?page=com.atlassian.jira.plugin.system.issuetabpanels%3Acomment-tabpanel&showAll=true
                // - can we use filehandles for locks?
                // - can we somehow check, once locked, if this build is no longer the latest?
                // - if yes to both, we can use those groovy functions as locks
                // milestone ([ordinal:3, label:"Milestone: Performing Approved Release"])
                // milestone ([ordinal:2, label: "Milestone: Approved Manual Release"])
              }
            }
            stage('Staging: Release') {
              // We need a real agent, because we want to do some real work.
              agent any
              when {
                // Evaluate the 'when' directive before allocating the agent.
                beforeAgent true
                // Only execute the step when the release has been approved.
                environment name: 'DO_STAGING_RELEASE', value: 'yes'
              }
              steps {
                // Make sure that only one release can happen at a time.
                // lock('staging-release') {
                  // As using the first milestone only would introduce a race 
                  // condition (assume that the older build would enter the 
                  // milestone first, but the lock second) and Jenkins does
                  // not support inter-stage locks yet, we need a second 
                  // milestone to make sure that older builds don't overwrite
                  // newer ones.

                  // TODO : milestones and locks dont work
                  // - can we use filehandles for locks?
                  // - can we somehow check, once locked, if this build is no longer the latest?
                  // - if yes to both, we can use those groovy functions as locks
                  // milestone ([ordinal:3, label:"Milestone: Performing Approved Release"])
          
                  // Now do the actual work here.
                  sh (script: "echo 'always enable staging deployments'", label: "Staging Echo")
                  sh([script:"echo \"DO_STAGING_RELEASE=$DO_STAGING_RELEASE\"", label: 'Echo $DO_STAGING_RELEASE'])
                // }
              }
            }
          }
        }
        stage('Canary') {
          // Unlike staging, canary parallel is conditionally executed
          // Execute if this is master automatically
          // Else, execute if this is feature branch and someone manually says "do this"
          stages('Canary') {
            stage('Canary: Release?') {
              agent none
              options {
                timeout time: 3, unit: 'DAYS' 
              }
              when {
                // If we're on a tag, we will automatically deploy to canary?
                // Well, we might not want to, if the tag we're building is older then whatever is on master
                // maybe only master and feature branches are deployed to canary?
                // 
                // or maybe asking if a canary release for a tag is appropriate (I think it is appropriate to ask?)
                // although we're saying it will be automatically deployed to a release... so...
                // lets think about how we want to handle master and tags a little more before deciding
                //
                // for now, we will only automatically deploy master to canary but not tags
                // arguably we need a way of regexp deploying to a bunch of clusters based on branch / tag names...?
                // not sure really... I'll think about this more...
                not { 
                  // anyOf{ branch "master"; buildingTag() } 
                  branch "master"
                }
              }
              steps {
                // TODO - DIY milestones and locks, somehow? to prevent old jobs from building?
                script {
                  env.DO_CANARY_RELEASE = input (
                      parameters: [choice(name: 'DO_CANARY_RELEASE', choices: ['yes', 'no'], description: 'Deploy this build in-line to canary')]
                  )
                }
              }
            }
            stage('Canary: Release') {
              agent any
              when {
                beforeAgent true
                // Only execute the step when the release has been approved.
                anyOf {
                  environment name: 'DO_CANARY_RELEASE', value: 'yes'
                  branch "master"
                  buildingTag()
                }
              }
              steps {
                // TODO - milestones and locks, somehow?
                sh([script:"echo \"DO_CANARY_RELEASE=$DO_CANARY_RELEASE\"", label: 'Echo $DO_CANARY_RELEASE'])
              }
            }
          }
        }
        stage('Tag') {
          // - on master, we should ask if this merge should be released
          // - if yes, jenkins should go through the release process
          // - tagging on master, uprevving, etc etc
          //
          // Then, upon running this job again for the tag, 
          // it will release the project to production automatically
          // ^^ I think that would be good? ^^
          stages('Tag') {
            stage('Tag: Release?') {
              agent none
              options {
                timeout time: 3, unit: 'DAYS' 
              }
              when {
                beforeAgent true
                branch "master"
              }
              steps {
                script {
                  env.DO_PRODUCTION_RELEASE = input (
                      parameters: [string(name: 'DO_PRODUCTION_RELEASE', defaultValue: 'do not release', description: 'Enter "release" to tag and deploy this revision to production, anything else not formally release (do not abort)')]
                  )
          
                }
                // TODO - milestones and locks, somehow?
              }
            }
            stage('Tag: Release') {
              agent any
              when {
                beforeAgent true
                environment name: 'DO_PRODUCTION_RELEASE', value: 'yes'
                branch "master"
              }
              steps {
                  // TODO - milestones and locks, somehow?
                  // Now do the actual work here.
                  sh([script:"echo \"DO_PRODUCTION_RELEASE=$DO_PRODUCTION_RELEASE\"", label: 'Echo $DO_PRODUCTION_RELEASE'])
                  sh([script:"echo \"I am doing the tag + release procedure for the project!\"", label: 'Tagging and releasing '])
                // }
              }
            }
          }
        }
        stage('Production') {
          // Execute if this is tagged automatically
          stages('Production') {
            stage('Production: Release?') {
              agent none
              options {
                timeout time: 3, unit: 'DAYS' 
              }
              when {
                // Evaluate the 'when' directive before allocating the agent.
                beforeAgent true
                buildingTag()
                // TODO - checking this matches a 'release' tag is advisable
                // TODO - may want to be able to point different major / minor releases at different
                //        clusters; e.g. 1.2.4 released (customer hotfix), vs 1.3.0 (new features)
              }
              steps {
                script {
                  // Automatically release, this is a tag
                  // Placeholder in case we want more complex logic later
                  env.DO_PROD_RELEASE = "release"
          
                }
                // TODO - milestones and locks, somehow?
              }
            }
            stage('Production: Release') {
              // We need a real agent, because we want to do some real work.
              agent any
              when {
                // Evaluate the 'when' directive before allocating the agent.
                beforeAgent true
                // Only execute the step when the release has been approved.
                environment name: 'DO_PROD_RELEASE', value: 'yes'
              }
              steps {
                // TODO - milestones and locks, somehow?
                  sh([script:"echo \"DO_PROD_RELEASE=$DO_PROD_RELEASE\"", label: 'Echo $DO_PROD_RELEASE'])
              }
            }
          }
        }
      }
    }
  }
}