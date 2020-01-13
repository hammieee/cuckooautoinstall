About this repo forked from buguroo
=======================

I have forked this and added some changes to help install cuckoo.
Since the master repo is not maintained, I have added a few more steps to make the painful
cuckoo installation quicker, easier and painless

OS: `ubuntu-18.04.3-desktop-amd64 <http://releases.ubuntu.com/18.04/>`

RAM: 8 GB

Processor: 4 Cores

About CuckooAutoinstall
=======================

`Cuckoo Sandbox <http://www.cuckoosandbox.org/>`_. auto install script

What is Cuckoo Sandbox?
-----------------------

Cuckoo Sandbox is a malware analysis system.

What does that mean? 
--------------------

It means that you can throw any suspicious file at it and get a report with
details about the file's behavior inside an isolated environment.

We created this at `Buguroo Offensive Security <http://www.buguroo.com>`_ initially to make the painful
cuckoo installation quicker, easier and painless

Supported systems
-----------------

Most of this script is not distro dependant (tough of course you've got to run
it on GNU/Linux), but package installation, at this moment supports only
debian derivatives.

Also, given that we use the propietary virtualbox version (most of the time OSE
edition doesn't fulfill our needs), this script requires that they've got
a debian repo in `Virtualbox Downloads <http://downloads.virtualbox.org>`_ 
for your distro. Forcing the distro in config file should make it work in
unsupported ones.

Authors
-------

`David Reguera García - Dreg <http://github.com/David-Reguera-Garcia-Dreg>`_ - `dreguera@buguroo.com <mailto:dreguera@buguroo.com>`_ - `@fr33project <https://twitter.com/fr33project>`_ 

`David Francos Cuartero - XayOn <http://github.com/Xayon>`_ - `dfrancos@buguroo.com <mailto:dfrancos@buguroo.com>`_ - `@davidfrancos <https://twitter.com/davidfrancos>`_


Quickstart guide
================

* Clone this repo & execute the script: *bash cuckooautoinstall.bash -v*

It will help to install most of the requirements and dependencies needed to install cuckoo

.. image:: /../screenshots/cuckooautoinstall.png?raw=true

It accepts parameters

::

    ┌─────────────────────────────────────────────────────────┐
    │                CuckooAutoInstall 0.2                    │
    │ David Reguera García - Dreg <dreguera@buguroo.com>      │
    │ David Francos Cuartero - XayOn <dfrancos@buguroo.com>   │
    │            Buguroo Offensive Security - 2015            │
    └─────────────────────────────────────────────────────────┘
    Usage: cuckooautoinstall.bash [--verbose|-v] [--help|-h] [--upgrade|-u]

        --verbose   Print output to stdout instead of temp logfile
        --help      This help menu
        --upgrade   Use newer volatility, yara and jansson versions (install from source)

For most setups, --upgrade is recommended always.

* Install other requirements

::

    sudo apt-get install libtiff5-dev libjpeg8-dev zlib1g-dev libfreetype6-dev liblcms2-dev libwebp-dev tcl8.6-dev tk8.6-dev python-tk

* Add a password (as root) for the user *'cuckoo'* created by the script

::

    passwd cuckoo

* Create a virtual environment

::

    virtualenv venv

* Activate the virtual environment

::

    . venv/bin/activate

* Install distorm3 and volatility

::

    pip install distorm3
    pip install -U git+https://github.com/volatilityfoundation/volatility.git

* Install tcpdump and allow non-root user to use tcpdump

::

    sudo apt-get install tcpdump apparmor-utils
    sudo aa-disable /usr/sbin/tcpdump
    sudo apt-get install tcpdump
    sudo groupadd pcap
    sudo usermod -a -G pcap cuckoo
    sudo chgrp pcap /usr/sbin/tcpdump
    sudo setcap cap_net_raw,cap_net_admin=eip /usr/sbin/tcpdump

* Create the virtual machines `http://docs.cuckoosandbox.org/en/latest/installation/guest/`
  or import virtual machines

::

  VBoxManage import virtual_machine.ova

* Add to the virtual machines with HostOnly option using vboxnet0

::

  vboxmanage modifyvm “virtual_machine" --hostonlyadapter1 vboxnet0

* Install cuckoo 

::

  pip install -U cuckoo
  
* Initiate cuckoo for the first time 

::

  cuckoo -d
  cuckoo community
  
* Configure cuckoo (`http://docs.cuckoosandbox.org/en/latest/installation/host/configuration/` )

Enable memory_dump (memory_dump = yes)
::
  gedit .cuckoo/conf/cuckoo.conf

Enable memory dump ([memory] enabled = yes)
::
  gedit .cuckoo/conf/processing.conf
  
Change guest profile (`https://github.com/volatilityfoundation/volatility/wiki/2.6-Win-Profiles`)
::
  gedit .cuckoo/conf/memory.conf
  
::
  guest_profile = Win7SP1x64

Enable mongodb for Web Interface and Generate HTML report
::

  gedit .cuckoo/conf/reporting.conf
  
::

    [mongodb]
    enabled = yes
    
    [singlefile]
    # Enable creation of report.html and/or report.pdf?
    enabled = yes
    # Enable creation of report.html?
    html = yes
  
* Execute cuckoo 

::
    cuckoo -d
    
* Run cuckoo web interface

:: 
    cuckoo web -H <IP address>

Script features
=================

* Installs by default Cuckoo sandbox with the ALL optional stuff: yara, ssdeep, django ...
* Installs the last versions of ssdeep, yara, pydeep-master & jansson.
* Solves common problems during the installation: ldconfigs, autoreconfs...
* Installs by default virtualbox and *creates the hostonlyif*.
* Creates the *'cuckoo'* user in the system and it is also added this user to *vboxusers* group.
* Enables *mongodb* in *conf/reporting.conf* 
* Creates the *iptables rules* and the ip forward to enable internet in the cuckoo virtual machines

::

    sudo iptables -A FORWARD -o eth0 -i vboxnet0 -s 192.168.56.0/24 -m conntrack --ctstate NEW -j ACCEPT
    sudo iptables -A FORWARD -m conntrack --ctstate ESTABLISHED,RELATED -j ACCEPT
    sudo iptables -A POSTROUTING -t nat -j MASQUERADE
    sudo sysctl -w net.ipv4.ip_forward=1

Enables run *tcpdump* from nonroot user

::

    sudo apt-get -y install libcap2-bin
    sudo setcap cap_net_raw,cap_net_admin=eip /usr/sbin/tcpdump

Fixes the *"TEMPLATE_DIRS setting must be a tuple"* error when running python manage.py from the *DJANGO version >= 1.6*. Replacing in *web/web/settings.py*

::

        TEMPLATE_DIRS = (
            "templates"
        )


becomes

::

        TEMPLATE_DIRS = (
            ("templates"),
        )


Install cuckoo as daemon
==========================

For this, we recommend supervisor usage.

Install supervisor

::

    sudo apt-get install supervisor

Edit */etc/supervisor/conf.d/cuckoo.conf* , like

::

        [program:cuckoo]
        command=python cuckoo.py
        directory=/home/cuckoo
        User=cuckoo

        [program:cuckoo-api]
        command=python api.py
        directory=/home/cuckoo/utils
        user=cuckoo

Reload supervisor

::

  sudo supervisorctl reload


iptables
========

As you probably have already noticed, iptables rules don't stay there after
a reboot. If you want to make them persistent, we recommend 
iptables-save & iptables-restore

::

    iptables-save > your_custom_iptables_rules
    iptables-restore < your_custom_iptables_rules



Extra help
==========

You may want to read:

* `Remote <./doc/Remote.rst>`_ - Enabling remote administration of VMS and VBox
* `OVA <./doc/OVA.rst>`_ - Working with OVA images
* `Antivm <./doc/Antivm.rst>`_ How to deal with malware that has VM detection techniques
* `VMcloak <./doc/Vmcloak.rst>`_ VMCloak - Cuckoo windows virtual machines management

TODO
====

* Improve documentation

Contributing
============

This project is licensed as GPL3+ as you can see in "LICENSE" file.
All pull requests are welcome, having in mind that:

- The scripting style must be compliant with the current one
- New features must be in sepparate branches (way better if it's git-flow =) )
- Please, check that it works correctly before submitting a PR.

We'd probably be answering to PRs in a 7-14 day period, please be patient.
