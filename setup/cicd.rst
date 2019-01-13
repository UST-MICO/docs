Continous Integration & Continous Deployment
============================================

Describes setup of a Continous Integration and Continous Deployment Pipeline with Kubernetes, Jenkins, and Docker hosted on Microsoft Azure. 

.. role:: bash(code)
    :language: bash

Prerequisites
-------------

* Install Docker (on Ubuntu: `<https://docs.docker.com/install/linux/docker-ce/ubuntu/>`_ )
* Install unzip :bash:`sudo apt-get install unzip`
* Install Maven
    * :bash:`sudo mkdir /opt/maven/`
    * :bash:`cd /opt/maven/`
    * :bash:`wget http://ftp.wayne.edu/apache/maven/maven-3/<VERSION>/binaries/apache-maven-<VERSION>-bin.zip`
    * :bash:`unzip apache-maven-<VERSION>`
    * :bash:`sudo nano /etc/profile.d/maven.sh`
    * Add the following lines to maven.sh:
        .. code-block:: bash
        
            export JAVA_HOME=<PATH_TO_YOUR_JAVA_HOME>
            export M2_HOME=/opt/maven
            export MAVEN_HOME=/opt/maven
            export PATH=${M2_HOME}/bin:${PATH}
    * :bash:`sudo source /etc/profile.d/maven.sh`

Kubernetes Setup
----------------

Create *Azure Kubernetes Service (AKS)* and *Azure Container Registry (ACR)* like described in :doc:`Kubernetes cluster <./kubernetes-cluster>`.

Jenkins Setup
-------------
* Deploy a Jenkins VM through the Azure Marketplace
* Create Jenkins environment variable to hold ACR login server with the name: ACR_LOGINSERVER (`<https://docs.microsoft.com/en-us/azure/aks/jenkins-continuous-deployment#create-a-jenkins-environment-variable>`_)
* Create Jenkins credentials to access ACR: `<https://docs.microsoft.com/en-us/azure/aks/jenkins-continuous-deployment#create-a-jenkins-credential-for-acr>`_
* Create a new Jenkins project: `<https://docs.microsoft.com/en-us/azure/aks/jenkins-continuous-deployment#create-a-jenkins-project>`_
* Create a Github Webhook: `<https://docs.microsoft.com/en-us/azure/aks/jenkins-continuous-deployment#create-a-github-webhook>`_
* Push the Docker image(s) to Azure Container Registry (ACR) manually (only first time)
* To be able to run the integration tests, DockerHub credentials with Base64 encoding are required. Provide them as environment variables. For more information see :doc:`mico-core/tests/integration-tests`. Set them by adjusting `~/.bashrc`:
    .. code-block:: bash

        export DOCKERHUB_USERNAME_BASE64=*****
        export DOCKERHUB_PASSWORD_BASE64=*****
* Add the following build scripts:
    * Build multi-module MICO project with maven: :bash:`mvn clean compile package -B -DskipTests`
    * Execute unit tests with maven: :bash:`mvn test`
    * Execute integration tests with maven: :bash:`mvn failsafe:integration-test`
    * [Optional] Stop the build if an integration test fails: :bash:`mvn verify`
    * Build and push MICO-Core Docker image: 
        .. code-block:: bash

            ACR_IMAGE_NAME="${ACR_LOGINSERVER}/mico-core:kube${BUILD_NUMBER}"
            docker build -t $ACR_IMAGE_NAME -f Dockerfile.mico-core .
            docker login ${ACR_LOGINSERVER} -u ${ACR_ID} -p ${ACR_PASSWORD}
            docker push $ACR_IMAGE_NAME
            DOCKERHUB_IMAGE_NAME="ustmico/mico-core:latest"
            docker tag $ACR_IMAGE_NAME $DOCKERHUB_IMAGE_NAME
            docker login -u ${DOCKERHUB_USERNAME} -p ${DOCKERHUB_PASSWORD}
            docker push $DOCKERHUB_IMAGE_NAME

    * Deploy MICO-Core to Kubernetes:
        .. code-block:: bash

            ACR_IMAGE_NAME="${ACR_LOGINSERVER}/mico-core:kube${BUILD_NUMBER}"
            kubectl set image deployment/mico-core mico-core=$ACR_IMAGE_NAME --kubeconfig /var/lib/jenkins/config

    * Build and push MICO-Frontend Docker image:
        .. code-block:: bash

            ACR_IMAGE_NAME="${ACR_LOGINSERVER}/mico-admin:kube${BUILD_NUMBER}"
            docker build -t $ACR_IMAGE_NAME -f Dockerfile.mico-admin .
            docker login ${ACR_LOGINSERVER} -u ${ACR_ID} -p ${ACR_PASSWORD}
            docker push $ACR_IMAGE_NAME
            DOCKERHUB_IMAGE_NAME="ustmico/mico-admin:latest"
            docker tag $ACR_IMAGE_NAME $DOCKERHUB_IMAGE_NAME
            docker login -u ${DOCKERHUB_USERNAME} -p ${DOCKERHUB_PASSWORD}
            docker push $DOCKERHUB_IMAGE_NAME

    * Deploy MICO-Frontend to Kubernetes:
        .. code-block:: bash

            ACR_IMAGE_NAME="${ACR_LOGINSERVER}/mico-admin:kube${BUILD_NUMBER}"
            kubectl set image deployment/mico-admin mico-admin=$ACR_IMAGE_NAME --kubeconfig /var/lib/jenkins/config

* Add the following post-build actions:
    * `Discard Old Builds` (Plugin `Discard Old Build <https://wiki.jenkins.io/display/JENKINS/Discard+Old+Build+plugin>`_ required)
        
        * `Max # of builds to keep`: 10 (or similar)
        * `Status to discard`: Check `Unstable` + `Failure`   

Adjust heap size of JRE
~~~~~~~~~~~~~~~~~~~~~~~

* Open the file `/etc/default/jenkins`
* Search for `JAVA_ARGS= '-Xmx256m'` (default)
* Remove the `#` to uncomment the line
* Adjust the size to the desired value
* Example:
    .. code-block:: bash

        JAVA_ARGS="-Xmx3g"
