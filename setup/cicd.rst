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
* Create Jenkins credentials to access DockerHub
* Create a Github Webhook: `<https://docs.microsoft.com/en-us/azure/aks/jenkins-continuous-deployment#create-a-github-webhook>`_
* To be able to run the integration tests, DockerHub credentials with Base64 encoding are required. Provide them as environment variables. For more information see :doc:`mico-core/tests/integration-tests`. Set them by adjusting `~/.bashrc`:
    .. code-block:: bash

        export DOCKERHUB_USERNAME_BASE64=*****
        export DOCKERHUB_PASSWORD_BASE64=*****
* Create a new Jenkins Pipeline project: `<https://jenkins.io/doc/book/pipeline/>`_
    * Enable "GitHub hook trigger for GITScm polling"
    * Provide Pipeline definition as a "Pipeline script from SCM"
        * Use "Git" and add the repository URL to `<https://github.com/UST-MICO/mico>`_
        * Set the script path to :bash:`Jenkinsfile`

Adjust heap size of JRE
~~~~~~~~~~~~~~~~~~~~~~~

* Open the file `/etc/default/jenkins`
* Search for `JAVA_ARGS= '-Xmx256m'` (default)
* Remove the `#` to uncomment the line
* Adjust the size to the desired value
* Example:
    .. code-block:: bash

        JAVA_ARGS="-Xmx3g"
