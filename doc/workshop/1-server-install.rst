..
  Copyright 2015-2018  Red Hat, Inc.

  This work is licensed under the Creative Commons Attribution 4.0
  International License. To view a copy of this license, visit
  http://creativecommons.org/licenses/by/4.0/.


Unit 1: Installing the FreeIPA server
=======================================

In this unit you will install a FreeIPA server.  All tasks in
subsequent units require the services and data provided by the
server. Read the FreeIPA deployment recommendations, see
https://www.freeipa.org/page/Deployment_Recommendations

First, in the directory containing the ``Vagrantfile`` (the clone of
this repository), execute ``vagrant up`` to bring up the Vagrant
environment.

::

  $ vagrant up
If you get an error with the storage pools when you run vagrant up,
then see if you already have pools that use /var/libvirt/images. If
you do, then remove them. You'll need to run the following if you
do not see any storage pools using virsh from your account:

::
   
  $ virsh pool-list --all
If there are any pools, then remove them:
::
  $ virsh pool-destroy <pool name>
  $ virsh pool-undefine <pool name>

If you still have the problem with pools, run the commands with sudo:
::
  $ sudo virsh pool-list --all
  
If there are any pools, then remove them:
::
  $ sudo virsh pool-destroy <pool name>
  $ sudo virsh pool-undefine <pool name>

If you had to remove existing pools, then now you can try to bring the
cluster up again:
::
   $ vagrant up

The Vagrant environment contains three hosts:

- ``server.ipademo.local``
- ``replica.ipademo.local``
- ``client.ipademo.local``

From the directory containing the ``Vagrantfile``, SSH into the
``server`` machine::

  $ vagrant ssh server


On ``server``, start the FreeIPA server installation program::

  [server]$ sudo ipa-server-install --no-host-dns --mkhomedir

The ``--no-host-dns`` argument is needed because there are no reverse
DNS records for the Vagrant environment.  For production deployment,
this important sanity check should not be skipped. The ``--mkhomedir``
flag configures PAM to create missing home directories when users log
into the host for the first time. FreeIPA supports automount so
consider using that for production deployments.

You will be asked a series of questions. Accept the defaults for most
of the questions, except as outlined below.

Configure FreeIPA's DNS server::

  Do you want to configure integrated DNS (BIND)? [no]: yes


Accept default values for the server hostname, domain name and realm::

  Enter the fully qualified domain name of the computer
  on which you're setting up server software. Using the form
  <hostname>.<domainname>
  Example: master.example.com.


  Server host name [server.ipademo.local]:

  Warning: skipping DNS resolution of host server.ipademo.local
  The domain name has been determined based on the host name.

  Please confirm the domain name [ipademo.local]:

  The kerberos protocol requires a Realm name to be defined.
  This is typically the domain name converted to uppercase.

  Please provide a realm name [IPADEMO.LOCAL]:


Enter passwords for *Directory Manager* (used to manage the
directory server) and *admin* (the main account used for FreeIPA
administration).  Use something simple that you're not going to
forget during the workshop!

::

  Certain directory server operations require an administrative user.
  This user is referred to as the Directory Manager and has full access
  to the Directory for system management tasks and will be added to the
  instance of directory server created for IPA.
  The password must be at least 8 characters long.

  Directory Manager password:
  Password (confirm):

  The IPA server requires an administrative user, named 'admin'.
  This user is a regular system account used for IPA server administration.

  IPA admin password:
  Password (confirm):


Do not configure a DNS forwarder (you will want to configure a DNS
forwarder for a real-world deployment but it is not needed for this
workshop) and accept the defaults for configuring the reverse zone::

  Checking DNS domain ipademo.local., please wait ...
  Do you want to configure DNS forwarders? [yes]: no
  No DNS forwarders configured
  Do you want to search for missing reverse zones? [yes]:
  Do you want to create reverse zone for IP 192.168.33.10 [yes]:
  Please specify the reverse zone name [33.168.192.in-addr.arpa.]:
  Using reverse zone(s) 33.168.192.in-addr.arpa.

Note: even though I typed 'yes' for searching for missing reverse zones,
it did not search. I added it manually after the server was configured.

Next, you will be presented with a summary of the server
configuration and asked for final confirmation.  Give confirmation to begin the
server installation::

  The IPA Master Server will be configured with:
  Hostname:       server.ipademo.local
  IP address(es): 192.168.33.10
  Domain name:    ipademo.local
  Realm name:     IPADEMO.LOCAL

  The CA will be configured with:
  Subject DN:   CN=Certificate Authority,O=IPADEMO.LOCAL
  Subject base: O=IPADEMO.LOCAL
  Chaining:     self-signed

  BIND DNS server will be configured to serve IPA domain with:
  Forwarders:       No forwarders
  Forward policy:   only
  Reverse zone(s):  33.168.192.in-addr.arpa.

  Continue to configure the system with these values? [no]: yes

The installation takes a few minutes; you will see output indicating
the progress. There are 28 steps that must complete.

When it completes, run ``kinit admin`` and enter your *admin*
password to obtain a Kerberos *ticket granting ticket* (TGT) for the
``admin`` user::

  [server]$ kinit admin
  Password for admin@IPADEMO.LOCAL:  <enter password>

Run ``klist`` to view your current Kerberos tickets::

  [server]$ klist
  Ticket cache: KEYRING:persistent:1000:1000
  Default principal: admin@IPADEMO.LOCAL

  Valid starting     Expires            Service principal
  10/15/15 01:48:59  10/16/15 01:48:57  krbtgt/IPADEMO.LOCAL@IPADEMO.LOCAL

Fixup: I added the reverse zone manually.
::
  [server]$ kinit admin
  Password for admin@IPADEMO.LOCAL:  <enter password>
Note: If you krbtgt is still valid, you will not need to kinit.::

  $ ipa dnszone-add --name-from-ip='192.168.33.10'
  [server]$ ipa dnszone-add --name-from-ip='192.168.33.10'
    Zone name [33.168.192.in-addr.arpa.]: 
    Zone name: 33.168.192.in-addr.arpa.
    Active zone: TRUE
    Authoritative nameserver: server.ipademo.local.
    Administrator e-mail address: hostmaster
    SOA serial: 1612868315
    SOA refresh: 3600
    SOA retry: 900
    SOA expire: 1209600
    SOA minimum: 3600
    BIND update policy: grant IPADEMO.LOCAL krb5-subdomain 33.168.192.in-addr.arpa. PTR;
    Dynamic update: FALSE
    Allow query: any;
    Allow transfer: none;
  [server]$ ipa dnsconfig-show
  ---------------------------------
  Global DNS configuration is empty
  ---------------------------------
    IPA DNS servers: server.ipademo.local
  [server]$ ipa dnszone-find
    Zone name: 33.168.192.in-addr.arpa.
    Active zone: TRUE
    Authoritative nameserver: server.ipademo.local.
    Administrator e-mail address: hostmaster
    SOA serial: 1612868316
    SOA refresh: 3600
    SOA retry: 900
    SOA expire: 1209600
    SOA minimum: 3600
    Allow query: any;
    Allow transfer: none;
    
    Zone name: ipademo.local.
    Active zone: TRUE
    Authoritative nameserver: server.ipademo.local.
    Administrator e-mail address: hostmaster.ipademo.local.
    SOA serial: 1612822823
    SOA refresh: 3600
    SOA retry: 900
    SOA expire: 1209600
    SOA minimum: 3600
    Allow query: any;
    Allow transfer: none;
  ----------------------------
  Number of entries returned 2
  ----------------------------

The FreeIPA server is now set up and you are ready to begin
enrolling client machines, creating users, managing services, and
more!

To prepare for the next unit, exit the ``server`` SSH session (but
do not shut the VM down).  The next essential unit is
`Unit 2: Enrolling client machines <2-client-install.rst>`_.

Alternatively, if you would like to immediately install a replica
server (essential for production deployments), you can take a detour
to `Unit 7: Replica installation <7-replica-install.rst>`_.
