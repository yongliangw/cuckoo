===
FAQ
===

Here you can find answers for various Frequently Asked Questions:

.. contents::
   :local:


General Questions
=================

.. _analyze_urls:

Can I analyze URLs with Cuckoo?
-------------------------------

.. versionadded:: 0.5
    Native support for URL analysis was added to Cuckoo.

.. versionchanged:: 2.0-rc1
    Cuckoo will not only start the browser (i.e., Internet Explorer) but will
    also attempt to actively instrument it in order to extract interesting
    results such as executed Javascript, iframe URLs, etc. See also our
    `2.0-rc1 blogpost`_.

Additional details on URL submissions is documented at :doc:`../usage/submit`,
but it boils down to::

    $ cuckoo submit --url http://www.example.com

.. _`2.0-rc1 blogpost`: https://cuckoosandbox.org/2016-01-21-cuckoo-sandbox-20-rc1.html

.. _general_volatility:

Can I use Volatility with Cuckoo?
---------------------------------

.. versionadded:: 0.5
    Cuckoo introduces support for optional full memory dumps, which are
    created at the end of the analysis process. You can use these memory dumps to
    perform additional memory forensic analysis with `Volatility`_.

Please also consider that we don't particularly encourage this: since Cuckoo
employs some rootkit-like technologies to perform its operations, the results
of a forensic analysis would be polluted by the sandbox's components.

.. _`Volatility`: http://code.google.com/p/volatility/

.. _esxi_reqs:

What do I need to use Cuckoo with VMware ESXi?
----------------------------------------------

To run with VMware vSphere Hypervisor (or ESXi) Cuckoo leverages on libvirt or
pyVmomi (the Python SDK for the VMware vSphere API).
VMware API are used to take control over virtual machines, though these APIs are
available only in the licensed version. In VMware vSphere free
edition these APIs are read only, so you will be unable to use it with Cuckoo.
For the minimum license needed, please have a look at VMware website.

Troubleshooting
===============

.. _troubles_upgrade:

After upgrade Cuckoo stops to work
----------------------------------

Probably you upgraded it in a wrong way.
It's not a good practice to rewrite the files due to Cuckoo's complexity and
quick evolution.

Please follow the upgrade steps described in :doc:`../installation/upgrade`.

.. _troubles_problem:

Cuckoo stumbles and produces some error I don't understand
----------------------------------------------------------

Cuckoo is a mature but always evolving project, it's possible that
you encounter some problems while running it, but before you rush into
sending emails to everyone make sure you read what follows.

Cuckoo is not meant to be a point-and-click tool: it's designed to be a highly
customizable and configurable solution for somewhat experienced users and
malware analysts.

It requires you to have a decent understanding of your operating systems, Python,
the concepts behind virtualization and sandboxing.
We try to make it as easy to use as possible, but you have to keep in mind that
it's not a technology meant to be accessible to just anyone.

That being said, if a problem occurs you have to make sure that you did everything
you could before asking for time and effort from our developers and users.
We just can't help everyone, we have limited time and it has to be dedicated to
the development and fixing of actual bugs.

    * We have extensive documentation, read it carefully. You can't just skip parts
      of it.
    * We have a mailing list archive, search through it for previous threads where
      your same problem could have been already addressed and solved.
    * We have a `Community`_ platform for asking questions, use it.
    * We have lot of users producing content on Internet, `Google`_ it.
    * Spend some of your own time trying fixing the issues before asking ours, you
      might even get to learn and understand Cuckoo better.

Long story short: use the existing resources, put some efforts into it and don't
abuse people.

If you still can't figure out your problem, you can ask help on our online communities
(see :doc:`../finalremarks/index`).
Make sure when you ask for help to:

    * Use a clear and explicit title for your emails: "I have a problem", "Help me" or
      "Cuckoo error" are **NOT** good titles.
    * Explain **in details** what you're experiencing. Try to reproduce several
      times your issue and write down all steps to achieve that.
    * Use no-paste services and link your logs, configuration files and details on your
      setup.
    * Eventually provide a copy of the analysis that generated the problem.

.. _`Community`: https://community.cuckoosandbox.org
.. _`Google`: http://www.google.com

Check and restore current snapshot with KVM
-------------------------------------------

If something goes wrong with virtual machine it's best practice to check current snapshot
status.
You can do that with the following::

    $ virsh snapshot-current "<Name of VM>"

If you got a long XML as output your current snapshot is configured and you can skip
the rest of this chapter; anyway if you got an error like the following your current
snapshot is broken::

    $ virsh snapshot-current "<Name of VM>"
    error: domain '<Name of VM>' has no current snapshot

To fix and create a current snapshot first list all machine's snapshots::

    $ virsh snapshot-list "<Name of VM>"
     Name                 Creation Time             State
     ------------------------------------------------------------
     1339506531           2012-06-12 15:08:51 +0200 running

Choose one snapshot name and set it as current::

    $ snapshot-current "<Name of VM>" --snapshotname 1339506531
    Snapshot 1339506531 set as current

Now the virtual machine state is fixed.

Check and restore current snapshot with VirtualBox
--------------------------------------------------

If something goes wrong with virtual it's best practice to check the virtual machine
status and the current snapshot.
First of all check the virtual machine status with the following::

    $ VBoxManage showvminfo "<Name of VM>" | grep State
    State:           powered off (since 2012-06-27T22:03:57.000000000)

If the state is "powered off" you can go ahead with the next check, if the state is
"aborted" or something else you have to restore it to "powered off" before::

    $ VBoxManage controlvm "<Name of VM>" poweroff

With the following check the current snapshots state::

    $ VBoxManage snapshot "<Name of VM>" list --details
    Name: s1 (UUID: 90828a77-72f4-4a5e-b9d3-bb1fdd4cef5f)
    Name: s2 (UUID: 97838e37-9ca4-4194-a041-5e9a40d6c205) *

If you have a snapshot marked with a star "*" your snapshot is ready, anyway
you have to restore the current snapshot::

    $ VBoxManage snapshot "<Name of VM>" restorecurrent

Unable to bind result server error
----------------------------------

At Cuckoo startup if you get an error message like this one::

    2014-01-07 18:42:12,686 [root] CRITICAL: CuckooCriticalError: Unable to bind result server on 192.168.56.1:2042: [Errno 99] Cannot assign requested address

It means that Cuckoo is unable to start the result server on the IP address written
in cuckoo.conf (or in machinery.conf if you are using the resultserver_ip option inside).
This usually happen when you start Cuckoo without bringing up the virtual interface associated
with the result server IP address.
You can bring it up manually, it depends from one virtualization software to another, but
if you don't know how to do, a good trick is to manually start and stop an analysis virtual
machine, this will bring virtual networking up.

In the case of VirtualBox the hostonly interface `vboxnet0` can be created as follows::

    # If the hostonly interface vboxnet0 does not exist already.
    $ VBoxManage hostonlyif create

    # Configure vboxnet0.
    $ VBoxManage hostonlyif ipconfig vboxnet0 --ip 192.168.56.1 --netmask 255.255.255.0

Error during template rendering
-------------------------------

.. versionchanged:: 2.0-rc1

In our 2.0-rc1 release a bug was introduced that looks as follows in the
screenshot below. In order to resolve this issue in your local setup, please
open the ``web/analysis/urls.py`` file and modify the 21st line by adding an
underscore as follows::

     -        "/(?P<ip>[\d\.]+)?/(?P<host>[a-zA-Z0-9-\.]+)?"
     +        "/(?P<ip>[\d\.]+)?/(?P<host>[ a-zA-Z0-9-_\.]+)?"

The official fixes for this issue can be found in the `following`_ `commits`_.

.. _`following`: https://github.com/cuckoosandbox/cuckoo/commit/9c704f50e70227ed21ae1b79ba90540c3087fc57
.. _`commits`: https://github.com/cuckoosandbox/cuckoo/commit/558ded1787bc3377c404ac14a0b3fdce37b49bf4

.. image:: ../_images/screenshots/error_template_rendering.png

501 Unsupported Method ('GET')
------------------------------

.. versionchanged:: 2.0-rc1

Since 2.0-rc1 Cuckoo supports both the `legacy Cuckoo Agent`_ as well as a
`new, REST API-based, Cuckoo Agent`_ for communication between the Guest and
the Host machine. The new ``Cuckoo Agent`` is an improved Agent in the sense
that it also allows usage outside of Cuckoo. As an example, it is used
extensively by `VMCloak`_ in order to automatically create, configure, and
cloak Virtual Machines.

Now in order to determine whether the Cuckoo Host is talking to the legacy or
new ``Cuckoo Agent`` it does a ``HTTP GET`` request to the root path (``/``).
The legacy Cuckoo Agent, which is based on ``xmlrpc``, doesn't handle that
specific route and therefore returns an error, ``501 Unsupported method``.

Having said that, the message is not actually an error, it is simply Cuckoo
trying to determine to which version of the ``Cuckoo Agent`` it is talking.

.. note::
    It should be noted that even though there is a new ``Cuckoo Agent``
    available, backwards compatibility for the legacy ``Cuckoo Agent`` is
    still available and working properly.

.. image:: ../_images/screenshots/unsupported_method.png

.. _`legacy Cuckoo Agent`: https://github.com/cuckoosandbox/cuckoo/blob/master/agent/agent.py
.. _`new, REST API-based, Cuckoo Agent`: https://github.com/jbremer/agent/blob/master/agent.py
.. _`VMCloak`: https://github.com/jbremer/vmcloak

.. _tcpdump_permission_denied:

Permission denied for tcpdump
-----------------------------

.. versionchanged:: 2.0.0

With the new Cuckoo structure in-place all storage is now, by default, located
in ``~/.cuckoo``, including the PCAP file, which will be stored at
``~/.cuckoo/storage/analysis/task_id/dump.pcap``. On Ubuntu with AppArmor
enabled (default configuration) ``tcpdump`` doesn't have write permission to
dot-directories in ``$HOME``, causing the permission denied message and
preventing Cuckoo from capturing PCAP files.

One of the workaround is as follows - by installing ``AppArmor utilities`` and
simply disabling the ``tcpdump`` AppArmor profile altogether (more appropriate
solutions are welcome of course)::

    sudo apt-get install apparmor-utils
    sudo aa-disable /usr/sbin/tcpdump

.. _pip_install_issue:

DistributionNotFound / No distribution matching the version..
-------------------------------------------------------------

.. versionchanged:: 2.0.0

Installing Cuckoo through the Python package brings its own set of problems,
namely that of outdated Python package management software. This FAQ entry
targets the following issue..::

    $ cuckoo
    Traceback (most recent call last):
    File "/usr/local/bin/cuckoo", line 5, in <module>
        from pkg_resources import load_entry_point
    File "/usr/lib/python2.7/dist-packages/pkg_resources.py", line 2749, in <module>
        working_set = WorkingSet._build_master()
    File "/usr/lib/python2.7/dist-packages/pkg_resources.py", line 446, in _build_master
        return cls._build_from_requirements(__requires__)
    File "/usr/lib/python2.7/dist-packages/pkg_resources.py", line 459, in _build_from_requirements
        dists = ws.resolve(reqs, Environment())
    File "/usr/lib/python2.7/dist-packages/pkg_resources.py", line 628, in resolve
        raise DistributionNotFound(req)
    pkg_resources.DistributionNotFound: tlslite-ng==0.6.0a3

.. as well as the following..::

    $ pip install cuckoo
    [ ... ]
    Could not find a version that satisfies the requirement tlslite-ng==0.6.0a3 (from HTTPReplay==0.1.15->Cuckoo==2.0) (from versions: 0.6.0-alpha5, 0.5.0-beta5, 0.5.0, 0.6.0-alpha4, 0.5.2, 0.5.1, 0.5.0-beta1, 0.5.0-beta2, 0.5.0-beta4, 0.5.0-beta3, 0.6.0-alpha2, 0.5.0-beta6, 0.6.0-alpha1, 0.6.0-alpha3)
    Cleaning up...
    No distributions matching the version for tlslite-ng==0.6.0a3 (from HTTPReplay==0.1.15->Cuckoo==2.0)
    Storing debug log for failure in /home/cuckoo/.pip/pip.log

Those issues - and related ones - are caused by outdated Python package
management software. Fortunately their fix is fairly trivial and therefore
the following command should do the trick::

    pip install -U pip setuptools

.. _openfiles24:

IOError: [Errno 24] Too many open files
---------------------------------------

It is most certainly possible running into this issue when analyzing samples
that have a lot of dropped files, so many that the :ref:`cuckoo_process` can't
allocate any new file descriptors anymore.

The easiest workaround for this issue is to bump the soft and hard file
descriptor limit for the current user. This may be done as documented in the
`following blogpost <https://easyengine.io/tutorials/linux/increase-open-files-limit/>`_.

Remember that you have to login to a new shell (i.e., usually check out first)
session in order for the changes to take effect.