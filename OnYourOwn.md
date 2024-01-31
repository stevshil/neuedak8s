# Continue Practicing

> &#x26a0;&#xfe0f; **NOTE:** Do **NOT** do any of this on company equipment.  Make sure you are using your own computer, and are happy to install the software.

> &#x26a0;&#xfe0f; **Disclaimer:** We do not take any responsiblity for damage to equipment or systems going wrong.  You follow these instructions at your own risk.

You don't need a big environment to practice using Kuberentes.

Here's what you need:

1. A PC with a minimum 16GB RAM and at least 20GB spare disk space.
2. Install **minikube**
    You can install it natively on Linux, Mac and Windows, see [https://minikube.sigs.k8s.io/docs/start/](https://minikube.sigs.k8s.io/docs/start/)
3. Alternatively if you don't want to install it natively you could use [VirtualBox](https://www.virtualbox.org/) which is a virtualisation software that works on Linux, Mac and Windows.
    If you use VirtualBox then you should install a Linux OS as the virtual machine, and follow the Linux installation of Minikube.

## Installing minikube on Linux

Run the following commands:

1. Download minikube executable

    ```bash
        curl -LO https://storage.googleapis.com/minikube/releases/latest/minikube-linux-amd64
    ```

2. Move it to a known locaiton

    ```bash
    sudo install minikube-linux-amd64 /usr/local/bin/minikube
    ```

3. Launch minikube as you did in at the beginning of this lab and away you go.