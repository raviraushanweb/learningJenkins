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
