Importing OVA
=======================
* Create a new VM outside the ubuntu host, setup properly and export it as .OVA file. 
* After that, import the OVA file as a new virtual machine in the Ubuntu host.

Configuring the cuckoo guest
----------------------------

* Install python 2.7 in windows 7 as it is the requirements

  `https://www.python.org/downloads/release/python-278/`

* Install .net framework 4.5

  `https://www.microsoft.com/en-us/download/details.aspx?id=30653`

* Install wps office
  
  `https://www.wps.com/`

* Install python Pillow

  `https://pillow.readthedocs.io/en/latest/installation.html`

* Copy “agent.py” from ubuntu host to the cuckoo guest

::

	Directory: /home/{user}/.cuckoo/agent/agent.py
  
* Disable firewalls and auto updates

* Shut down the Virtual machine (Win7) and export as OVA format

When importing in Ubuntu
------------------------------

1.	Rename the cuckoo guest to “cuckoo1”
2.	Run the cuckoo guest inside ubuntu host
3.	Run agent.py (if you want to hide the terminal, rename it to .pyw)
4.	Take a snapshot
