pipeline {

    agent any

    environment {

        NETLIFY_SITE_ID = '2237487a-ebf2-4e48-b243-41c611efccae'
        NETLIFY_AUTH_TOKEN = credentials('netlify-token')
    }

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

                    #used to install dependencies in a Node.js project, it installs the packages from package-lock.json
                    npm ci

                    npm run build
                
                '''
            }
        } 
        */

        stage("Tests") {

            parallel {

                stage('Units tests') {
                    agent {
                        docker {
                            image 'node:18-alpine'
                            reuseNode true
                        }
                    }

                    steps {

                
                        sh '''
                            #echo 'Inside Test stage'
                            # test -f build/index.html   
                            
                            test -f build/index.html && echo "File exists"
                            npm test
                        '''
                    }

                    post {
                        always {
                            junit 'jest-results/junit.xml' 
                                   
                        }
                    }


                }

                stage('E2E') {

                    agent {

                        docker {
                            // Playwright docker image, link : https://playwright.dev/docs/docker
                            image 'mcr.microsoft.com/playwright:v1.49.1-noble'
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

                            npx playwright test --reporter=html
                        ''' 
                        
                    }


                    post {
                        always {                               
                            publishHTML([allowMissing: false, alwaysLinkToLastBuild: false, keepAll: false, reportDir: 'playwright-report', reportFiles: 'index.html', reportName: 'Playwright HTML Report', reportTitles: '', useWrapperFileDirectly: true])   
                        }
                    }

          
                }
            }
        }

        stage('Deploy') {

            agent {
                docker {
                    image 'node:18-alpine'
                    reuseNode true
                }
            }
            steps {
                sh '''

                    # we are installing netlify as a local project dependency
                    npm install netlify-cli

                    node_modules/.bin/netlify --version

                    echo "Deploying to production. Site ID : $NETLIFY_SITE_ID"
                    
                    node_modules/.bin/netlify status

                    node_modules/.bin/netlify deploy --dir=build --prod

                '''
            }
        }
        
    }

    
}







