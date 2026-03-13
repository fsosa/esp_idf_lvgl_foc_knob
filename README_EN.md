# Development Log
- [x] GC9A01 display bring-up — add component `esp_lcd_gc9a01: "^2.0.0"`
- - Add the GC9A01 component library
- - Update the corresponding LCD pin assignments
- [x] Run LVGL
- - For custom development, add `esp_lvgl_port 1.4.0`; the LVGL version should be 8.4 (it will automatically pull LVGL 9). **In the future, if the official component is updated, you may encounter the issue of LVGL 9 being downloaded automatically — workaround: manually create a `components` folder and place both `esp_lvgl_port` and `LVGL 8` inside it.**
- - In SDK Configuration, update the LVGL parameters: color depth 16 (RGB565), enable "swap the 2 bytes", etc.
- - LVGL rotation parameters must be set, and after changing them the rotation direction must also be configured:
```c
const lvgl_port_display_cfg_t disp_cfg = {
        .io_handle = lcd_io,
        .panel_handle = lcd_panel,
        .buffer_size = EXAMPLE_LCD_H_RES * EXAMPLE_LCD_DRAW_BUFF_HEIGHT * sizeof(uint16_t),
        .double_buffer = EXAMPLE_LCD_DRAW_BUFF_DOUBLE,
        .hres = EXAMPLE_LCD_H_RES,
        .vres = EXAMPLE_LCD_V_RES,
        .monochrome = false,
        /* Rotation values must be same as used in esp_lcd for initial settings of the screen */
        .rotation = {
            .swap_xy = false,//change here
            .mirror_x = true,//change here
            .mirror_y = false,//change here
        },
        .flags = {
            .buff_dma = true,
        }
    };
    lvgl_disp = lvgl_port_add_disp(&disp_cfg);
    lv_disp_set_rotation(lvgl_disp, LV_DISP_ROT_NONE);
```

- [x] Export UI files from SquareLine Studio and run
- - In SquareLine Studio settings, change the LVGL header path to `lvgl.h`
- [x] LVGL physical input implemented
- [x] FOC motor running
- [x] Motor data used as LVGL input
- [x] Knob motor control running — motor haptic/force feedback controlling LVGL
- - The FOC PID has a special characteristic: it uses time-based D and I term calculation, so the built-in FOC PID must be used to drive FOC motion
- [x] USB HID — implemented Surface Dial, keyboard volume keys, media keys, scroll wheel, mouse, button, and other functions
- - Attempted to enable HID on screen switch: succeeded, but calling `tinyusb_driver_uninstall()` to unregister the USB device causes the host to not re-fetch the HID report descriptors when `HID_init` is called again
- - Final solution: call `HID_init` only once, keep HID always-on; do not unregister the HID device when switching back to other screens
- [x] Bluetooth HID
- - **Problem**: BLE enumerates four HID devices at once — keyboard, mouse, Surface Dial, and custom media device. After the first pairing, all four devices work correctly. After the ESP32 powers off and reboots, reconnecting to the already-paired Bluetooth devices causes one or more of the four devices to fail to work properly.
- - **Analysis**: When the Surface Dial is excluded, leaving only three HID devices, the connection is very stable — power-cycle reconnection causes no issues. Adding the Surface Dial triggers the problem. However, enumerating the Surface Dial alone does not cause failures. Preliminary conclusion: the issue is related to the Surface Dial's HID descriptor, which is a niche/uncommon descriptor that is rarely used.
- - **Resolution**: Switched to a different BLE HID demo to implement the Surface Dial HID descriptor — now works correctly. Reference project: https://www.bilibili.com/video/BV1AC41157nm
- [] WiFi web server
- -  [x] WiFi provisioning: open a web page via AP mode, enter WiFi SSID and password to connect to the user's own network
- - []  Web OTA: implement over-the-air firmware updates via web page
- - []  Authorization code: retrieve the ESP32's unique MAC address, encrypt it with an algorithm; users enter the authorization code on the web page to unlock full firmware features.
- - []  Provide a complete binary: after enabling WiFi, the web page displays the chip's unique MAC address; the user provides the MAC to the author, who decrypts and generates an authorization code; the user enters the code on the web page, and the ESP32 validates it locally before unlocking all features.
- - []  Web page: customize knob button functions, haptic intensity, and knob motion mode
- - []  Web page: upload an image to replace the icon
- [] MIDI input
- [] MQTT smart home

...
