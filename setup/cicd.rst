CI/CD Pipeline
==============

Describes setup of a Continous Integration and Continous Deployment Pipeline with Kubernetes, Jenkins, and DockerHub.

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

Create a Kubernetes cluster with *Azure Kubernetes Service (AKS)* like described in :doc:`setup/Azure Kubernetes Service <./aks>`.

Jenkins Setup
-------------
* Deploy a Jenkins VM through the Azure Marketplace
* Connect to the VM via ssh to update Jenkins and other applications: :bash:`ssh spethso@ust-mico-jenkins.westeurope.cloudapp.azure.com`
* Create Jenkins credentials to access DockerHub
* Create a Github Webhook: `<https://docs.microsoft.com/en-us/azure/aks/jenkins-continuous-deployment#create-a-github-webhook>`_
* To be able to run the integration tests, DockerHub credentials with Base64 encoding are required. Provide them as environment variables. For more information see :doc:`mico-core/tests/integration-tests`. Set them by adjusting `~/.bashrc`:
    .. code-block:: bash

        export DOCKERHUB_USERNAME_BASE64=*****
        export DOCKERHUB_PASSWORD_BASE64=*****
* Install BlueOcean Plugins for a better Pipeline UI
* Install Git Server Plugin, Git Plugin, Git Plugin for BlueOcean and GitHub Plugin for BlueOcean
* Under GitHub in Configuration add a new GitHub Server
    * API URL: `<https://api.github.com>`_
    * Provide your GitHub Access Token as credentials
    * Enable manage hooks
* Create a new Jenkins Pipeline project: `<https://jenkins.io/doc/book/pipeline/>`_
    * Enable "GitHub hook trigger for GITScm polling"
    * Provide Pipeline definition as a "Pipeline script from SCM"
        * Use "Git" and add the repository URL to `<https://github.com/UST-MICO/mico>`_
        * Set the script path to :bash:`Jenkinsfile`
        * Set leightweight checkout to only checkout the Jenkinsfile and not the complete repository
        
Pipeline using the Jenkinsfile
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
The Jenkinsfile is in the root folder of the repository and contains the pipeline stages which are executed.
The pipeline is triggered on a push on the master branch.

* Checkout Stage
    * This stage performs the checkout of the repository
* Docker Build Stage
    * mico-core and mico-admin Dockerimages are built in parallel
* Unit test stage
    * There is an own Dockerfile for unittests which is built in this stage
    * The Dockerimage runs the unittests using maven
* Integration test stage
    * There is an own Dockerfile for integration tests which is built in this stage
    * The Dockerimage runs the integrationtests using maven
* Push image stage
    * In this stage the Dockerimages for mico-core and mico-admin are pushed to Dockerhub, both one time with :bash:`kube` and the build number and one time with :bash:`latest` tag
* Deploy on Kubernetes stage
    * mico-core and mico-admin are deployed in parallel on Kubernetes using the :bash:`kubectl set image`
* Docker clean up stage
    * All Dockerimages older than 10 days are deleted in this stage
* Post actions
    * After the normale pipeline stages the build status is sent to the ust-mico slack workspace.

Adjust heap size of JRE
~~~~~~~~~~~~~~~~~~~~~~~

* Open the file `/etc/default/jenkins`
* Search for `JAVA_ARGS= '-Xmx256m'` (default)
* Remove the `#` to uncomment the line
* Adjust the size to the desired value
* Example:
    .. code-block:: bash

        JAVA_ARGS="-Xmx3g"

Travis CI
---------
* To check pull requests before merge into master there is a Travis CI which builds everything with Maven.
    * Travis runs unit tests, but no integration tests
    * At the current state, Travis CI builds mico-core and mico-admin, but runs only unit tests for mico-core
* URL `<https://travis-ci.org/UST-MICO/mico>`_
