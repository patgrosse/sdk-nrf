.. _lib_zigbee_fota:

Zigbee FOTA
###########

The Zigbee firmware over-the-air (Zigbee FOTA) library provides functions for transferring a firmware file through a Zigbee network as an upgrade candidate to the DFU target that is used in the :ref:`lib_dfu_target` library.
The update process is started automatically, once an appropriate firmware file is found on a OTA server inside the Zigbee network.

The library provides a definition of a Zigbee endpoint, that implements ZCL OTA client cluster.
Once included, the library will communicate through it in order to:
* Discover Zigbee OTA server.
* Poll for new firmware images.
* Transfer the new firmware.
* Control the upgrade process.

Overview
********

There are two entities participate in the Zigbee OTA Upgrade process:

    OTA Upgrade Client – the OTA Upgrade Client runs on a target, that is the device that is being upgraded, and is responsible for downloading and installing the new firmware. The OTA Upgrade Client incorporates a DFU controller that manages the DFU process (see Device Firmware Update process). This means that each OTA target can be in a different state regarding the OTA process at a given point of time.
    OTA Upgrade Server – the OTA Upgrade Server is a server that provides a firmware image. The OTA Upgrade Server can be a standalone 3rd party device, or it can be instantiated as an nRF52840 DK in conjunction with the nRF Util tool.

The library uses the ZCL API to download the image.
After downloading the Zigbee OTA image header, all mandatory fields are verified (manufacturer ID, image type, hardware and firmware version).
The image type in Zigbee header is defined by the :option:`CONFIG_ZIGBEE_FOTA_IMAGE_TYPE` Kconfig option and may be different that the MCUboot image type value.
If all values are accepted, the first frament of the firmware image is downloaded.
After receiving the fragment, the :ref:`lib_dfu_target` library is used to identify the type of the image that is being downloaded.
Examples of image types are modem upgrades and upgrades handled by MCUboot.

Once the download has been started, all received data fragments are passed to the :ref:`lib_dfu_target` library.
The :ref:`lib_dfu_target` library takes care of where the upgrade candidate is stored, depending on the image type that is being downloaded.

When the download client sends the event indicating that the download has completed, the received firmware is tagged as an upgrade candidate, and the OTA server is queried for an update time.
Once the OTA server triggers the update process, the library sends a :cpp:enumerator:`ZIGBEE_FOTA_EVT_FINISHED<zigbee_fota::ZIGBEE_FOTA_EVT_FINISHED>` callback event.
When the consumer of the library receives this event, it should issue a reboot command to apply the upgrade.

The Zigbee FOTA library can be used in the :ref:`zigbee_light_switch_sample` sample.



API documentation
*****************

| Header file: :file:`include/zigbee/zigbee_fota.h`
| Source files: :file:`subsys/zigbee/lib/zigbee_fota/src/`

.. doxygengroup:: zigbee_fota
   :project: nrf
   :members:
