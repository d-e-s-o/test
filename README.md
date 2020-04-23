esxvm
=====

Purpose
-------

**esxvm** is a program aiding in managing virtual machines on an ESX
host. It is designed with scriptability, extensibility and
customizability as the main design goals. These goals are achieved by a
plugin-based infrastructure: the main program merely provides the means
to find and run plugins. The actual interaction with a host is performed
by plugins.

```diff
--- cat.py
+++ cat.py
@@ -13,7 +13,7 @@ def localFileCompleter(parser, values, word):

 parser = CompletingArgumentParser(prog="cat")
 parser.add_argument(
-  "files", nargs="+",
+  "files", nargs="+", completer=localFileCompleter,
   help="Files to cat."
 )

```

<span style=“color:green;”>blubb</span>

<style
  type="text/css">
h1 {color:red;}

p {color:blue;}
</style>
<p>okay</p>

Installation
------------

The **esxvm** package is self-contained in that it requires nothing more
than a Python 3 installation. All dependencies are shipped with it.

To install the program the packages have to be made known to Python by
either installing them globally or adjusting the ``PYTHONPATH``
environment variable appropriately. Most of the time a bash alias (other
shells have similar means) fits this purpose perfectly:

``alias vm='PYTHONPATH="<path>/esxvm/pyvmomi:<path>/esxvm/requests:<path>/esxvm/pyyaml/lib3:<path>/esxvm/esxvm/src" python3 "<path>/esxvm/esxvm/src/deso/vm/vm.py"'``


Configuration
-------------

**esxvm** is generally stateless -- all the information it requires are
taken from command line arguments. However, it supports reading from a
configuration file to allow for easier accessibility (by reducing the
amount of typing). This ``YAML`` file can be used to supply certain
arguments to a set of plugins (or the main program). The available
arguments depend on the individual plugins and can be inquired using the
-h/--help option.

The file resides in ``${XDG_CONFIG_HOME}/esxvm/esxvm`` (or
``~/.config/esxvm/esxvm`` if ``XDG_CONFIG_HOME`` is not defined). An
example looks as follows:

```yaml
- 'esxvm': [
    '--debug',
    '--verbose',
    '--plugin-dir=<path>/esxvm/esxvm/src/plugins',
    '--plugin-dir=<path>/esxvm-plugins/esxvm-plugins/src/plugins',
  ]
- 'make': [
    # 20 GiB disk.
    '--disk=20480',
  ]
- 'on': ['--force']
- 'off': ['--force']
- 'reset': ['--force']
- 'suspend': ['--force']
- 'status': ['--multi-match']
- '*': [
    '--host=<esx-ip>',
    '--username=root',
    '--password=<password>',
  ]
```

A couple of things are of interest here. ``esxvm`` is a special key that
refers to the main program. In this example we want the program to be
more verbose and we also supply two directories to search for plugins.

Following this special value are configurations for a couple of plugins
referenced by the commands to invoke them (the order is of no
importance). For example, ``on`` is the command to invoke the poweron.py
core plugin which can be used to power on a virtual machine.

Last but not least, the configuration file supports matching based on
(shell-style) patterns, that is, the asterisk entry matches *all*
plugins (but not the main program).


Examples
--------

(The following exmaples assume **esxvm** to be invokable as ``vm`` as is
the case in the configuration mentioned beforehand)

- Create a virtual machine named ``test`` with two virtual CPUs and power
  it on:<br />
  ``$ vm make --cpus=2 test && vm on test``

  or (safe to use in scripts because 'test' might not uniquely identify
  a VM):<br />
  ``$ vm on $(vm make --cpus=2 --print-id test)``

- List all registered virtual machines and their IDs:<br />
  ``$ vm ls``

- Find and register all virtual machines containing 'foo' in their name
  (based on their .vmx file names):<br />
  ``$ vm reg -m '*foo*'``

- Power on all VMs registered on the host (the virtual machines need to
  be either powered off or suspended for this operation to succeed; use
  the -f/--force switch to relieve this restriction):<br />
  ``$ vm on '*' -m``

- Forcefully restart all VMs starting with 'gentoo' (and powering on
  powered off or suspended ones):<br />
  ``$ vm reset 'gentoo*' --multi-match --force``

- Inquire the power status of all virtual machines:<br />
  ``$ vm status -m '*'``

- Destroy the VM with ID 42 (must not be powered on):<br />
  ``$ vm kill 42``


Support
-------

The module is tested with Python 3. There is no work going on to
ensure compatibility with Python 2.
