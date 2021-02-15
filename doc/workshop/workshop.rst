..
  Copyright 2015, 2016  Red Hat, Inc.

  This work is licensed under the Creative Commons Attribution 4.0
  International License. To view a copy of this license, visit
  http://creativecommons.org/licenses/by/4.0/.


Introduction
============

FreeIPA_ is a centralised identity management system.  In this
workshop you will learn how to deploy FreeIPA servers and enrol
client machines, define and manage user and service identities, set
up access policies, configure network services to take advantage of
FreeIPA's authentication and authorisation facilities and issue
X.509 certificates for services.

.. _FreeIPA: http://www.freeipa.org/page/Main_Page


Curriculum overview
-------------------

Mandatory:

- `Unit 1: Installing the FreeIPA server <1-server-install.rst>`_
- `Unit 2: Enrolling client machines <2-client-install.rst>`_
- `Unit 3: User management and Kerberos authentication <3-user-management.rst>`_
- `Unit 4: Host-based access control (HBAC) <4-hbac.rst>`_

Optional units—choose the topics that are relevant to you:

- `Unit 5: Web application authentication and authorisation <5-web-app-authnz.rst>`_
- `Unit 6: Service certificates <6-cert-management.rst>`_
- `Unit 7: Replica installation <7-replica-install.rst>`_
- `Unit 8: Sudo rule management <8-sudorule.rst>`_
- `Unit 9: SELinux User Maps <9-selinux-user-map.rst>`_
- `Unit 10: SSH user and host key management <10-ssh-key-management.rst>`_
- `Unit 11: Kerberos ticket policy <11-kerberos-ticket-policy.rst>`_


Editing files on VMs
--------------------

Parts of the workshop involve editing files on virtual
machines.  The ``vi`` and GNU ``nano`` editors are available on the
VMs.  If you are not familiar with ``vi`` or you are unsure of what to use, you
should choose ``nano``.


Example commands
----------------

This guide contains many examples of commands.  Some of the commands
should be executed on your host, others on a particular guest VM.
For clarity, commands are annotated with the host on which they are
meant to be executed, as in these examples::

  $ echo "Run it on virtualisation host (no annotation)"

  [server]$ echo "Run it on FreeIPA server"

  [client]$ echo "Run it on IPA-enrolled client"

  ...


Preparation
===========

Some preparation is needed prior to the workshop.  The workshop is
designed to be carried out in a Vagrant_ environment that configures
three networked virtual machines (VMs) with all software needed for
the workshop.  **The goal of this preparation** is to ``vagrant up``
the VMs.  After this preparation is completed you are ready to begin
the workshop.

.. _Vagrant: https://www.vagrantup.com/


Requirements
------------

For the FreeIPA workshop you will need to:

- Run this workshop on Fedora with 64-bit hardware. We have only
  tested this on Fedora 32.

- Install **Vagrant** and **libvirt**.
  
- Use Git to clone the repository containing the ``Vagrantfile``

- Fetch the Vagrant *box* for the workshop

- Add entries for the guest VMs to your hosts file (so you can
  access them by their hostname)

Please set up these items **prior to the workshop**.  More detailed
instructions follow.


Install Vagrant
------------------------------

Fedora 32
^^^^^^

If you intend to use the ``libvirt`` provider (recommended), install
``vagrant-libvirt`` and ``vagrant-libvirt-doc``::

  $ sudo dnf install -y vagrant-libvirt vagrant-libvirt-doc

Also ensure you have the latest versions of ``selinux-policy`` and
``selinux-policy-targeted``. (Just run sudo dnf update).

Allow your regular user ID to start and stop Vagrant boxes using ``libvirt``.
Add your user to ``libvirt`` group so you don't need to enter your administrator
password everytime::

  $ sudo gpasswd -a ${USER} libvirt
  $ newgrp libvirt

Finally restart the services::

  $ sudo systemctl restart libvirtd
  $ sudo systemctl restart polkit

Note: The newgrp and two restart commands will need to be run whenever
you logout and log back in again to your terminal session. You can
see what happens if you run id before and after the newgrp command
is run.

Clone the freeipa repository
---------------------

This repository contains the ``Vagrantfile`` that is used for the
workshop, which you will need locally. You'll find it in the
doc/workshop folder.

::

  $ git clone https://github.com/freeipa/freeipa.git


Fetch Vagrant box
-----------------

Fetch the Vagrant box.  It is > 600MB. It may take a while depending
on your internet connection speed.

::

  $ vagrant box add netoarmando/freeipa-workshop


Edit the Vagrant file to increase memory
----------------------
Use your favorite editor to change the memory from 1024 to 2048.
::
  config.vm.provider :libvirt do |libvirt|
    libvirt.qemu_use_session = false
    libvirt.memory = 2048   <------------- Change from 1024
   
If you do not have enough RAM in your host computer, you can bring
up the virtual machines individually. The two IPA servers need to
have 2048 MB of memory but you do not need to run the replica
except during the lesson 7 on IPA Replica installation. Just bring
up the server first and let it stay up. then bring up the client
when you start the client lessons. You can stop the client and then
bring up the replica for lesson 7, then shut it down for the
remainder of the lessons. All lessions but lesson 7 just need
to have the server and client up. Lesson 1 only needs to have
the server up.
  
Add hosts file entries
----------------------

*This step is optional.  All units can be completed using the CLI
only.  But if you want to access the FreeIPA Web UI or other web
servers on the VMs from your browser, follow these instructions.*

Add the following entries to your hosts file::

  192.168.33.10   server.ipademo.local
  192.168.33.11   replica.ipademo.local
  192.168.33.20   client.ipademo.local

On Unix systems (including Mac OS X), the hosts file is ``/etc/hosts``
(you need elevated permissions to edit it.)


Next step
---------

You are ready to begin the workshop.  Continue to
`Unit 1: Installing the FreeIPA server <1-server-install.rst>`_.


After the workshop
------------------

Here are some contact details and resources that may help you after
the workshop is over:

- IRC: ``#freeipa`` and ``#sssd`` (Freenode)

- ``freeipa-users@lists.fedorahosted.org`` `mailing list
  <https://lists.fedoraproject.org/archives/list/freeipa-users@lists.fedorahosted.org/>`_

- `How To guides <https://www.freeipa.org/page/HowTos>`_: large
  index of articles about specialised tasks and integrations

- `Troubleshooting guide
  <https://www.freeipa.org/page/Troubleshooting>`_: how to debug
  common problems; how to report bugs

- `Bug tracker <https://pagure.io/freeipa>`_

- Information about the `FreeIPA public demo
  <https://www.freeipa.org/page/Demo>`_ instance

- `Deployment Recommendations
  <https://www.freeipa.org/page/Deployment_Recommendations>`_:
  things to consider when going into production

- `Documentation index
  <https://www.freeipa.org/page/Documentation>`_

- `FreeIPA Planet <http://planet.freeipa.org/>`_: aggregate of
  several FreeIPA and identity-management related blogs

- `GitHub organisation <https://github.com/freeipa>`_.  In addition
  to the `main repository <https://github.com/freeipa/freeipa>`_
  there are various tools, CI-related projects and documentation.

- `Development roadmap <https://www.freeipa.org/page/Roadmap>`_
