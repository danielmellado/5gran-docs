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

=========================================================
Virtual Developer Environment - Compact Hub Cluster (3:0)
=========================================================

We'll be creating a virtualized cluster containing:

* 3-Master OpenShift, including ACM, as hub cluster.
* SNO cluster, managed by ACM as spoke.

In order to do so, we'll be relying on kcli as a tool to handle the creation
and configuration of the VMs.

.. note::

  You'll be in need of a server to handle this, as its memory requirements
  would be probably greater than your laptops. You should consider at least
  16GBi of ram for every node and 32Gbi for the spoke and lab-installer ones.


Prerequisites
~~~~~~~~~~~~~

* Install kcli https://github.com/karmab/kcli.git
* Clone locally https://github.com/karmab/kcli-openshift4-baremetal
* On the `kcli-openshift4-baremetal` directory, download a copy of your pull
  secret file and name it as `openshift_pull.json`


Lab Architecture
~~~~~~~~~~~~~~~~

.. code-block:: console

   +------------------------+--------+----------------------+------------------------------------------------------+------+---------+
   |          Name          | Status |         Ips          |                        Source                        | Plan | Profile |
   +------------------------+--------+----------------------+------------------------------------------------------+------+---------+
   |     lab-installer      |   up   | 2620:52:0:1302::978d | CentOS-Stream-GenericCloud-8-20210603.0.x86_64.qcow2 | lab  |  kvirt  |
   |      lab-master-0      |   up   |  2620:52:0:1302::20  |                                                      | lab  |  kvirt  |
   |      lab-master-1      |   up   |  2620:52:0:1302::21  |                                                      | lab  |  kvirt  |
   |      lab-master-2      |   up   |  2620:52:0:1302::22  |                                                      | lab  |  kvirt  |
   | lab-mgmt-spoke1-node-0 |   up   |  2620:52:0:1302::23  |                                                      | lab  |  kvirt  |
   +------------------------+--------+----------------------+------------------------------------------------------+------+---------+

On top of the 3 masters and spoke commented above, the lab will also make of an
additional VM, called lab-installer, from which we'll access the kubeconfig
files and kubernetes/openshift files.

Once deployed, you can access such machine by using:

.. code-block:: console

   $ kcli lab-installer


Parameters file
~~~~~~~~~~~~~~~

Kcli needs a parameters file in order to run the installation. To do so, create
a file with the following parameters, name it as `kcli_parameters.yml` and put
it under the `kcli-openshift4-baremetal` directory.

.. code-block:: yaml

   lab: true
   provisioning_enable: false
   pool: default
   disconnected: true
   virtual_masters: true
   virtual_workers_number: 0
   launch_steps: true
   deploy_openshift: true
   version: stable
   tag: 4.9
   cluster: lab
   domain: karmalabs.com
   baremetal_cidr: 2620:52:0:1302::/64
   baremetal_net: lab-baremetal
   virtual_masters_memory: 16384
   virtual_masters_numcpus: 8
   api_ip: 2620:52:0:1302::2
   ingress_ip: 2620:52:0:1302::3
   baremetal_ips:
   - 2620:52:0:1302::20
   - 2620:52:0:1302::21
   - 2620:52:0:1302::22
   - 2620:52:0:1302::23
   baremetal_macs:
   - aa:aa:aa:aa:bb:01
   - aa:aa:aa:aa:bb:02
   - aa:aa:aa:aa:bb:03
   disconnected_operators:
   - advanced-cluster-management
   ztp_spokes:
   - name: mgmt-spoke1
     masters_number: 1
     workers_number: 0
     virtual_nodes_number: 1
   vmrules:
   - lab-installer:
        disks: [200]
   lab_extra_dns:
   - assisted-service-open-cluster-management
   - assisted-service-assisted-installer
   - assisted-image-service-open-cluster-management


Create cluster
~~~~~~~~~~~~~~

From the `kcli-openshift4-baremetal` directory, run:

.. code-block:: console

   $ kcli create plan -f plans/kcli_plan_ztp.yml --paramfile kcli_parameters.yml lab


Debug installation process
~~~~~~~~~~~~~~~~~~~~~~~~~~

Once you run the create plan command, just follow the installation process and
debug issues by following the output of cloud-init in the installer VM. Take
not that this may take a while to complete.

.. code-block:: console

   $ kcli ssh lab-installer
   $ tail -f /var/log/cloud-init-output.log
