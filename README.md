# Yamaha Synth LCD Firmware Patches

It's common for hardware devices to include LCD displays built around HD44780-compatible LCD controllers. Despite first being manufactured in the early 80s, displays featuring this controller are so common that technicians can easily find compatible replacements for 40 year old[^1] displays today.

## The Issue

One common issue encountered when replacing LCD screens with a single row of 16 characters is that only half of the character data will be printed to the new display. 

## The Cause

This is because different LCD screens, despite being built around HD44780-compatible controllers, can display their character data in different ways. Internally, the HD44780 LCD controller contains 80 bytes of DDRAM (Display Data RAM). This is organised into two lines of 40 ASCII characters. From the KS0066U[^2] datasheet: 
> When 1-line display mode, DDRAM address is from '00H' to '4FH'.
> In 2-line display mode, DDRAM address in the 1st line is from '00H' to '27H', and DDRAM address in the 2nd line is from '40H' to '67H'.

Even though these lines are much wider than most LCD screens, the HD44780 supports 'scrolling' horizontally through the data.

To save on manufacturing cost[^3], some HD44780-compatible LCD screens with a single row of 16 characters print the first 8 characters as normal, and then print the contents of the second 'line' in the second 8 spaces. This behaviour is so common that the firmware of many common devices are designed around it, including many synthesizers. Fortunately, if you purchased an LCD screen that functions differently, this behaviour can be corrected in the firmware with minimal changes.

## The Fix

In many cases, this issue can be fixed by only changing a few bytes in the ROM. Usually by changing the offset passed to the 'Set DDRAM Address' command in the LCD print routine. This repository aims to collect the 'recipes' for fixing this issue in various synthesizers.

The individual file entries contain the necessary changes to make to the firmware ROMs.

## How-To

The individual 'recipe' files contained in this repository follow this format:

|Offset in ROM|Original Contents|Change To|Function|
|--|--|--|--|
|`0xABCD`|`0xC8`|`0x90`|LCD Print subroutine - Check for end of 'line 1'|
|`0xDBCA`|`0xC0`| `0x88` |LCD Print subroutine - Set position to start of 'line 2'|

Using a hex editor, change the byte at the specified offset to match the data specified in the 'Change To' column.

## Contributions

If you'd like to contribute a recipe for fixing a particular ROM, the preferred way to contribute to the project is by raising a *Pull Request* on Github. If you do not have a Github account, feel free to send your contribution to the lead maintainer (@ajxs).

## Requests

If there's a particular synth you'd like to see added, feel free to contact the lead maintainer (@ajxs).

[^1]: The earliest reference to the HD44780 that I can find online is a [https://archive.org/details/Hitachi-DotMarixLiquidCrystalDisplayControllerandDriverLCD-IIHD44780UsersManualOCR/page/n1/mode/2up]('preliminary' user's manual), dated March 1981.

[^2]: The KS0066U is a HD44780-compatible LCD controller manufactured by Samsung.

[^3]: Hitachi produced a companion segment driver chip, the LCD44100, which could expand the LCD segments rendered from a contiguous block of DDRAM. The HD44780 was only designed to drive displays with a width of 8 characters. To avoid the need for the extra segment driver chip, the LCD display manufacturer could configure the LCD to simply take the second block of 8 characters from the second line.
