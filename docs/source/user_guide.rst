User Guide
===========

In order to access the Jupyterhub instance now, an user would need to connect via ssh to the BETIF machine and bind a port through the ``-D``  flag:

.. code-block:: console

   $ ssh -ND 9999 <BETIF_machine>

and use the BETIF server as SOCKS proxy (the instructions on how to do this depend on the browser used):

* **Firefox**: *Settings* -> *Network settings* -> select *Manual proxy configuration* and type ``localhost`` on **Host SOCKS** with ``9999`` port;
* **Safari** and **Chrome** (on MacOS): Go to *SystemPreferences* -> *Network* -> *Wi-Fi* -> Click on *Details* next to the connected Wi-Fi -> *proxy* -> enable **SOCKS proxy** -> put the same information written above.
