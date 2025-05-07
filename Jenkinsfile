pipeline {

    agent any

    environment {

        NETLIFY_SITE_ID = '2237487a-ebf2-4e48-b243-41c611efccae'
        NETLIFY_AUTH_TOKEN = credentials('netlify-token')
        REACT_APP_VERSION = "1.0.${BUILD_ID}"
        
    }

    stages {


        // Single line comment
        /* Multi-line comment
        if we want a shell command to be commented out in the shell script use # in front of it.
        eg. : #ls -la */

        stage('AWS') {
            
            agent {
                docker {
                    image 'amazon/aws-cli'
                    args "--entrypoint=''"
                }
            }
            environment {
                    #we can name it as per our requirement.
                    AWS_S3_BUCKET = "learn-jenkins-rva-28-04-2025"

            }

            steps {

                withCredentials([usernamePassword(credentialsId: 'my-aws', passwordVariable: 'AWS_SECRET_ACCESS_KEY', usernameVariable: 'AWS_ACCESS_KEY_ID')]) {
                 sh '''
                    aws --version
                    echo "Hello! S3, This is RVA" > index.html
                    aws s3 cp index.html s3://${AWS_S3_BUCKET}/index.html
                    aws s3 ls

                '''
                }  
            }
        }
        
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

                stage('E2E local') {

                    agent {

                        docker {
                            // Playwright docker image, link : https://playwright.dev/docs/docker
                            image 'my-playwright-image'
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
                            # serve has been installed globally using the Dockerfile
                            # & : this at the end , will start the server in the background and rest of the pipeline steps will be executed.
                            serve -s build &
                            sleep 10

                            npx playwright test --reporter=html
                        ''' 
                        
                    }


                    post {
                        always {                               
                            publishHTML([allowMissing: false, alwaysLinkToLastBuild: false, keepAll: false, reportDir: 'playwright-report', reportFiles: 'index.html', reportName: 'Playwright E2E local', reportTitles: '', useWrapperFileDirectly: true])   
                        }
                    }

          
                }
            }
        }


        stage('Deploy Staging & E2E Test') {

            agent {

                docker {
                        // Playwright docker image, link : https://playwright.dev/docs/docker
                        image 'my-playwright-image'
                        reuseNode true

                        /*args -u 'root:root' -> this command will start the container as a root user.
                        Dont do this because we are currently accessing the workspace as a user: aakash sharma, 
                        later it will throw error regarding workspace files access permission 
                        from aakash sharma user to root user. */
                    }
            }

            environment {

                CI_ENVIRONMENT_URL = 'STAGING_URL_TO_BE_SET'
            }
                            
            steps {

                //sh 'test -f build/index.html'
                sh '''

                    # 'netlify' & 'jq' has been installed globally using the Dockerfile
                    netlify --version
                    echo "Deploying to Staging. Site ID : $NETLIFY_SITE_ID"
            
                    # used to check the current authentication status and team information for the Netlify account we're logged into.
                    netlify status  

                    # Deploys the project to Netlify using the build directory as the source.
                    # The --json flag outputs the response in JSON format and saves it in deploy-output.json.
                    netlify deploy --dir=build --json > deploy-output.json
                    sleep 5

                    # Uses jq to parse the JSON output and extract the deploy_url field, which contains the deployed site's URL.
                    # The . (period) in .deploy_url is used in JSON parsing with jq (or jq) to specify the field that we want to extract from the JSON structure.
                    # The . (dot) before deploy_url means that we are accessing a top-level key in the JSON.
                    # jq -r '.deploy_url' deploy-output.json

                    CI_ENVIRONMENT_URL=$(jq -r '.deploy_url' deploy-output.json)
                    
                    npx playwright test --reporter=html
                    
                ''' 
                
            }

            post {
                always {                               
                    publishHTML([allowMissing: false, alwaysLinkToLastBuild: false, keepAll: false, reportDir: 'playwright-report', reportFiles: 'index.html', reportName: 'Playwright E2E staging', reportTitles: '', useWrapperFileDirectly: true])   
                }
            }

                
        }


        stage('Deploy Production & E2E Test') {

            agent {

                docker {
                    // Playwright docker image, link : https://playwright.dev/docs/docker
                    image 'my-playwright-image'
                    reuseNode true

                    /*args -u 'root:root' -> this command will start the container as a root user.
                    Dont do this because we are currently accessing the workspace as a user: aakash sharma, 
                    later it will throw error regarding workspace files access permission 
                    from aakash sharma user to root user. */
                }
            }

            environment {

                CI_ENVIRONMENT_URL = 'https://sensational-cobbler-19e2d0.netlify.app'
            }
                          
                    
            steps {

                //sh 'test -f build/index.html'
                sh '''

                    # 'netlify' & 'jq' has been installed globally using the Dockerfile
                    node --version
                    netlify --version

                    echo "Deploying to production. Site ID : $NETLIFY_SITE_ID"
                    
                    # used to check the current authentication status and team information for the Netlify account we're logged into.
                    netlify status

                    netlify deploy --dir=build --prod
                    sleep 5

                    echo "REACT_APP_VERSION : $REACT_APP_VERSION"

                    npx playwright test --reporter=html
                ''' 
                
            }


            post {
                always {                               
                    publishHTML([allowMissing: false, alwaysLinkToLastBuild: false, keepAll: false, reportDir: 'playwright-report', reportFiles: 'index.html', reportName: 'Playwright E2E Prod', reportTitles: '', useWrapperFileDirectly: true])   
                }
            }

          
        }


        
    }

    
}







