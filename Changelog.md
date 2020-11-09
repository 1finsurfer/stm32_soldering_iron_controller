# Changelog

## 05-Nov-2010 - DavidAlfa Rewrite

**Global Changes:**

* Deleted *oops* orphan file "temp.c"
* Check setup.h for more hints.
* Added a global define "Soft_SPI" that will disable hardware SPI and bitbang the data by software, useful when the display simply doesn't work and you want to trace the problem.
* Tidy everything so STM32CUBEMX doesn't destroy all the code after each code generation.


[Core/Src/main.c](Core/Src/main.c):

* Clear up the main, moving lots of init and variables stuff to their respective functions.
* Added CheckReset function to enable resetting defaults
* All the ADC init stuff moved to MX_ADC_Init
* ADC set to single channel, as the STM072 has only one ADC.
* Set ADC sampling to 13 cycles, more than double needed to fully charge the ADC pin. It was set to 239 cycles in * PTDreamer code, no sense.
* PWM timer set to 50Hz (48MHz / 480 divider / 2000 PWM period)
* The 2000PWM period is also changed in iron.c for power calculation.
* Program timer set to 1mS, and takes some definitions from the timings in setup.h.
* Added SPI transfer complete callback for the dma.
* The DMA transfers blocks of 128bytes (an oled row), and automatically sends the next row command. After 8 rows are * sent, the DMA disables.
* Every call to update_display triggers the DMA again.
* Added WAKE pin to interrupt-on-pin-change callback, so it clears the current iron timeout as long as the handle * moves, preventing sleep from activating. Also exits sleep or standby modes.
* Removed most iron code from callback to the iron handler.

[Core/Src/iron.c](Core/Src/iron.c):

* Moved a ton of stranded variables into a single, ordered struct.
* Changed some  vars  for better understanding, deleted and added few more for better functionality.
* Fixed sleep and standby behaviors.
* Take Current_temp store  interval from setup.h, with the option of no saving (prevents flash degrading).
* Moved all the  iron-related init stuff to IronInit.

[Core/Src/settings.c](Core/Src/settings.c):

* Restoresettings also copies PID values to CurrentPID Struct.
* Set Default contrast to max.

[Drivers/generalIO/adc_global.c](Drivers/generalIO/adc_global.c):

* Moved main.c ADC Start to a new function ADC_Start_DMA.
* Moved main.c ADC Cal to a new function ADC_Cal.
* Created  RollingTypeDef typedef, every function using an ADC channel should create its own.
* (Ex. Irontemp, VInput, Vref, ColdJunctionTemp, each has one defined)
* Added var "init" to clear buffer on init.
* Added RollingUpdate function to make the average of the ADC buffer and the rolling buffer itself if any.
* By default the rolling buffer size is set to 1(No rolling buffer average), as it will conflict with no iron * detection.
* Created ADC_to_mV function. Calcs not using floats for better performance, while extreme precision it's not needed * at all in this application.

[Drivers/generalIO/tempsensors.c](Drivers/generalIO/tempsensors.c):

* Set iron min temp to 100ºC, max to 480ºC
* Coldjunction changed name to readColdJunctionSensorTemp_C_x10,
* as now it's returning ºC_x_10 instead milli_ºC.
* We overflowed the int over 65ºC, while underflowed on negative readings.
* No need that precision anyway.
* All functions that read coldjunction now divide /10 instead /1000, 
* Changed algorithm to a table-based one, with values within -20°C to 125°C
* Error is about 0.16ºC while much faster, skipping floats.
* Changed return to int for signed temperature return.
* The other functions also take in count the sign.

[Drivers/generalIO/voltagesensors.c](Drivers/generalIO/voltagesensors.c):

* GetSupplyVoltage changed getSupplyVoltage_v_x10, it returns V_x10 now.
* Corrected final calculation taking in count the resistor divider constant.
* Added getReferenceVoltage_mv_x10

[Drivers/graphics/ugui.c](Drivers/graphics/ugui.c):

* Moved UG_GUI data here, changed to UG_GUI "gui" to "user_gui" and declared extern on ugui.h

[Drivers/graphics/ssd1306.c](Drivers/graphics/ssd1306.c):

* All the variables related to the OLED buffer, SPI and DMA work flow are declared here.
* Changed everything to work also in Soft_SPI mode.
* Added simple GPIO defs for CS, RES, DC  signals like Set_CS, Clear_CS, etc.

[Drivers/graphics/gui/debug_screen.c](Drivers/graphics/gui/debug_screen.c):

* Added value titles, better ordering.
* Moved debug_screen2 to a separate file

[Drivers/graphics/gui/debug_screen2.c](Drivers/graphics/gui/debug_screen2.c):

* Added value titles, better ordering.

[Drivers/graphics/gui/gui.c](Drivers/graphics/gui/gui.c):

* guiInit: Moved all the gui init stuff here.
* Now requires a passing of the PWM timer used.
* Added a Free-RAM-like function for debugging purposes, commented by default.
* This was due "oled_AddScreen" allocation a lot of RAM inadvertently, causing Hard Faults while the compiler said * 35% RAM usage.
* Each widget takes about 100bytes of RAM, so if a screen has 10 widgets (titles, values, buttons), sum up! A screen * could take 3KB easily!
* So I had to tweak a bit the screens, unifying text widgets into a single one and such things.

[Drivers/graphics/gui/main_screen.c](Drivers/graphics/gui/main_screen.c):

* Set NoIronADC detection value to 4000. That's about 650ºC.
* Added a nicer thermometer (Personal liking). The old is still there, commented out.
* Reordered Iron modes, so now when rotating, it goes Standby->Sleep->Set->Boost.
* SetPoint widget unselected on boot, I found it annoying having to unselect it every time to move to other widget.
* Added update_readings and Last_xxx_variables, so we don't update the readings on every screen update.
* We can refresh the screen very fast, making the gui responsive, while the readings update rate is human readable.
* Added Vsupply widget.
* Removed decimals from Ambient temperature widget to save some screen space for the Vsupply widget.
* Moved all the informative widgets (ambient temp, iron power, Vsupply) to the upper parts of the screen, while the * adjustable widgets are now on the bottom.
* Enabled NO IRON widget. When No iron condition occurs, all the adjustable widgets are disabled until the condition * dissapears.
* Tweaked iron temp position.
* All the values are now right-justified so the values are always next to their labels. Added the "justify" option in * widget typedef
* Now:	100C,  99C,   1C.
* Before:	100C, 99 C, 1C

[Drivers/graphics/gui/oled.c](Drivers/graphics/gui/oled.c):

* oled_draw: Added wait to DMA transfer, if we draw while transmitting the screen will artifact.
* We could just return without drawing, and draw at next drawing call.

[Drivers/graphics/gui/settings_screen.c](Drivers/graphics/gui/settings_screen.c):

* Changed a lot of things.
* Defined max / min values to remove data glitches/overflows/underflows.
* Changed SAVE/EXIT buttons to SAVE/BACK and increased size.
* SAVE/EXIT returns to Settings menu instead main screen.
* Power/contrast have big representation.

[Drivers/graphics/gui/screen.c](Drivers/graphics/gui/screen.c):

* Splash Screen: New big full splash screen BETA logo.

[Drivers/graphics/gui/widgets.c](Drivers/graphics/gui/widgets.c):

* Added displayWidget.justify option in typedef.
* Changed default_widgetUpdate to use this.
* Changed insertDot to add a 0 if there's nothing before the '.'
* Changed default_widgetProcessInput to make checks on the value before adding or resting to prevent over/underflow of the int.
