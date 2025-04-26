# General tips not specific to a single chip

## (Micro) SD Cards

Micro SD cards have 8 pins and their slots can have up to 3 pins.

### Card Pins

The SD card itself can run in the following "modes".

| Mode     |      |      |      |      |      |         |
|----------|------|------|------|------|------|---------|
| SPI      | SCLK | MISO | MOSI |      |      | CS      |
| SD 1-bit | CLK  | CMD  | DAT0 |      |      |         |
| SD 4-bit | CLK  | CMD  | DAT0 | DAT1 | DAT2 | DAT3/CD |

[source](
https://academy.cba.mit.edu/classes/networking_communications/SD/SD.pdf#%5B%7B%22num%22%3A83%2C%22gen%22%3A0%7D%2C%7B%22name%22%3A%22XYZ%22%7D%2C55%2C721%2C0%5D).

All signals in a column use the same pin on the SD card.

SPI mode can be used by almost any microcontroller but SD mode requires special hardware.
The [ESP32](https://docs.espressif.com/projects/esp-idf/en/stable/esp32/api-reference/storage/sdmmc.html), [ESP32-S3](https://docs.espressif.com/projects/esp-idf/en/v5.4.1/esp32s3/api-reference/storage/sdmmc.html) and [ESP32-P4](https://docs.espressif.com/projects/esp-idf/en/v5.4.1/esp32p4/api-reference/storage/sdmmc.html) have this special hardware.

Ideally all pins of the SD card are connected on the development board (if the chip supports SD mode) but sometimes there aren't enough GPIO pins or other peripherals need to take priority.

When running low on pins, the CS pin of the card can be placed on a GPIO expander. Like how [this development board](https://www.waveshare.com/wiki/ESP32-S3-Touch-LCD-4.3B) does it in its [schematic](https://files.waveshare.com/wiki/ESP32-S3-Touch-LCD-4.3B/ESP32-S3-Touch-LCD-4.3B-Sch.pdf).
Only do this as a last resort, as it may require special libaries to use the SD card.

### Slot Pins

SD Card slots often have a "Card Detect" (CD) pin and a "Write Protect" (WP) pin.

Ideally expose both pins, CD is more important than WP, as it's the most reliable
way for to dectect if a card is present, inserted or removed.
The DAT3/CD could be used for this too but it can only be used once at start up to
detect if the SD is present or inserted, after this it can't be used again.
It's more important to expose the CD pin from the slot itself.

These pins are usually (some chips may need the CD/WP pins to be connected directly) fine to go on a GPIO expander if needed but it should be one
that supports interrupts like the [PCF8574](https://www.ti.com/lit/ds/symlink/pcf8574.pdf).
This allows the main chip to get an interrupt when the card is inserted/removed.
