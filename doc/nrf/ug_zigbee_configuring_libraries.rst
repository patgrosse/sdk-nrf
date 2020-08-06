.. _ug_zigbee_configuring_libraries:

Configuring Zigbee libraries in |NCS|
#####################################

The Zigbee protocol in |NCS| can be customized by enabling and configuring several libraries.

This page lists options and steps required for configuring the available libraries.

.. _ug_zigbee_configuring_components_handler:

Configuring default signal handler
**********************************

The default signal handler provides a default logic to handle ZBOSS stack signals.

The default signal handler can be used by calling the :cpp:func:`zigbee_default_signal_handler()` in the application's :cpp:func:`zboss_signal_handler()` implementation.
For more information, see :ref:`lib_zigbee_signal_handler`.

.. _ug_zigbee_configuring_components_ota:

Configuring Zigbee FOTA
***********************

The Zigbee Over The Air Device Firmware Upgrade (Zigbee FOTA) library provides a mechanism to upgrade the firmware of the device through the Zigbee network.

For more information about Zigbee FOTA, see :ref:`lib_zigbee_fota`.

The Zigbee FOTA can be enabled and configured using the following Kconfig options:

* :option: `CONFIG_ZIGBEE_FOTA` - This option must be enabled to use the Zigbee FOTA.
* :option: `CONFIG_ZIGBEE_FOTA_ENDPOINT` - This option allows to select the endpoint ID, on which the ZCL OTA cluster will be implemented. This endpoint ID must not be used in the application code for other purposes.
* :option: `CONFIG_ZIGBEE_FOTA_DATA_BLOCK_SIZE` - This option allows to specify the maximum transferred Zigbee OTA image chunk size.
* :option: `CONFIG_ZIGBEE_FOTA_HW_VERSION` - This option allows to specify the device's hardware version, used for selecting the correct Zigbee OTA image.
* :option: `CONFIG_ZIGBEE_FOTA_MANUFACTURER_ID` - This option allows to specify the device's manufacturer ID, used for selecting the correct Zigbee OTA image.
* :option: `CONFIG_ZIGBEE_FOTA_IMAGE_TYPE` - This option allows to specify the Zigbee OTA image type.

Due to the fact, that the Zigbee OTA DFU performs the upgrade through the DFU target library, the are several non-Zigbee options that are required to configure the update process:
* :option: `CONFIG_MCUBOOT_IMAGE_VERSION` - This option specifies the current image version.
* :option: `CONFIG_DFU_TARGET_MCUBOOT` - This option enables updates that are performed by MCUboot.
* :option: `CONFIG_IMG_MANAGER` - This option enables the support for managing DFU image downloaded using MCUboot.
* :option: `CONFIG_IMG_ERASE_PROGRESSIVELY` - This option instructs MCUboot to erase flash memory progressively, which allows to avoid long wait times at the beginning of the DFU process.

Enabling Zigbeee FOTA in an application
=======================================

If the application would like to use Zigbee FOTA functionality, there are several code snippets, that should be added to its main file:

* The Zigbee OTA DFU library provides only the definition of the OTA endpoint. The application has to include it inside the device context:

.. code-block:: c

    #include <zigbee_fota.h>
    extern zb_af_endpoint_desc_t ota_upgrade_client_ep;
    ZBOSS_DECLARE_DEVICE_CTX_2_EP(<your_device>_ctx, ota_upgrade_client_ep, <your_application>_ep);

* The application is informed about the update status though a callback. The callback should at least reboot the device once the firmware update is completed:

.. code-block:: c

    static void ota_evt_handler(const struct zigbee_fota_evt *evt)
    {
        switch (evt->id) {
        case ZIGBEE_FOTA_EVT_FINISHED:
            LOG_INF("Reboot application.");
            sys_reboot(SYS_REBOOT_COLD);
            break;
        }
    }

* Apart from the library initialization, the application must pass ZCL events to the Zigbee FOTA library. If the application does not implement additional ZCL event handlers, the Zigbee FOTA handler may be passed directly to the ZBOSS:

.. code-block:: c

    /* Initialize Zigbee FOTA download service. */
    zigbee_fota_init(ota_evt_handler);
    /* Register callback for handling ZCL commands. */
    ZB_ZCL_REGISTER_DEVICE_CB(zigbee_fota_zcl_cb);

* The periodical OTA server discovery must be started from the signal handler. The application should pass received signals to the Zigbee FOTA library:

.. code-block:: c

    void zboss_signal_handler(zb_bufid_t bufid)
    {
        /* Pass signal to the OTA client implementation. */
        zigbee_fota_signal_handler(bufid);
        ...

* In order to inform the MCUboot about successful device firmware upgrade, the application has to call the following function once it is sure that all intended functionalities work after the upgrade:

.. code-block:: c

    boot_write_img_confirmed();

Generating Zigbee FOTA upgrade image
====================================

By enabling the Zigbee OTA DFU, the west tool will automatically generate the upgrade image.
In order to specify the target device of the generated image, the following additional Kconfig options can be used:
* :option: `CONFIG_ZIGBEE_FOTA_COMMENT` - This option allows to specify human-readable image name.
* :option: `CONFIG_ENABLE_ZIGBEE_FOTA_MIN_HW_VERSION` and :option: `CONFIG_ZIGBEE_FOTA_MIN_HW_VERSION` - This option allows to specify the minimum hardware version of the device, that will accept the generated image.
* :option: `CONFIG_ENABLE_ZIGBEE_FOTA_MAX_HW_VERSION` and :option: `CONFIG_ZIGBEE_FOTA_MAX_HW_VERSION` - This option allows to specify the maximum hardware version of the device, that will accept the generated image.

The manufacturer ID, image type and version of the generated image are obtained from the application settings.
