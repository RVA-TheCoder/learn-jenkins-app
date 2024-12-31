pipeline {
    agent any

    stages {
        // Single line comment
        /* Multi-line comment
        if we want a shell command to be commented out in the shell script use # in front of it.
        eg. : #ls -la */

        /*
        stage('Build') {

            agent {
                docker {
                    image 'node:18-alpine'
                    reuseNode true
                }
            }
            steps {
                sh '''
                    ls -la
                    node --version
                    npm --version
                    npm ci
                    npm run build
                
                '''
            }
        }
        */ 
        stage ("Test") {

            agent {
                docker {
                    image 'node:18-alpine'
                    reuseNode true
                }
            }
            
            steps{

                //sh 'test -f build/index.html'
                sh '''
                    test -f build/index.html && echo "File exists"
                    npm test
                ''' 
                         
            }
        }

        stage ("E2E") {

            agent {
                docker {
                    // Playwright docker image, link : https://playwright.dev/docs/docker
                    image 'mcr.microsoft.com/playwright:v1.39.0-noble'
                    reuseNode true

                    /*args -u 'root:root' -> this command will start the container as a root user.
                    Dont do this because we are currently accessing the workspace as a user: aakash sharma, 
                    later it will throw error regarding workspace files access permission 
                    from aakash sharma user to root user. */
                }
            }
            
            steps{

                //sh 'test -f build/index.html'
                sh '''
                    # We can also install serve as a local dependency to our project.
                    # npm install serve : serve will be installed as a local NPM dependency, so that we dont have access permission error.
                    npm install serve

                    # & : this at the end , will start the server in the background and rest of the pipeline steps will be executed.
                    node_modules/.bin/serve -s build &
                    sleep 10

                    npx playwright test
                ''' 
                         
            }
        }

    }

    post {
        always {
            junit 'test-results/junit.xml'
        }
    }
}







