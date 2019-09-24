pipeline {
  // We want to use agents per stage to avoid blocking our build agents
  // while we are waiting for user input.
  agent none
  // The question mark naming convention is helpful to show you which
  //  approval stage belongs to which work stage.
  stages {
      stage('Canary: Release?') {
        // Don't allocate an agent because we don't want to block our
        // slaves while waiting for user input.
        agent none
        options {
          // Optionally, let's add a timeout that we don't allow ancient
          // builds to be released.
          timeout time: 3, unit: 'DAYS' 
        }
        // TODO - when branch name is not master
        //when {
        //  // Evaluate the 'when' directive before allocating the agent.
        //  beforeAgent true
        //  // Only execute the step when the release has been approved.
        //  environment name: 'DO_RELEASE', value: 'yes'
        //}
        steps {
          // The input statement has to go to a script block because we
          // want to assign the result to an environment variable. As we 
          // want to stay as declarative as possible, we put noting but
          // this into the script block.
          script {
            // Assign the 'DO_RELEASE' environment variable that is going
            //  to be used in the next stage.
            env.DO_RELEASE = input (
                //message: "Deploy to canary?",
                parameters: [choice(name: 'DO_RELEASE', choices: ['yes', 'no'], description: 'Deploy this build in-line to canary')]
            )
    
          }
          // In case you approved multiple pipeline runs in parallel, this
          // milestone would kill the older runs and prevent deploying
          // older releases over newer ones.
          milestone ([ordinal:1, label: "Milestone: Approved Manual Release"])
        }
      }
      stage('Canary: Release') {
        // We need a real agent, because we want to do some real work.
        agent any
        when {
         // Evaluate the 'when' directive before allocating the agent.
         beforeAgent true
         // Only execute the step when the release has been approved.
         environment name: 'DO_RELEASE', value: 'yes'
        }
        steps {
          // Make sure that only one release can happen at a time.
          lock('release') {
            // As using the first milestone only would introduce a race 
            // condition (assume that the older build would enter the 
            // milestone first, but the lock second) and Jenkins does
            // not support inter-stage locks yet, we need a second 
            // milestone to make sure that older builds don't overwrite
            // newer ones.
            milestone ([ordinal:2, label:"Milestone: Performing Approved Release"])
    
            // Now do the actual work here.
            sh([script:"echo \"DO_RELEASE=$DO_RELEASE\"", label: 'Echo $DO_RELEASE'])
          }
        }
      }
  }
}
