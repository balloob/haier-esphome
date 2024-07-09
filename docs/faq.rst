Frequently Asked Questions
==========================

.. contents:: Table of Contents

Compatibility and Support
-------------------------

**Q:** Is there a list of supported models?
*******************************************

**A:** No, Haier has a wide range of models, and different models are available in different regions. Sometimes the same model can have different names depending on the country where it is sold. The only way to know if your model is supported is to try it out.

**Q:** How can I be sure that your component will support the selected model when buying a new AC?
**************************************************************************************************

**A:** Unfortunately, there is no guarantee until you try it. The best way is to check the user manual. Usually, if the AC supports ESP modules, there will be information about which application the native module works with. So far, all models that work with smartAir2, hOn, and EVO applications also work with this component, although sometimes additional tuning may be required.

**Q:** How ESP module should be connected to the AC?
****************************************************

**A:** On AC side there is a conector ususally marked as CN34 or CN35 it is a 4 pin connector with pins marked as RXD, TXD, GND, +5V. It is 4 pin 5264 molex connector. On ESP side used USB Type-A connector (it is not real USB just UART with USB connector) or JST SM04B-GHS-TB connector.

**Q:** Can I use ESP8266 with this component for my Haier AC?
*************************************************************

**A:** ESP8266 is powerful enough to handle communication with Haier AC. But it has limited resources and in case you also want some advanced features of ESPHome such as web_server, mqtt, etc. it may not be enough. It is recommended to use ESP32.

**Q:** Can I use an ESP board with a USB Type-A connector for my Haier AC that has a USB Type-A port?
*****************************************************************************************************

**A:** The USB Type-A port on your Haier AC operates using UART protocol, not standard USB communication. Most ESP32 modules use a separate chip (like CP2102 or CH340) for USB communication, which only supports USB protocol and cannot interface with UART protocol directly. Therefore, these cannot communicate with your AC's UART-over-USB Type-A port.

However, ESP modules equipped with the ESP32-S3 chip have native USB support and can potentially communicate using UART protocol if the USB Type-A connector is directly connected to the ESP pins without an intermediary chip. This setup allows for creating an ESPHome configuration that utilizes the same pins for UART communication.

Currently, the `M5Stack AtomS3U <https://shop.m5stack.com/products/atoms3u>`_ is one confirmed board that supports this configuration, though there may be others.

**Sample ESPHome Configuration for M5Stack AtomS3U:**

.. code-block:: yaml

    esphome:
      name: haier
      platformio_options:
        board_build.flash_mode: dio

    esp32:
      board: esp32-s3-devkitc-1
      framework:
        type: arduino

    wifi:
      ssid: !secret wifi_ssid
      password: !secret wifi_password

    uart:
      baud_rate: 9600
      tx_pin: 19
      rx_pin: 20

    logger:
      level: WARN

    climate:
      - platform: haier
        name: Haier AC
    
This configuration is a starting point for integrating your Haier AC with an ESP32-S3 based board.


Troubleshooting
---------------

**Q:** I connected the ESP module but all I can see in the logs is "Answer timeout for command 61, phase SENDING_INIT_1" and nothing is working. What should I do?
******************************************************************************************************************************************************************

**A:** This warnings means that the ESP module is not receiving any response from the AC. It can be caused by several reasons:

- **Configuration Issues:** There might be issues with your ESPHome configuration.
- **Hardware Problems:** The problem could lie with the ESP module or other hardware components.
- **Wiring Issues:** Incorrect wiring or problems with the pins could be causing communication failures.
- **Protocol Mismatch:** The AC might use a different protocol for communication or may not support serial communication at all.

**Troubleshooting Steps:**

1. Check the Wiring: Ensure all connections are secure. Try using different pins for communication.
2. Test with Different Hardware: If possible, test with a different ESP module or AC unit to isolate the issue.
3. Use a Simulator: If you're familiar with C++ and cmake, consider using a simulator to diagnose the issue. I have simulator applications for both the hOn and smartAir2 protocols:

   - hOn Simulator: `hOn simulator application <https://github.com/paveldn/HaierProtocol/tree/main/tools/hon_simulator>`_
   - smartAir2 Simulator: `smartAir2 simulator application <https://github.com/paveldn/HaierProtocol/tree/main/tools/smartair2_simulator>`_

These simulators are compatible with Windows and Linux.

**Using the Simulator:**

1. Connect your ESP32 to a PC using a TTL to USB converter.
2. Start the appropriate simulator. You should see requests from the ESP module in the simulator's console.

   - No Requests Seen: If the simulator does not show any requests from the ESP, there may be an issue with the TX wire.
   - No Responses in ESPHome Logs: If you see requests in the simulator but no responses in the ESPHome logs, the problem could be with the RX wire.
   - Requests and Responses Visible: If both requests and responses are visible, your ESP is working correctly. The issue may be on the AC side.

**Q:** I am constantly seeing the warning "Component haier took a long time for an operation" in the logs, is this normal?
**************************************************************************************************************************

**A:** Yes, this message has always been there, but in the latest versions, it was changed to a warning. This message is shown when the component is too busy processing something. The biggest delays are usually related to logging and operations with the web server. If you want to reduce the chance of seeing this message, you can decrease the log level to "warning". The warning level is sufficient for a fully working system. This message is usually not a problem but can indicate that some other component (like Wi-Fi) that should work in real-time is suffering from delays. For more information, refer to: https://github.com/esphome/issues/issues/4717

**Q:** I am seeing the warning "Answer handler error, msg=01, answ=03, err=7". What should I do?
************************************************************************************************

**A:** This warning means that the AC denied the control command. It can happen in two cases: either the AC is using a different type of control or the structure of the status packet is different. You can try using the `control_method: SET_SINGLE_PARAMETER`. If that doesn't help, you can try to figure out the size of different parts of the status packet using this method: `Haier protocol overview <./docs/protocol_overview.rst>`_. If nothing helps, you can create an issue on GitHub.

**Q:** What does the "Unsupported message received: type xx" message in the logs indicate?
*******************************************************************************************

**A:** This message may appear for several reasons:

1. **Slow AC Response:** Your AC unit is responding slowly to requests, consider increasing the `response_timeout` parameter from its default value of 200 ms to 400 ms.
2. **Overloaded ESP:** Your ESP module is too busy to process messages in time, increasing `response_timeout` won't resolve the issue. Instead, try disabling some components, lowering the log level, or upgrading to a more powerful ESP board.
3. **Unrecognized Messages:** Your AC might be sending new types of messages that the component does not recognize. If adjusting the timeout and optimizing ESP performance don't help, capture the logs and create an issue on GitHub for further assistance.

**Q:** My ESP is communicating with the AC, but I can't control it. Or I can control it, but my sensors show the wrong information.
***********************************************************************************************************************************

**A:** Most likely, you have one of two problems: either the wrong control method or the wrong status packet structure. You can try using the `control_method: SET_SINGLE_PARAMETER`. If that doesn't help, you can try to figure out the size of different parts of the status packet using this method: `Haier protocol overview <./docs/protocol_overview.rst>`_.

Feature Requests
----------------

**Q:** My AC has a cool feature that is not supported by your component. Can you add it?
****************************************************************************************

**A:** First, you need to figure out if the feature is supported by the serial protocol. There is some functionality that is supported only by the IR remote. The easiest way to check is by using the IR remote:

- Start capturing logs from your ESP modules.
- Wait 10 - 15 seconds.
- Enable the feature using the remote.
- Wait 10 - 15 seconds.
- Disable the feature using the remote.
- Wait 10 - 15 seconds.
- Stop capturing logs.
- Check the logs for changes in the status packet.

If all messages that look like this "Frame found: type 02, data: 6D 01 ..." are the same, the feature you want to add is not supported by the serial protocol. If you see some changes in the status packet, you can create a feature request on GitHub with the logs you collected.
