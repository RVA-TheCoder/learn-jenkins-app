pipeline {
    agent any

    stages {
        // Single line comment
        /* Multi-line comment
        if we want a shell command to be commented out in the shell script use # in front of it.
        eg. : #ls -la */

        
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

        stage('Test') {
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
        }
        

    }

    
}







