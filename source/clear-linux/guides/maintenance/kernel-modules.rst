.. _kernel-modules:

Add kernel modules 
##################

Kernel modules are additional pieces of software capable of being inserted 
into the Linux kernel to add functionality, such as a hardware driver. 
Kernel modules may already be part of the Linux source tree (in-tree) or may 
come from an external source, such as directly from a vendor (out-of-tree).  

In cases where drivers beyond those enabled by default in |CL-ATTR| are
needed it may be necessary to:

.. contents:: :local:
   :depth: 1
   :backlinks: top

Check if the module is available through |CL|
=============================================

Using an existing module is significantly easier to maintain and retains 
signature verification of the |CL| kernel. For more information on |CL| 
security practices, see the :ref:`security` page.

|CL| comes with many upstream kernel modules available for use.  If 
you require a kernel module, be sure to check whether it is already available in |CL| first. 

You can search for kernel module file names, which end with the :file:`.ko` 
file extension, using the :command:`swupd search` command. For example: 
:command:`sudo swupd search ${module_name}.ko`.
See :ref:`swupd-search` for more information. 

Request the module be added to |CL|
===================================

If the kernel module you need is already open source 
(e.g. in the Linux upstream) and likely to be useful to others, 
consider submitting a request to add or enable in the |CL| kernel.

Make enhancement requests to the |CL| distribution `on GitHub`_ .

Build and load an out-of-tree module
====================================

In some cases you may need an out-of-tree kernel module that is not 
available through |CL|.

You can build and load out-of-tree kernel modules, however you must:

* disable secure boot
* disable kernel module integrity checking
* build the module against new versions of the Linux kernel

.. note::

   Any time the kernel is upgraded on your Clear Linux system, you will 
   need to rebuild your out-of-tree modules.

This approach works well for individual development or testing. 
For a more scalable and customizable approach, consider using the 
`mixer tool`_ to provide a custom kernel and updates.

Build kernel module
-------------------

#. From a |CL| system, ensure you are running the *native* kernel. 
   Currently only the native kernel is enabled to build and load
   out-of-tree modules.

   .. code-block:: bash

      $ uname -r
      4.XX.YY-ZZZZ.native
   Ensure *.native* is in the kernel name

#. Install the `linux-dev` bundle to obtain the kernel headers, which are
   required for compiling kernel modules.

   .. code-block:: bash

      sudo swupd bundle-add linux-dev

#. Follow instructions from the kernel module source code to compile the 
   kernel module.


Load kernel module
------------------

#. Disable Secure Boot in your system's UEFI settings, if you have enabled
   it. The loading of new out-of-tree modules modifies the signatures Secure
   Boot relies on for trust. 


#. Disable signature checking for the kernel by modifying the kernel boot 
   parameters and reboot the system. 

   All kernel modules from |CL| have been signed to enforce kernel security. 
   However, out-of-tree modules break this chain of trust so this mechanism 
   needs to be disabled.
  
   .. code-block:: bash

      sudo mkdir -p /etc/kernel/cmdline.d
      echo "module.sig_unenforce" | sudo tee /etc/kernel/cmdline.d/allow-unsigned-modules.conf

#. Update the boot manager and reboot the system to implement the changed 
   kernel parameters.

   .. code-block:: bash

        sudo clr-boot-manager update
        sudo reboot

   .. note::

      :command:`clr-boot-manager update` does not return any
      console output if successful.

   
#. After rebooting, out-of-tree modules can be manually loaded with 
   :command:`insmod`. 

   .. code-block:: bash

      sudo insmod ${path_to_module}


Optional: Use `modprobe` to specify module options and aliases
--------------------------------------------------------------

Use :command:`modprobe` to load a module and set options.  

Because :command:`modprobe` can add or remove more than one module, due to 
modules having dependencies, a method of specifying what options are 
to be used with individual modules is useful. This can be done with 
configuration files under the :file:`/etc/modprobe.d` directory. 

.. code-block:: bash

   sudo mkdir /etc/modprobe.d

All files underneath the :file:`/etc/modprobe.d` directory 
that end with the :file:`.conf` extension specify module options to use when
loading. This can also be used to create convenient aliases for modules or 
they can override the normal loading behavior altogether for those with 
special requirements. 

You can find more info on module loading in the modprobe.d manual page:

.. code-block:: bash

   man modprobe.d

Optional: Configure kernel modules to load at boot
--------------------------------------------------

Use the :file:`/etc/modules-load.d` configuration directory to 
specify kernel modules to load automatically at boot.

.. code-block:: bash

   sudo mkdir /etc/modules-load.d

All files underneath the :file:`/etc/modules-load.d` directory 
that end with the :file:`.conf` extension contain a list of module names 
of aliases (one per line) to load at boot.

You can find more info on module loading in the modules-load.d manual page:

.. code-block:: bash

   man modules-load.d

.. _`on GitHub`: https://github.com/clearlinux/distribution 
.. _`mixer tool`: https://clearlinux.org/features/mixer-tool
