.. _lib_zigbee_fota:

Zigbee FOTA
###########

The Zigbee firmware over-the-air (Zigbee FOTA) library provides Zigbee endpoint definition, which implements clusters responsible for transferring a firmware file through the Zigbee network.
The received data is passed as an upgrade candidate through the :ref:`lib_dfu_target` library API.

The library uses the endpoint to:

* Discover Zigbee OTA server by sending Match Descriptor Request broadcast messages.
* Query the discovered server periodically to check if there is a suitable image for the update.
* Transfer the new firmware using the ZCL OTA cluster commands.
* Allow the OTA server to manage the upgrade process using the ZCL OTA control commands.

The Zigbee FOTA functionality has been added to the :ref:`zigbee_light_switch_sample` sample.

OTA upgrade entities
********************

The following entities participate in the Zigbee OTA Upgrade process:

* OTA Upgrade Client – Runs on the target, that is the device that is being upgraded, and is responsible for downloading and installing the new firmware.
* OTA Upgrade Server – Provides the firmware image.
  The OTA Upgrade Server can be either a standalone third-party device or it can be instantiated on an nRF52840 DK using nRF Util's `DFU over Zigbee`_ procedure.

OTA upgrade process
*******************

The library uses :ref:`nrfxlib:zboss`'s ZCL API to download the image.
After downloading the Zigbee OTA image header, the stack verifies the following mandatory fields:

* Manufacturer ID
* Image type - Defined by the :option:`CONFIG_ZIGBEE_FOTA_IMAGE_TYPE` Kconfig option; it may be different than the MCUboot image type value.
* Hardware version
* Firmware version

If all values are accepted, the first fragment of the firmware image is downloaded.

After receiving the first fragment, the :ref:`lib_dfu_target` library is used to identify the type of the image that is being downloaded.
For example, the image type can be modem upgrades and upgrades handled by MCUboot.
Once the download is started, all received data fragments are passed to the :ref:`lib_dfu_target` library.
The library takes care of where the upgrade candidate is stored, depending on the image type that is being downloaded.

When the download is completed, the download client sends an appropriate event.
At this point, the received firmware is tagged as an upgrade candidate and the OTA server is queried for an update time.

Once the OTA server triggers the update process, the library sends a :cpp:enumerator:`ZIGBEE_FOTA_EVT_FINISHED <zigbee_fota::ZIGBEE_FOTA_EVT_FINISHED>` callback event.
When the consumer of the library receives this event, it should issue a reboot command to apply the upgrade.

Configuration
*************

To enable the Zigbee FOTA library, set the :option:`CONFIG_ZIGBEE_FOTA` Kconfig option.

To configure the Zigbee FOTA library, use the following options:

* :option:`CONFIG_ZIGBEE_FOTA_HW_VERSION`
* :option:`CONFIG_ZIGBEE_FOTA_DATA_BLOCK_SIZE`
* :option:`CONFIG_ZIGBEE_FOTA_ENDPOINT`
* :option:`CONFIG_ZIGBEE_FOTA_PROGRESS_EVT`
* :option:`CONFIG_ZIGBEE_FOTA_MANUFACTURER_ID`
* :option:`CONFIG_ZIGBEE_FOTA_IMAGE_TYPE`
* :option:`CONFIG_ZIGBEE_FOTA_COMMENT`
* :option:`CONFIG_ENABLE_ZIGBEE_FOTA_MIN_HW_VERSION`
* :option:`CONFIG_ZIGBEE_FOTA_MIN_HW_VERSION`
* :option:`CONFIG_ENABLE_ZIGBEE_FOTA_MAX_HW_VERSION`
* :option:`CONFIG_ZIGBEE_FOTA_MAX_HW_VERSION`

For detailed steps about configuring the Zigbee FOTA in a Zigbee application, see :ref:`ug_zigbee_configuring_components_ota`.

Limitations
***********

The Zigbee FOTA library has the following limitations:

* The endpoint definition in the library includes the endpoint ID, defined with :option:`CONFIG_ZIGBEE_FOTA_ENDPOINT`.
  When using the Zigbee FOTA library, this endpoint ID cannot be used for other endpoints.
* The Zigbee FOTA upgrades are currently only supported on the nRF52840 DK board (PCA10056).
* In case of an MCU reset between the completion of the OTA image transfer and a postponed firmware upgrade, the upgrade will be applied immediately.

API documentation
*****************

| Header file: :file:`include/zigbee/zigbee_fota.h`
| Source files: :file:`subsys/zigbee/lib/zigbee_fota/src/`

.. doxygengroup:: zigbee_fota
   :project: nrf
   :members:
