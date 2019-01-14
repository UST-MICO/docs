Kubernetes Components
=====================

.. toctree::

    minikube
    azure
    istio
    knative

MICO runs and operates on Kubernetes.
This chapter describes different hosting environment and relevant third-party components like Istio and Knative.

Instructions in this section are meant to be executed with Bash in a Debian/Ubuntu like development environment. If you are using Windows 10, consider to use `Windows Subsystem for Linux (WSL)`_. Alternatives could be Cygwin_ or `Git Bash`_.

At first `Install and Set Up kubectl`_.

Install on Ubuntu / Debian:

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
