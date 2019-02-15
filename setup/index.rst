Setup
=====

.. toctree::

    setup
    aks
    ide
    cicd
    kubernetes/index

This chapter covers all information about the current MICO setup. The setup itself is described in :doc:`MICO Setup <setup>`.
The Azure Kubernetes Service (AKS) is the used for the current Kubernetes development environment and is described in :doc:`Azure Kubernetes Service <aks>`.

More details about different Kubernetes components that are used for MICO are described in section :doc:`Kubernetes <kubernetes/index>`.

All instructions are meant to be executed with Bash in a Debian/Ubuntu like development environment. If you are using Windows 10, consider to use `Windows Subsystem for Linux (WSL)`_. Alternatives could be Cygwin_ or `Git Bash`_.

At first `Install and Set Up kubectl`_.

Installation on Ubuntu / Debian:

.. code-block:: bash

   sudo apt-get update && sudo apt-get install -y apt-transport-https
   curl -s https://packages.cloud.google.com/apt/doc/apt-key.gpg | sudo apt-key add -
   echo "deb http://apt.kubernetes.io/ kubernetes-xenial main" | sudo tee -a /etc/apt/sources.list.d/kubernetes.list
   sudo apt-get update
   sudo apt-get install -y kubectl


.. _Windows Subsystem for Linux (WSL): https://docs.microsoft.com/en-us/windows/wsl/install-win10
.. _Cygwin: https://www.cygwin.com/
.. _Git Bash: https://git-scm.com/
.. _Install and Set Up kubectl: https://kubernetes.io/docs/tasks/tools/install-kubectl/
