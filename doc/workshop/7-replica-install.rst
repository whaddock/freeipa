Unit 7: Replica installation
==============================

**Prerequisites**:

- `Unit 1: Installing the FreeIPA server <1-server-install.rst>`_

FreeIPA is designed to be run in a replicated multi-master
environment.  In this unit, we will install a replica of the
existing master.  For recommended production topologies, see
https://www.freeipa.org/page/Deployment_Recommendations#Servers.2FReplicas.

Read through the Red Hat configuration of the replica server,
https://access.redhat.com/documentation/en-us/red_hat_enterprise_linux/7/html/linux_domain_identity_authentication_and_policy_guide/creating-the-replica

If you have disabled the ``allow_all`` HBAC rule, add a new rule
that will **allow ``admin`` to access the ``sshd`` service on any
host**.

Client installation
-------------------

The first step of replica creation is to enrol the machine that will
become the replica.  SSH to the ``replica`` VM and enrol it per
`Unit 2: Enrolling client machines <2-client-install.rst>`_

Replica promotion
-----------------

Now promote the client to server.  We will set up the replica
*without* the CA and DNS role.  In a production deployment there
should be at least one instance of these services in each data
center.  These roles may also be configured later via
``ipa-ca-install(1)`` and ``ipa-dns-install(1)``.

::

  [replica]$ sudo ipa-replica-install 
  Password for admin@IPADEMO.LOCAL:
  ipaserver.install.server.replicainstall: ERROR    Reverse DNS resolution of address 192.168.33.10 (server.ipademo.local) failed. Clients may not function properly. Please check your DNS setup. (Note that this check queries IPA DNS directly and ignores /etc/hosts.)
  Continue? [no]: yes
  Run connection check to master
  Connection check OK
  Configuring directory server (dirsrv). Estimated time: 30 seconds
    [1/41]: creating directory server instance
    [2/41]: enabling ldapi
  ...

The rest of the replica installation process is almost identical to
server installation.  One important difference is the initial
replication of data to the new Directory Server instance::

  [28/41]: setting up initial replication
  Starting replication, please wait until this has completed.
  Update in progress, 4 seconds elapsed
  Update succeeded

After ``ipa-replica-install`` finishes, the replica is operational.
LDAP changes on any server will be replicated to all other servers.

CA Installation
-----------------
Now, install the CA on the replica.
::
  sudo ipa-ca-install

  [replica]$ sudo ipa-ca-install
  Directory Manager (existing master) password: 

  Run connection check to master
  Connection check OK
  Configuring certificate server (pki-tomcatd). Estimated time: 3 minutes
    [1/24]: creating certificate server db
    [2/24]: setting up initial replication
  Starting replication, please wait until this has completed.
  Update in progress, 4 seconds elapsed
  Update succeeded

    [3/24]: creating installation admin user
    [4/24]: configuring certificate server instance
    [5/24]: exporting Dogtag certificate store pin

    ...
    [23/24]: enabling CA instance
    [24/24]: configuring certmonger renewal for lightweight CAs
  Done configuring certificate server (pki-tomcatd).
  Updating DNS system records
  [replica]$ 

    [23/24]: enabling CA instance
    [24/24]: configuring certmonger renewal for lightweight CAs
  Done configuring certificate server (pki-tomcatd).
  Updating DNS system records
  [replica]$ 


DNS Installation
-----------------
Now, install the CA on the replica.
::
  
  [replica]$ sudo ipa-dns-install --no-forwarders 

  The log file for this installation can be found in /var/log/ipaserver-install.log
  ==============================================================================
  This program will setup DNS for the FreeIPA Server.
  ...

    [6/7]: creating replica keys
    [7/7]: configuring ipa-dnskeysyncd to start on boot
  Done configuring DNS key synchronization service (ipa-dnskeysyncd).
  Restarting ipa-dnskeysyncd
  Restarting named
  Updating DNS system records
  ==============================================================================
  Setup complete

  Global DNS configuration in LDAP server is empty
  You can use 'dnsconfig-mod' command to set global DNS options that
  would override settings in local named.conf files


	You must make sure these network ports are open:
		TCP Ports:
		  * 53: bind
		UDP Ports:
		  * 53: bind
  [replica]$ 

You can now proceed to
`Unit 8: Sudo rule management <8-sudorule.rst>`_
or
`return to the curriculum overview <workshop.rst#curriculum-overview>`_
to see all the available topics.
