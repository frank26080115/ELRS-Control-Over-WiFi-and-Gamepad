# Developer Commentary

This whole things started as an additional feature to my [Shrew ESC](https://github.com/frank26080115/Shrew-Dual-Brushed-ESC-with-ELRS). This ESC is supposed to be compact yet very powerful, but a bit intimidating to novice users. So I thought it might be nice to add a feature to make it easier for novice users to get started with, by eliminating the cost of buying a full blown RC transmitter.

In the backend code of ELRS, I added a few more handlers for the `ESPAsyncWebServer`, including one to handle `WebSocket`.

On the frontend, I added one new HTML page, one JavaScript file. I added the touchscreen joystick using https://github.com/bobboteck/JoyStick/ and then I also added the [Gamepad API](https://developer.mozilla.org/en-US/docs/Web/API/Gamepad_API/Using_the_Gamepad_API)

Adding the additional handlers to the source code seems to have caused a performance hit. Simultaneous HTTP requests stopped being serviced reliabily. So I modified all the HTML pages in the ELRS project use sequencial HTTP requests instead of simultaneous.

The frontend will send data into the backend using a WebSocket. The data is simply 16 channels worth of 16 bit data, and that data is written to the `ChannelData` array inside the backend, which is where decoded CRSF data goes.

To map which gamepad axis goes to which CRSF channel, there's a textbox in the web interface where I expect the user to write a custom "mixer". For example, to map a gamepad's first X axis to channel 2, write `channel[1] = MyGamepad.axes[0] * 819 + 992;` into that textbox;

For ELRS receivers: After receiving the packet from the WebSocket, some flags gets set to signal that new data has arrived. Then the original code will either send that data out the UART as a traditional CRSF packet, and/or set the servo PWM generator duty cycles.

For ELRS transmitters: I wrote a backdoor function into the `handset` class that represents the CRSF input from the radio handset. After receiving the packet from the WebSocket, the backdoor is used to make it look like the handset has new data.

Because the original ELRS code shutsdown most of the non-Wi-Fi functions, I made my new code only effective if the Wi-Fi auto-on interval is set super short, and under this condition, the Wi-Fi initialization code does not perform those shutdowns.

However, ther reason why those shutdowns happen is partly because of performance. It is easily observable that with Wi-Fi enabled, the PWM generator on the ESP8285 will exhibit a lot more jitter than usual.

I have considered directly connnecting a gamepad to the ESP32 via Bluetooth, but this proved to be extremely difficult for many reasons. The underlying framework that ExpressLRS is built upon simply does not support HID devices over Bluetooth Classic. Migrating to an updated framework is extremely risky. Also, adding a user interface to perform Bluetooth scanning and secure pairing will take a ton of development time. Often people skipped this and just hardcoded MAC addresses into PlayStation DualShock 4 controllers, but I found out the same trick does not work with the newer PlayStation 5 DualSense controllers. Even if I got all of this figured out, an additional challenge would be finding a way of implementing a mixer mechanism. Doing so with JavaScript is extremely easy since the smartphone is doing most of the work and provides an easy interface to edit the mixer, but if the gamepad is connected to the ESP32, then an entire interpreter would need to be implemented internally.
