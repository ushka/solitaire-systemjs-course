stage 'CI'
node {

    //git branch: 'jenkins2-course', 
       // url: 'https://github.com/ushka/solitaire-systemjs-course'
    
    checkout scm

    // pull dependencies from npm
    sh 'npm install'

    // stash code & dependencies to expedite subsequent testing
    // and ensure same code & dependencies are used throughout the pipeline
    // stash is a temporary archive
    stash name: 'everything', 
          excludes: 'test-results/**', 
          includes: '**'
    
    // test with PhantomJS for "fast" "generic" results
    sh 'npm run test-single-run -- --browsers PhantomJS'
    
    // archive karma test results (karma is configured to export junit xml files)
    step([$class: 'JUnitResultArchiver', 
          testResults: 'test-results/**/test-results.xml'])
          
}

// Setup agent node
node('mac') {
    sh 'ls'
    sh 'rm -rf *'
    unstash 'everything' 
    sh 'ls'
}

// parallel integration testing
stage 'Browser Testing'
parallel chrome: {
    runTests("Chrome")
}, firefox: {
    runTests("Firefox")
}, safari: {
    runTests("Safari")
}

def runTests(browser) {
    node {
        sh 'rm -rf *'
        unstash 'everything'
        sh "npm run test-single-run -- --browsers ${browser}"
        step([$class: 'JUnitResultArchiver',
            testResults: 'test-results/**/test-results.xml']) 
    }
}

node {
    notify("Deploy to staging?")
}

input 'Deploy to staging?'

node {
  // write build number to index page so we can see this update
  sh "echo '<h1>${env.BUILD_DISPLAY_NAME}</h1>' >> app/index.html"
  // deploy to a docker container mapped to port 3000
  sh 'docker-compose up -d --build'
  notify 'Solitaire Deployed!'
}

def notify(status) {
    emailext (
      to: "redstar_selecta@msn.com",
      subject: "${status}: Job '${env.JOB_NAME} [${env.BUILD_NUMBER}]'",
      body: """<p>${status}: Job '${env.JOB_NAME} [${env.BUILD_NUMBER}]':</p>
        <p>Check console output at <a href='${env.BUILD_URL}'>${env.JOB_NAME} [${env.BUILD_NUMBER}]</a></p>""",
    )
}


