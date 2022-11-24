# Jenkins Note

## Official Links

-   [Jenkins website](https://www.jenkins.io/)
-   [Jenkins documentation](https://www.jenkins.io/doc/book/)
-   [Jenkins plugins](https://plugins.jenkins.io/)

## Install Jenkins on Mac

-   Step 1: Installing the Jenkins using brew
    ```
    brew install jenkins-lts
    ```
-   Step 2: Start the Jenkins server
    ```
    brew services start jenkins-lts
    ```
-   Other commands
    -   Restart the jenkins service
        ```
        brew services restart jenkins-lts
        ```
    -   Update the jenkins version
        ```
        brew upgrade jenkins-lts
        ```
-   Step 3: By default it runs on port 8080, open browser and go to [http://localhost:8080/](http://localhost:8080/)

## Jenkins declarative: [official docs](https://www.jenkins.io/doc/book/pipeline/syntax/)

-   the code must be enclosed within a _pipeline_ block
    ```
    pipeline {
      /* insert declarative code here */
    }
    ```
-   Top level agent, the options are invoked after entering the agent. Ex, when using timeout it will be only applied to the execution within the agent
    ```
    node("myAgent") {
      timeout(unit: 'SECONDS', time: 5) {
          stage("One"){
              sleep 10
              echo 'hello'
          }
      }
    }
    ```
-   Stage agents, the options are invoked before entering the agent and before checking any when conditions. In this case, when using timeout, it is applied before the agent is allocated.
    ```
      timeout(unit: 'SECONDS', time: 5) {
      stage("One"){
          node {
              sleep 10
              echo 'Hello'
          }
      }
    }
    ```
-   Parameters for agent
    -   any
    -   none
    -   label
    -   node
    -   docker
        ```
        agent {
            docker {
                image 'maven:3.8.1-adoptopenjdk-11'
                label 'my-defined-label'
                args '-v /tmp:/tmp'
            }
        }
        ```
        ```
            agent {
                docker {
                    image 'myregistry.com/node'
                    label 'my-defined-label'
                    registryUrl 'https://myregistry.com/'
                    registryCredentialsId 'myPredefinedCredentialsInJenkins'
                }
            }
        ```
    -   dockerfile: the Jenkinsfile must be loaded from either a Multibranch Pipeline or a Pipeline from SCM
        ```
            agent {
                // Equivalent to "docker build -f Dockerfile.build --build-arg version=1.0.2 ./build/
                dockerfile {
                    filename 'Dockerfile.build'
                    dir 'build'
                    label 'my-defined-label'
                    additionalBuildArgs  '--build-arg version=1.0.2'
                    args '-v /tmp:/tmp'
                }
            }
        ```
    -   kubernetes: the Jenkinsfile must be loaded from either a Multibranch Pipeline or a Pipeline from SCM
        ```
            agent {
                kubernetes {
                    defaultContainer 'kaniko'
                    yaml '''
            kind: Pod
            spec:
            containers:
            - name: kaniko
                image: gcr.io/kaniko-project/executor:debug
                imagePullPolicy: Always
                command:
                - sleep
                args:
                - 99d
                volumeMounts:
                - name: aws-secret
                    mountPath: /root/.aws/
                - name: docker-registry-config
                    mountPath: /kaniko/.docker
            volumes:
                - name: aws-secret
                secret:
                    secretName: aws-secret
                - name: docker-registry-config
                configMap:
                    name: docker-registry-config
            '''
            }
        ```
-   **Post Section**
    -   The post section defines one or more additional steps that are run upon the completion of a Pipeline's or stage's run (depending on the location of the post section within the Pipeline). The condtion blocks are executed in the order shown below.
    -   _always_: Run the steps in the post section regardless of the completion status of the Pipelins' or stage's run
    -   _changed_: Only run the steps in post if the current Pipeline's run has a different completion status from its previous run.
    -   _fixed_: Only run the steps in post if the current Pipeline's run is successful and the previous run failed or was unstable.
    -   _regression_: Only run the steps in post if the current Pipeline's or status is failure, unstable, or aborted and the previous run was successful.
    -   _aborted_: Only run the steps in post if the current Pipeline’s run has an "aborted" status, usually due to the Pipeline being manually aborted. This is typically denoted by gray in the web UI.
    -   _failure_: Only run the steps in post if the current Pipeline’s or stage’s run has a "failed" status, typically denoted by red in the web UI.
    -   _success_: Only run the steps in post if the current Pipeline’s or stage’s run has a "success" status, typically denoted by blue or green in the web UI.
    -   _unstable_: Only run the steps in post if the current Pipeline’s run has an "unstable" status, usually caused by test failures, code violations, etc. This is typically denoted by yellow in the web UI.
    -   _unsuccessful_: Only run the steps in post if the current Pipeline’s or stage’s run has not a "success" status. This is typically denoted in the web UI depending on the status previously mentioned (for stages this may fire if the build itself is unstable).
    -   _cleanup_: Run the steps in this post condition after every other post condition has been evaluated, regardless of the Pipeline or stage’s status.
    ```
    pipeline {
        agent any
        stages {
            stage('Example') {
                steps {
                    echo 'Hello World'
                }
            }
        }
        post {
            always {
                echo 'I will always say Hello again!'
            }
        }
     }
    ```
-   **stages**
    -   Containing a sequence of one or more stage directives, the stages section is where the bulk of the work described by a Pipeline will be located. At a minimum, it is recommended that stages contain at lead one stage directive for each discrete part of the CD process, such as Build, Test, and Deploy.
    ```
    pipeline {
        agent any
        stages {
            stage('Example') {
                steps {
                    echo 'Hello World'
                }
            }
        }
    }
    ```
-   **steps**
    -   The steps section defines a series of one or more steps to be executed in a given stage directive.
    ```
    pipeline {
        agent any
        stages {
            stage('Example') {
                steps {
                    echo 'Hello World'
                }
            }
        }
    }
    ```
-   **Directives**
    -   **environment**
        -   The environment directive specifies a sequence of key-value pairs which will be defined as environment variables for all steps, or stage-specific steps, depending on where the environment directive is located within the Pipeline.  
            This directive supports a special helper method credentials() which can be used to access pre-defined Credentials by their identifier in the Jenkins environment.
        -   Supported Credentials Type
            -   _Secret Text_: the environment variable specified will be set to the Secret Text content
            -   _Secret File_: the environment variable specified will be set to the location of the File file that is temporarily created
            -   _Username and password_: the environment variable specified will be set to username:password and two additional environment variables will be automatically defined: MYVARNAME_USR and MYVARNAME_PSW respectively.
            -   _SSH with Private Key_: the environment variable specified will be set to the location of the SSH key file that is temporarily created and two additional environment variables may be automatically defined: MYVARNAME_USR and MYVARNAME_PSW (holding the passphrase).
        ```
        pipeline {
            agent any
            environment {
                CC = 'clang'
            }
            stages {
                stage('Example') {
                    environment {
                        AN_ACCESS_KEY = credentials('my-predefined-secret-text')
                    }
                    steps {
                        sh 'printenv'
                    }
                }
            }
        }
        ```
        ```
        pipeline {
            agent any
            stages {
                stage('Example Username/Password') {
                    environment {
                        SERVICE_CREDS = credentials('my-predefined-username-password')
                    }
                    steps {
                        sh 'echo "Service user is $SERVICE_CREDS_USR"'
                        sh 'echo "Service password is $SERVICE_CREDS_PSW"'
                        sh 'curl -u $SERVICE_CREDS https://myservice.example.com'
                    }
                }
                stage('Example SSH Username with private key') {
                    environment {
                        SSH_CREDS = credentials('my-predefined-ssh-creds')
                    }
                    steps {
                        sh 'echo "SSH private key is located at $SSH_CREDS"'
                        sh 'echo "SSH user is $SSH_CREDS_USR"'
                        sh 'echo "SSH passphrase is $SSH_CREDS_PSW"'
                    }
                }
            }
        }
        ```
-   **options**
    -   Available Options
        -   buildDiscarder  
            ex: options { buildDiscarder(logRotator(numToKeepStr: '1')) }
        -   checkoutToSubdirectory  
            ex: options { checkoutToSubdirectory('foo') }
        -   disableConcurrentBuilds  
            ex: options { disableConcurrentBuilds() }
        -   disableResume  
            ex: options { disableResume() }
        -   newContainerPerStage
        -   overrideIndexTriggers
        -   preserveStashes  
            ex: options { preserveStashes() }  
            ex: options { preserveStashes(buildCount: 5) }
        -   quietPeriod  
            ex: options { quietPeriod(30) }
