Running Zephyr on LiteX with VexRiscv
=====================================

This section contains tutorial on how to build and run a shell sample for the Zephyr RTOS running on the LiteX soft SoC with an RV32 VexRiscv CPU on `Future Electronics Avalanche Board <https://www.microsemi.com/existing-parts/parts/139680>`_ with a Microsemi PolarFire FPGA as well as in the `Renode open source simulation framework <https://renode.io>`_.

Building your system
--------------------

Zephyr
++++++

The support for VexRiscv is currently in the process of being merged into mainline Zephyr (`Pull Request #14915 <https://github.com/zephyrproject-rtos/zephyr/pull/14915>`_).
A working port can be obtained from Antmicro Zephyr fork on the `litex-vexriscv branch <https://github.com/antmicro/zephyr/tree/litex-vexriscv>`_.

If you want to skip building Zephyr manually, you can download a precompiled `ELF file <https://antmicro.com/projects/renode/litex_vexriscv--zephyr-shell.elf-s_750684-21ab1a23b11ad242acd76f85621380e15b377173>`_ and `binary file <https://antmicro.com/projects/renode/litex_vexriscv--zephyr-shell.bin-s_57912-448675102fa144363b4fb41336bdf02017c4090b>`_.

Building a shell sample
~~~~~~~~~~~~~~~~~~~~~~~

Clone the zephyr port::

   git clone -b litex-vexriscv https://github.com/antmicro/zephyr litex-vexriscv-zephyr
   cd litex-vexriscv-zephyr

.. note::

   For details on how to download and setup Zephyr toolchain, see `the official documentation <https://docs.zephyrproject.org/latest/getting_started/installation_linux.html#install-the-zephyr-software-development-kit-sdk>`_.

Configure the environment::

   export ZEPHYR_TOOLCHAIN_VARIANT=zephyr
   export ZEPHYR_SDK_INSTALL_DIR=/opt/zephyr-sdk/
   source ./zephyr-env.sh

Generate the project build files::

   cd samples/subsys/shell/shell_module
   mkdir build
   cd build
   cmake -DBOARD=arty_litex_vexriscv ..

Build it::

   make

As a result, you should find ``zephyr.elf`` and ``zephyr.bin`` in the ``zephyr`` folder.

Running
-------

Preparing the platform
++++++++++++++++++++++

.. tabs::

   .. group-tab:: Hardware

      Download a pregenerated bitstream of LiteX with VexRiscv and BIOS preloded to RAM:

      .. code-block:: text

         wget https://github.com/riscv/risc-v-getting-started-guide/releases/download/tip/bitstream-litex-vexriscv-avalanche.job

      Load it onto the Avalanche board using the `PolarFire FlashPro <https://www.microsemi.com/product-directory/programming/4977-flashpro#software>`_ tool.
      You can refer to the "Creating a Job Project from a FlashPro Express Job" section of the tool's official `User Guide <https://coredocs.s3.amazonaws.com/Libero/12_0_0/Tool/flashpro_express_ug.pdf>`_.


   .. group-tab:: Renode

      .. note::

         Support for LiteX in Renode will soon be available in pre-built packages.
         In the meantime, refer to `the documentation <https://renode.readthedocs.io/en/latest/advanced/building_from_sources.html>`_ for how to install from sources.

      Start Renode and create an instance of emulated LiteX+VexRiscv board:

      .. code-block:: text

         mach create "litex-vexriscv"
         machine LoadPlatformDescription @platforms/cpus/litex_vexriscv.repl


Loading Zephyr
++++++++++++++

.. tabs::

   .. group-tab:: Hardware

      In this example, Zephyr will be loaded onto the board over a serial connection.
      Download and run the ``litex_term.py`` script (shipped with `LiteX <https://github.com/enjoy-digital/litex>`_) on your host computer and connect it to the board via serial:

      .. code-block:: text

         wget https://raw.githubusercontent.com/enjoy-digital/litex/master/litex/utils/litex_term.py
         chmod u+x litex_term.py

         ./litex_term.py --serial-boot --kernel zephyr.bin /dev/ttyUSB1

   .. group-tab:: Renode

      To load the binary onto the emulated platform, just do:

      .. code-block:: text

         sysbus LoadELF @zephyr.elf

      .. note::

         LiteX bios plays a role of a bootloader and is required on hardware to run Zephyr.

         In Renode, however, you can load an ELF file to RAM and set CPU PC to its entry point, so there is no need for a bootloader.


Running Zephyr
++++++++++++++

.. tabs::

   .. group-tab:: Hardware

      Reset the board.

      You should see the following output:

      .. code-block:: text

         [TERM] Starting....

                 __   _ __      _  __
                / /  (_) /____ | |/_/
               / /__/ / __/ -_)>  <
              /____/_/\__/\__/_/|_|

          (c) Copyright 2012-2019 Enjoy-Digital
          (c) Copyright 2012-2015 M-Labs Ltd

          BIOS built on Apr  9 2019 14:40:45
          BIOS CRC passed (8c8ddc55)

         --============ SoC info ================--
         CPU:       VexRiscv @ 100MHz
         ROM:       32KB
         SRAM:      32KB
         L2:        8KB
         MAIN-RAM:  262144KB

         --========= Peripherals init ===========--
         Memtest OK

         --========== Boot sequence =============--
         Booting from serial...
         Press Q or ESC to abort boot completely.
         sL5DdSMmkekro
         [TERM] Received firmware download request from the device.
         [TERM] Uploading zephyr.bin (57912 bytes)...
         [TERM] Upload complete (7.6KB/s).
         [TERM] Booting the device.
         [TERM] Done.
         Executing booted program at 0x40000000




         uart:~$

   .. group-tab:: Renode

      Open UART window and start the emulation::

         showAnalyzer sysbus.uart
         start

      As a result, in the UART window you will see the shell prompt:

      .. code-block:: text

         uart:~$


Now you can use the UART window to interact with the shell, e.g.:

.. code-block:: text

   uart:~$ help
   Please press the <Tab> button to see all available commands.
   You can also use the <Tab> button to prompt or auto-complete all commands or its subcommands.
   You can try to call commands with <-h> or <--help> parameter for more information.
   Shell supports following meta-keys:
   Ctrl+a, Ctrl+b, Ctrl+c, Ctrl+d, Ctrl+e, Ctrl+f, Ctrl+k, Ctrl+l, Ctrl+u, Ctrl+w
   Alt+b, Alt+f.
   Please refer to shell documentation for more details.

   uart:~$ kernel version
   Zephyr version 1.14.0