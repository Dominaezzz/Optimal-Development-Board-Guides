# ESP32-S3

Datasheet: https://www.espressif.com/sites/default/files/documentation/esp32-s3_datasheet_en.pdf

Technical Reference Manual: https://www.espressif.com/sites/default/files/documentation/esp32-s3_technical_reference_manual_en.pdf

Hardware Design Guidelines: https://www.espressif.com/sites/default/files/documentation/esp32-s3_hardware_design_guidelines_en.pdf

Schematic Checklist: https://docs.espressif.com/projects/esp-hardware-design-guidelines/en/latest/esp32s3/schematic-checklist.html

## I2C

There are three I2C controllers. [I2C0, I2C1](https://docs.espressif.com/projects/esp-idf/en/stable/esp32s3/api-reference/peripherals/i2c.html#) and [RTC_I2C](https://docs.espressif.com/projects/esp-idf/en/stable/esp32s3/api-reference/system/ulp-risc-v.html#rtc-i2c).

I2C0 and I2C1 are able to use any pins for SDA/SCL.
RTC I2C is only able to use [GPIO1/GPIO3 for SDA and GPIO0/GPIO2 for SCL](https://www.espressif.com/sites/default/files/documentation/esp32-s3_datasheet_en.pdf#table.6).

RTC I2C can be used by the [ULP](https://docs.espressif.com/projects/esp-idf/en/stable/esp32s3/api-reference/system/ulp-risc-v.html). This allows sensors to be checked during deep sleep, which is important for saving power. It also allows the ULP to be used as a 3rd core for polling sensors.

If you need to choose pins for an I2C bus, choose GPIO1 for SDA and GPIO2 for SCL.
If you need another I2C bus, choose GPIO3 for SDA and GPIO0 for SCL.
If you need yet another bus for some reason you may choose any pins.
If you need a 4th bus, you shouldn't be using this chip :D

| I2C     | SDA   | SCL   |
|---------|-------|-------|
| 1st bus | GPIO1 | GPIO2 |
| 2nd bus | GPIO3 | GPIO0 |
| 3rd bus |  ANY  |  ANY  |

GPIO1/GPIO2 is preferred over GPIO0/GPIO3 because the latter are [strapping pins](https://www.espressif.com/sites/default/files/documentation/esp32-s3_datasheet_en.pdf#section.3).

## Low Power Boards

If you have a battery powered development board that is intended to be used with BLE,
you should setup a 32.768kHz clock source as described [here](https://www.espressif.com/sites/default/files/documentation/esp32-s3_hardware_design_guidelines_en.pdf#subsubsection.2.4.2) or [here](https://docs.espressif.com/projects/esp-hardware-design-guidelines/en/latest/esp32s3/schematic-checklist.html#rtc-clock-source-optional).

If you already have a [PCF8563](https://www.nxp.com/products/analog-and-mixed-signal/real-time-clocks/real-time-clock-calendar:PCF8563) in your design, you can simply attach
the CLKOUT to GPIO15(XTAL_32K_P) with the right capacitor (20 pF, according to the hardware design guidelines) between them.

## USB ports

GPIO19 and GPIO20 should be exposed as a USB port (ideally [USB Type-C](https://en.wikipedia.org/wiki/USB-C)) and not used for anything else.
This alone, allows the port to be used for debugging, console or [USB OTG](https://docs.espressif.com/projects/esp-idf/en/stable/esp32s3/api-reference/peripherals/usb_host.html).

More info [here](https://docs.espressif.com/projects/esp-hardware-design-guidelines/en/latest/esp32s3/schematic-checklist.html#usb).

### Optional extras

Depending on the type of development board, adding more USB ports may be sensible.
If you expect your development board to be used as a USB device or USB host,
you should strongly consider including one or all of the following.

#### USB-UART

An additional port for a USB-UART bridge chip would be useful for logging.
This is important when the main USB port above is being used for OTG.
If this is skipped, strongly consider breaking out the main UART/serial pins,
GPIO43(TX) and GPIO44(RX) instead (like [this devkit](https://cdn.shopify.com/s/files/1/0617/7190/7253/files/T-Deck-Plus-lilygo_12.jpg?v=1723257113) does it).

#### USB external PHY

Like I said above, the main USB port can't be used for debugging whilst it
is being used for OTG. The real solution for this is to add [an external USB
transceiver/PHY (in addition to the internal one)](https://www.espressif.com/sites/default/files/documentation/esp32-s3_technical_reference_manual_en.pdf#figure.caption.290).
This allows developers to use the internal transceiver for USB-JTAG (debugging/logging)
and use the external transceiver for USB-OTG.

This is preferable to USB-UART if the UART/serial pins can be broken out instead.

Espressif has an overview of this feature [here](https://docs.espressif.com/projects/esp-iot-solution/en/latest/usb/usb_overview/usb_phy.html).

[TUSB1106PWR](https://www.ti.com/product/TUSB1106/part-details/TUSB1106PWR) is a transceiver that has been [known to work](https://github.com/espressif/esp-idf/issues/9366#issuecomment-1191278722).

Any GPIO pins can be used to connect to the transceiver but I suggest recommend
using:
- At least GPIO39, GPIO40, GPIO41 and GPIO42. They are JTAG pins which will no longer be needed once we have an external trasceiver.
- Any unused non-RTC pins. This means GPIO22 and above.

## GPIO Interrupts

Interrupt signals should be wired to any pins between GPIO4 and GPIO21 as they can
be used by the [ULP](https://docs.espressif.com/projects/esp-idf/en/stable/esp32s3/api-reference/system/ulp-risc-v.html) and they can also be used to wake the chip from deep sleep.

GPIO0..GPIO3 may also be used if they're not already used for I2C.
Do not use GPIO19 or GPIO20 for anything besides USB.

Avoid using any other pins for interrupt signals.

## SPI

In general you can pick any pins.
However, if you don't have any other use for the following pins, you should use them for SPI.

| Pin Name | GPIO Number |
|----------|-------------|
| MOSI/D0  | GPIO11      |
| MISO/D1  | GPIO13      |
| SCLK     | GPIO12      |
| CS0      | GPIO10      |

If you need another SPI bus, prefer using RTC pins for this. (This means GPIO21 and below)
The ULP can bitbang the SPI master protocol, which is important for low power usage.

## I2S (Microphones and speakers)

Prefer using non-RTC pins for this. (This means GPIO22 and above)

Why? The ULP can't do I2S.
