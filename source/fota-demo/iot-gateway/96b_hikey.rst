.. highlight:: sh

.. _iot-gateway-96b_hikey:

IoT Gateway: 96Boards HiKey
===========================

This page describes how install a customized Debian distribution onto
the `96Boards HiKey <http://www.96boards.org/product/hikey/>`_
reference platform for using the HiKey as an IoT Gateway device.

.. note::

   Before following these instructions, make sure to set up your
   :ref:`IoT Device <iot-devices>` first.

If you're not familiar with 96Boards or the HiKey, detailed
documentation is available from `96boards.org <https://96boards.org>`_
as well as LeMaker.

- https://www.96boards.org/documentation/ConsumerEdition/HiKey/README.md/
- `LeMaker Hardware User Guide
  <https://www.96boards.org/wp-content/uploads/2015/02/HiKey_User_Guide_Rev0.2.pdf>`_

.. note::

   A `96Boards UART daughter board
   <https://www.seeedstudio.com/96Boards-UART-p-2525.html>`_ is useful
   for accessing the HiKey's UART.

.. contents::
   :local:

Download Software
-----------------

Download the following reference platform components.

**Note**: This requires a working install of fastboot and Python on
your host PC.

UEFI release for HiKey (build 147)::

    wget http://builds.96boards.org/snapshots/reference-platform/components/uefi/147/release/hikey/l-loader.bin
    wget http://builds.96boards.org/snapshots/reference-platform/components/uefi/147/release/hikey/fip.bin
    wget http://builds.96boards.org/snapshots/reference-platform/components/uefi/147/release/hikey/ptable-linux-8g.img
    wget https://raw.githubusercontent.com/96boards/burn-boot/master/hisi-idt.py

Debian Stretch IoT Reference Platform Build for HiKey (build 28)::

    wget http://builds.96boards.org/snapshots/reference-platform/debian-iot/28/hikey/hikey-boot-linux-20170204-28.uefi.img.gz
    wget http://builds.96boards.org/snapshots/reference-platform/debian-iot/28/hikey/hikey-rootfs-debian-stretch-iot-20170204-28.emmc.img.gz

Extract the .gz files::

    gunzip hikey-*

Place your HiKey into Recovery Mode
-----------------------------------

- Turn off the HiKey board
- Make sure pin1-pin2 and pin3-pin4 on J15 are linked (J601 on LeMaker HiKeys; top left side of the board)
- Connect HiKey micro-USB to PC with USB cable
- Power the HiKey board

.. image:: /_static/fota-demo/HiKeyRecovery.jpg
   :align: center

See `HiKey's board recovery documentation
<https://github.com/96boards/documentation/blob/master/ConsumerEdition/HiKey/Installation/BoardRecovery.md#set-board-link-options>`_
for more information about the recovery process.

Flash the software
------------------

::

    sudo python hisi-idt.py --img1=l-loader.bin
    sudo fastboot flash ptable ptable-linux-8g.img
    sudo fastboot flash fastboot fip.bin
    sudo fastboot flash boot hikey-boot-linux-20170204-28.uefi.img
    sudo fastboot flash system hikey-rootfs-debian-stretch-iot-20170204-28.emmc.img

Boot the device
---------------

Remove the jumper from J15 pins 3-4, and power cycle the board. The
GRUB bootloader will boot into the software image within 5 seconds.

Some error messages are expected during the first boot:

- Failed to start Raise network interfaces.
- Failed to start Tinyproxy lightweight HTTP Proxy.
- Failed to start Bluetooth Init Script for HiKey.
- Failed to start Wait for Network to be Configured.
- Failed to start OpenBSD Secure Shell server.
- Failed to start /etc/rc.local Compatibility.

You may see the "Raise network interfacs" and "Tinyproxy lightweight
HTTP Proxy" on subsequent boots as well.

**At time of writing (Jan 23, 2017), HDMI must be unplugged when
booting. The issue is tracked in**
https://bugs.96boards.org/show_bug.cgi?id=473.

You will automatically be logged in as root on the HiKey's serial
console. You can now proceed to connect to the network for the first
time.

Connect the HiKey gateway to your network via WiFi or use an Ethernet dongle
----------------------------------------------------------------------------

Use this command at the HiKey console to connect your gateway to a WiFi network (if using a USB Ethernet dongle, this step is not required)::

    nmcli device wifi connect <SSID> password <PASSWORD>

Set the location of gitci.com in /etc/hosts
-------------------------------------------

To allow connected IoT Devices to connect to the hawkBit server on
your workstation via the HiKey gateway, you will need to add an entry
to the /etc/hosts file for gitci.com which points at the IP address of
your workstation that is hosting your hawkBit instance,
i.e. 192.168.0.43. If you do not configure this in your /etc/hosts
file, you will not be able to connect to your hawkBit instance. ::

    # Example; your workstation's IP address may be different.
    echo "192.168.0.43 gitci.com" >> /etc/hosts

Done!
-----

Congratulations! You should have previously configured an IoT Device
using the previous pages in this guide. It will automatically connect
to the HiKey Gateway via 6LoWPAN, and be able to communicate with the
hawkBit server. If you haven't done so yet, the instructions are at
:ref:`iot-devices`.
