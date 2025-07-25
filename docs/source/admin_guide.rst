===================
Administrator Guide
===================

This section provides guidance for administrators managing the BETIF/DIFAET system. It covers installation, configuration, and maintenance tasks to ensure the system runs smoothly.

-----------------------------------------------
Kernel-based Virtual Machine (KVM) Installation
-----------------------------------------------

The BETIF-DIFAET machine runs AlmaLinux 9 as its operating system, and, as is common across many Linux distributions, it supports Kernel-based Virtual Machines (KVM) for virtualizing bare-metal 
resources. KVM is a full virtualization solution for Linux on x86 hardware containing virtualization extensions (Intel VT or AMD-V). This tool allows to run multiple virtual machines running 
unmodified Linux or Windows images. Each virtual machine has private virtualized hardware: a network card, disk, graphics adapter, etc.

In order to test the steps to create the computing platform, 4 VMs have been spawned based on AlmaLinux 9 as Operating system and 16 vCPUs, 32 GB of RAM and 80 GB of disk space, through 
the ``virt-install`` interface.

:numref:`create-master-node-vm` can be used to create a virtual machine acting as a master node for the BETIF-DIFAET system.

.. code-block:: bash
    :caption: : Script to create a master node VM for BETIF-DIFAET
    :name: create-master-node-vm

    #!/usr/bin/env bash
    vm_name='alma9-test-master'
    vm_memory='32768'
    vm_cpus='16'
    vm_disk='/var/lib/libvirt/images/AlmaLinux-9-GenericCloud-latest.x86_64.qcow2'

    ci_user_data='user-data'
    ci_network_config='network-configv3'
    qemu-img create -f qcow2 \
        -b /var/lib/libvirt/images/AlmaLinux-9-GenericCloud-latest.x86_64.qcow2 \
        -F qcow2 /var/lib/libvirt/images/rke2-master-AlmaLinux-9-test-master.qcow2 80G

    vm_disk='/var/lib/libvirt/images/rke2-master-AlmaLinux-9-test-master.qcow2'

    virt-install \
        --connect qemu:///system \
        --name "$vm_name" \
        --memory "$vm_memory" \
        --machine q35 \
        --vcpus "$vm_cpus" \
        --cpu host-passthrough \
        --import \
        --cloud-init user-data="$ci_user_data" \
        --osinfo name=almalinux9 \
        --disk "$vm_disk" \
        --virt-type kvm \
        --network network=private-net \
        --network network=default \
        --noautoconsole

:numref:`create-worker-node-vm` can be used to create a virtual machine acting as a worker node for the BETIF-DIFAET system.

.. code-block:: bash
    :caption: : Script to create a worker node VM for BETIF-DIFAET
    :name: create-worker-node-vm

    #!/usr/bin/env bash

    if [ -z "$1" ]; then
        echo "Usage: $0 <worker-number>"
        exit 1
    fi

    N="$1"
    vm_name="alma9-test-worker-$N"
    vm_memory='32768'
    vm_cpus='16'
    vm_base_disk='/var/lib/libvirt/images/AlmaLinux-9-GenericCloud-latest.x86_64.qcow2'

    ci_user_data='user-data'
    ci_network_config='network-configv3'

    vm_disk="/var/lib/libvirt/images/rke2-master-AlmaLinux-9-test-worker-$N.qcow2"

    qemu-img create -f qcow2 -b "$vm_base_disk" -F qcow2 "$vm_disk" 80G

    virt-install \
        --connect qemu:///system \
        --name "$vm_name" \
        --memory "$vm_memory" \
        --machine q35 \
        --vcpus "$vm_cpus" \
        --cpu host-passthrough \
        --import \
        --cloud-init user-data="$ci_user_data" \
        --osinfo name=almalinux9 \
        --disk "$vm_disk" \
        --virt-type kvm \
        --network network=private-net \
        --noautoconsole    

A :ref:`private-network-interface` was also created to enable direct connections between the different VMs which for debug and testing purposes is still left open to allow direct access to the 
worker nodes. In the actual deployment this network will block access to the worker VMs, leaving only the Master accessible via SSH.

.. code-block:: xml
    :caption: : Private network interface
    :name: private-network-interface

    <network>
      <name>private-net</name>
      <forward mode='nat'/>
      <bridge name="virbr1"/>
      <ip address="10.10.142.1" netmask="255.255.255.0">
        <dhcp>
          <range start="10.10.142.100" end="10.10.142.200"/>
        </dhcp>
      </ip>
    </network>

--------------------------------
Turning the VMs in a K8s cluster
--------------------------------

With KVM, it was possible to set up 4 different VMs with a chosen flavour in terms of virtualized hardware and software. The next step to provide a computing platform is to setup a Kubernetes 
Cluster which will allow to host all the services needed for the research environment, from authentication to end-user tools for launching code.

To streamline the process of setting up the cluster, RKE2, also known as Rancher Kubernetes Engine 2 [RKE2]_, was used. It is a Kubernetes distribution developed by Rancher (now part of SUSE) that 
emphasizes security, stability, and ease of deployment. It's designed to be fully Kubernetes-conformant, meaning it behaves in accordance with the official Kubernetes standards and APIs, making it 
compatible with standard Kubernetes tooling and workloads.

RKE2 is packaged as a single binary, which simplifies installation and maintenance. This binary includes everything needed to run a Kubernetes node, including the container runtime (which is containerd, 
rather than Docker), as well as the control plane and networking components. This design eliminates many of the dependencies and complexities found in traditional Kubernetes setups.

:numref:`install-rke-master` can be used to install RKE2 on the master node:

.. code-block:: bash
    :caption: : Script to install RKE2 on the master node
    :name: install-rke-master

    mkdir -p /etc/rancher/rke2/
    echo """
    tls-san:
      - 192.168.122.59
      - 10.10.142.115
    """ > /etc/rancher/rke2/config.yaml
    curl -sfL https://get.rke2.io | sh -
    systemctl enable rke2-server.service
    systemctl start rke2-server.service
    sudo cp /etc/rancher/rke2/rke2.yaml /home/clouduser/
    sudo chown clouduser /home/clouduser/rke2.yaml
    export KUBECONFIG=/home/clouduser/rke2.yaml

:numref:`install-rke-worker` can be used to install RKE2 on the worker node:

.. code-block:: bash
    :caption: : Script to install RKE2 on the worker node
    :name: install-rke-worker

    #!/bin/bash
    mkdir -p /etc/rancher/rke2/
    echo """
    server: https://10.10.142.115:9345
    token: K10e67c7985e7db4f9ed9b0353ae10f53c179a51eb4ed8443ca8596873a3327188d::server:c5cb82b52a5650b010f9e3e5f6e76b52
    node-name: worker-1
    """ > /etc/rancher/rke2/config.yaml
    curl -sfL https://get.rke2.io | INSTALL_RKE2_TYPE="agent" sh -
    systemctl enable rke2-agent.service
    systemctl start rke2-agent.service

Where ``node-name`` is the name of the worker node, which can be set to ``worker-1``, ``worker-2``, etc. depending on the number of worker nodes in the cluster.

Using RKE2, the computing architecture shown in :numref:`betif-arch` was built:

* The bare-metal server stands as foundation for all virtualization layers above;
* The Kubernetes cluster is made up of three Virtual Machines created using KVM and can comunicate through a private network;
* The Master node is accessible from the host.

.. _betif-arch:

.. figure:: betif_arch.png
   :alt: BETIF-DIFAET architecture

   Schematic of the BETIF-DIFAET architecture.

-----------------------------------
Deploying the BETIF-DIFAET platform
-----------------------------------

IN PROGRESS

----------
References
----------

.. [RKE2] https://docs.rke2.io/