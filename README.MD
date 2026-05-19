# Freecut Air

Freecut Air is an open source project to replace the Cricut Expression's proprietary software stack. It combines a custom firmware for the Cricut itself (a fork of FreeExpression), a Python exporter that exports SVGs into self-contained .fcj (Freecut Job) files, and an ESP32-based wifi bridge that mounts inside the machine that runs a web UI for sending those jobs over the network to the cutter. Plug in the Cricut, open a browser, drop in a design, and cut.

Originally forked from https://github.com/thetazzbot/FreeExpression, which was forked from https://github.com/Arlet/Freecut.


## About the Cricut

> Cricut cutting machines can be had for about $50 or less on Craigslist. They contain two stepper motors, a moving solenoid powered knife holder that generates 12 volts to the solenoid using PWM. They typically contain wheels that control speed, pressure, and other settings (one of them is usually an optical encoder, the others are potentiometers), a display of sorts, and a keyboard that has a ton of fairly useless buttons with green LED backlights. Additionally, an onboard USB port and the Atmel ATMega1281 chip running at 16mhz provides all the horsepower you would need. The stepper motors are controlled by eight mosfets. So in reality, you get a fairly decent XY controller board with a PWM output for solenoid or laser control for cheap!


## PLEASE READ BEFORE FLASHING

REPROGRAMMING THE CPU WILL PERMANENTLY ERASE YOUR ORIGINAL FIRMWARE. THERE IS NO WAY TO GET THE ORIGINAL FIRMWARE BACK OTHER THAN BUYING ANOTHER MACHINE.** 

- Original firmware cannot be copied from another machine as it has been
locked and cannot be read from the CPU

- Original firmware cannot be restored through any PC application used for normal firmware upgrades

- This firmware has only been tested on the machine that I have. It may not work on your machine. It may cause permanent damage, and it
will certainly void your warranty

- This firmware cannot read your original cartridges ever


## Flashing


## Expression Firmware

### Command Language

Currently supports HPGL (HP Graphic Language). Cutting speed and pressure are not taken from the language input, only from keyboard and associated dials.


### Keypad Key Functions

- On/Off : Powers the machine and executes carriage homing sequence.
- Move buttons : Moves the carriage and the media freely.
- Home (ResetAll) : Re-homes the carriage, media is assumed to be unloaded.
- Load Paper		 : Pulls in media or cutting mat about 1/2".
- Unload Paper	 : Moves the mat/media back to the loading point. 
- X,Y Offset (BackSpace): Sets the current X and Y position as location 0,0 for following cuts
- Cutter Down (Numeral 1) : Drops the cutter down. This only works when the media is loaded. Useful  for measuring actual cutting force with a scale.
- Cutter Up (Numeral 2)  : Moves the cutter away from the media.
- Cutting Speed	(Xtra2) : In conjunction with the +/keys the cutting speed can be chosen in 9 steps.
- Cutting Pressure (Xtra1) : In conjunction with the +/keys the cutting pressure can be adjusted in 9 steps.
- Letter H : HPGL language selection, (default)
- Letter G: G-code language selection, see file g-code.c for supported g-code tokens
- Stop: Aborts the currently cached cutting operations, but new data arriving from the PC will still trigger subsequent motion.


### Dials

- Dial Speed: 	Also adjusts the cutting speed and is used to read the cutting speed after power up. Only the mid range of the speed choices can be selected, use the +/keys to get all the way to the end.
- Dial Pressure: Also adjusts the cutting pressure and is used to read the initial pressure on power-up. Only the mid range of the pressure choices can be selected, use the +/keys for the full range.
- Dial Size:  Not used. The machine can cut up to 80"x12"wide.


### Media Lock

After the machine moves the media/carriage, there is a time of about 1 minute (depending on speed setting) where the motors stay engaged and you can not move the media/carriage by hand. After this timeout the motors go into standby and you can move carriage and media by hand. This timeout is useful so that one can cut the same shape multiple times and not loose registration - for thick material multi-cut.


### Legal

*Cricut is a trademark of PROVO CRAFT and this firmware is **not** a PROVO CRAFT product. This firmware was developed completely independently, without any documentation of PROVO CRAFT products, and without any and all original firmware. If you have problems with this firmware, do not consult PROVO CRAFT.