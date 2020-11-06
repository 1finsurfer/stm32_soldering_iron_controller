# stm32_soldering_iron_controller

<!-- MarkdownTOC -->

* [Status](#status)
* [Warning](#warning)
* [Backup First!](#backup-first)
* [Instructions](#instructions)
* [History](#history)
  * [Working](#working)
  * [Todo](#todo)
* [Where are Docs?](#where-are-docs)
* [Compatibility](#compatibility)

<!-- /MarkdownTOC -->

<a id="status"></a>
## Status

* Tested on STM32F072 (on a QUICKO T12 board). Needs further testing on other MCUs / boards
* Should be more broadly compatible with other MCUs from STM32 family
* We need to tweak and do a little back-porting of this code
* To get it to work again back on the STM32F103 (original v2.1 hardware)
* DavidAlfa has worked to simplify the project structure and build configuration
* To make a unified codebase that's more easy to share across different boards / hardware

<a id="warning"></a>
## Warning

Newer hardware is often inferior! With bad low quality component(s) such as:

* Bad Op-Amp
* Bad 3v3 Regulator

Which can then result in bad performance / bad temperature regulation.

* It is recommended to chech these above suspect components. And if necessary order better alternatives parts and replace them (before proceeding further).
* This is especially true for many of the newer v2.1 boards and v3.x boards by KSGER.

<a id="backup-first"></a>
## Backup First!

The simplest way to backup is actually to buy a new MCU. Then remove the original MCU with hot air, or chipquik, or bizmuth low temperature solder. And set it aside in case the Open Firmware here fails to work. Since there are so many versions of the hardware PCB. KSGER and friends keeps changing them all the time.

Unfortunately we cannot backup the firmware with programmer PC link. This is because the program in the MCU memory is security protected. It cannot be read fully. So the original commercial firmware cannot be backup over SWD protocol, or JTAG whatever. Not for these STMF1xx series MCUs. Sorry to say that. 

<a id="instructions"></a>
## Instructions

* Developed on STM32CubeIDE - download and use that. This IDE includes the MX code generator
* And alternative IDE is PlatformIO. Recommended but not tested
* Being STM32 there is a code generation aspect for the chip pinout
* Check the .ioc file in STM32CUBEMX, all settings are there
* Use the same pin names, check setup.h for setting few parameters

More instructions to follow later.

<a id="history"></a>
## History

* Custom firmware for the chinese kseger soldering iron controller
* Original version by PTDreamer
* Updated branch - for v2.1s, from newer fork: `LuckyTomas`, flawless_testing` branch
* Merged to master, with Docs, and minor update by Dreamcat4

Since DavidAlfa rewrite:

* Compatibility for STM32F072 added by DavidAlfa
* Adapted and refined with many improvements by DavidAlfa
* Changelog file started by DavidAlfa, available here as [Changelog.md](Changelog.md)


<a id="working"></a>
### Working

* Buzzer: Untouched, working the same as original fw.
* Oled: It uses hardware SPI + DMA to transmit each row to the oled screen (128bytes), sends the change row command between chunks and stops DMA after last row is sent. DMA is triggered in each update_display() call.
* To prevent graphical artifacts, oled_draw() waits for the SPI DMA to stop before drawing the oled buffer.
* Rotary encoder: untouched and works well.

<a id="todo"></a>
### Todo

* I2C eeprom - better than writing to 

<a id="where-are-docs"></a>
## Where are Docs?

* Docs have been split out into seperate 'docs' repo. Because the size of assets kept ballooning in git
* This repo is just only for source code now. This is simply code fork of PTDreamer meant for v2.1s Hardware

Well which Docs are you looking for?

* **[New Docs, Full Repo](https://github.com/dreamcat4/t12-t245-controllers-docs)**
* **[Backup of PTDreamer Blog](https://github.com/dreamcat4/t12-t245-controllers-docs/tree/master/research/ptdreamer)** 
* **[Hardware Compatibility Docs](https://github.com/dreamcat4/t12-t245-controllers-docs/tree/master/controllers/stm32-t12-oled)**
* **[How to Compile & Flash This Code with Official ST Toolchain](https://github.com/dreamcat4/t12-t245-controllers-docs/tree/master/tools/software/STM32CubeIDE)**
* *Docs for Compiling / Flashing with VSCode and PlatformIO* - TBD. Not done.

<a id="compatibility"></a>
## Compatibility







