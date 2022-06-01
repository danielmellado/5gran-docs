..
        This work is licensed under a Creative Commons Attribution 3.0 Unported
      License.

      http://creativecommons.org/licenses/by/3.0/legalcode

      Convention for heading levels in Neutron devref:
      =======  Heading 0 (reserved for the title in a document)
      -------  Heading 1
      ~~~~~~~  Heading 2
      +++++++  Heading 3
      '''''''  Heading 4
      (Avoid deeper levels because they do not render well.)

=====================================================
Virtual Developer Environment - SNO Hub Cluster (1:0)
=====================================================

We'll be creating a virtualized cluster containing:

* Single Node OpenShift (SNO) cluster, as hub cluster, including ACM, GitOps, BMO, and TALO operators.
* SNO cluster, managed by ACM as spoke.

In order to do so, we'll be relying on kcli as a tool to handle the creation
and configuration of the VMs.

.. note::

  TODO: You'll be in need of a server to handle this, as its memory requirements
  would be probably greater than your laptops. You should consider at least
  64GBi of ram for the SNO and 32Gbi for the spoke one.


Prerequisites
~~~~~~~~~~~~~

* Install kcli https://github.com/karmab/kcli.git
* Clone locally https://github.com/leo8a/ztp-for-ran-samples
* Install Ansible https://docs.ansible.com/ansible/latest/installation_guide/index.html


Lab Architecture
~~~~~~~~~~~~~~~~

.. code-block:: console

    +---------------+--------+-----------------+----------------------------------------------------+-------------+---------+
    |      Name     | Status |        Ip       |                       Source                       |     Plan    | Profile |
    +---------------+--------+-----------------+----------------------------------------------------+-------------+---------+
    |  hub-master-0 |   up   | 192.168.122.101 | rhcos-410.84.202201251210-0-openstack.x86_64.qcow2 | ztp-ran-lab |  kvirt  |
    | hub-spoke-sno |   up   |                 |                                                    | ztp-ran-lab |  kvirt  |
    +---------------+--------+-----------------+----------------------------------------------------+-------------+---------+

Once the lab is deployed, the kubeconfig file will be available on your
`~/.kube/config <https://github.com/leo8a/ztp-for-ran-samples/blob/master/ran-ztp-workflow/bootstrap_ran_ztp_lab.sh#L20>`_
path.

You may access the `hub-master-0` machine, at any time, by using:

.. code-block:: console

   $ kcli ssh hub-master-0

And the `hub-spoke-sno` machine with a `fixed IP address <https://github.com/leo8a/ztp-for-ran-samples/blob/sushy-siteconfig/siteconfig/sushy-spoke-sno.yaml#L45-L56>`_,
once has booted up, by using:

.. code-block:: console

   $ ssh core@192.168.122.30


Parameters file
~~~~~~~~~~~~~~~

KCLI needs a parameters file in order to run the installation. To do so, we
provided a `kcli_parameters.yml <https://github.com/leo8a/ztp-for-ran-samples/blob/master/ran-ztp-workflow/ztp-ran-plan.yml#L1-L19>`_
file in the `ztp-for-ran-samples` directory. As a complement, below you can see
default parameters already.

.. code-block:: yaml

    parameters:
      cluster: hub
      version: stable
      network: default
      tag: '4.10'
      masters: 1
      workers: 0
      memory: 65536
      numcpus: 32
      disk_size: 100
      pull_secret: /root/pull_secret.json
      apps:
      - users
      - odf-lvm-operator
      - openshift-gitops-operator
      - advanced-cluster-management

* On the lab server, download a copy of your pull secret file and put it under the specified path `/root/pull_secret.json`.


Create cluster
~~~~~~~~~~~~~~

From the `ran-ztp-workflow` directory, run:

.. code-block:: console

   $ ./bootstrap_ran_ztp_lab.sh


Debug installation process
~~~~~~~~~~~~~~~~~~~~~~~~~~

Once you run the bootstrapping script, just follow the installation process and
debug issues by following the standard output of in the provided script. Take
not that this may take a while to complete.

Additionally, you may debug issues also using the Hub console and/or the ArgoCD
dashboard. To do so, just connect to the installation using `shuttle <https://github.com/sshuttle/sshuttle>`_
and navigate to the following URL:

- SNO Hub Console: https://console-openshift-console.apps.hub.karmalabs.com
- RHACM Console: https://multicloud-console.apps.hub.karmalabs.com
- ArgoCD Dashboard: https://openshift-gitops-server-openshift-gitops.apps.hub.karmalabs.com

.. code-block:: console

   $ sshuttle -r root@<remote_lab_host> 192.168.122.0/24
