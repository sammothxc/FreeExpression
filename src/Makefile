# =============================================================================
# Makefile for FreeExpression — Cricut Expression open-source firmware
# =============================================================================
#
# Derived from the Atmel Studio 7 project files (FreeExpression.cproj and
# m2u8/m2u8.cproj). Targets GNU make + avr-gcc on Linux/macOS.
#
# Build layout:
#   m2u8/  — u8glib graphics library, built as a static library (libm2u8.a)
#   .      — firmware sources, linked against libm2u8.a
#
# Usage:
#   make                  build firmware (default: Debug, matches working .hex)
#   make CONFIG=Release   build release variant
#   make clean            remove all build artefacts
#   make size             show flash/SRAM usage of the built ELF
#   make check            verify ISP wiring (read chip signature, no flash)
#   make prog             flash freecut.hex to the chip via avrdude
#   make read_fuses       read current fuse bytes
#
# Programmer config — override on command line if needed, e.g.:
#   make prog AVRDUDE_PROGRAMMER=usbasp AVRDUDE_PORT=usb
# =============================================================================

# ---------- Target chip & clock ----------------------------------------------
MCU          := atmega1281
F_CPU        := 16000000UL
TARGET       := freecut

# ---------- Build configuration: Debug (default) or Release ------------------
CONFIG       ?= Debug

# ---------- Toolchain --------------------------------------------------------
CC           := avr-gcc
AR           := avr-ar
OBJCOPY      := avr-objcopy
OBJDUMP      := avr-objdump
SIZE         := avr-size

# ---------- Source lists (from FreeExpression.cproj) -------------------------
SRC := \
    cli.c           \
    dial.c          \
    display.c       \
    display_lcd.c   \
    display_oled.c  \
    flash.c         \
    g_code.c        \
    hpgl.c          \
    keypad.c        \
    main.c          \
    scale.c         \
    serial.c        \
    shvars.c        \
    spi.c           \
    stepper.c       \
    timer.c         \
    usb.c

# ---------- u8glib sources (from m2u8/m2u8.cproj) ----------------------------
# Built as a separate static library: m2u8/libm2u8.a
U8G_SRC := \
    u8g_bitmap.c                    \
    u8g_circle.c                    \
    u8g_clip.c                      \
    u8g_com_api.c                   \
    u8g_com_api_16gr.c              \
    u8g_com_atmega_sw_spi.c         \
    u8g_com_i2c.c                   \
    u8g_com_io.c                    \
    u8g_com_null.c                  \
    u8g_cursor.c                    \
    u8g_delay.c                     \
    u8g_dev_null.c                  \
    u8g_dev_ssd1325_nhd27oled_bw_new.c \
    u8g_dev_ssd1325_nhd27oled_gr_new.c \
    u8g_ellipse.c                   \
    u8g_font.c                      \
    u8g_font_data.c                 \
    u8g_line.c                      \
    u8g_ll_api.c                    \
    u8g_page.c                      \
    u8g_pb.c                        \
    u8g_pb14v1.c                    \
    u8g_pb16h1.c                    \
    u8g_pb16h2.c                    \
    u8g_pb16v1.c                    \
    u8g_pb16v2.c                    \
    u8g_pb32h1.c                    \
    u8g_pb8h1.c                     \
    u8g_pb8h1f.c                    \
    u8g_pb8h2.c                     \
    u8g_pb8h8.c                     \
    u8g_pb8v1.c                     \
    u8g_pb8v2.c                     \
    u8g_pbxh16.c                    \
    u8g_pbxh24.c                    \
    u8g_polygon.c                   \
    u8g_rect.c                      \
    u8g_rot.c                       \
    u8g_scale.c                     \
    u8g_state.c                     \
    u8g_u16toa.c                    \
    u8g_u8toa.c                     \
    u8g_virtual_screen.c

# Prefix u8g sources with their directory
U8G_DIR      := m2u8
U8G_SRC_FULL := $(addprefix $(U8G_DIR)/,$(U8G_SRC))
U8G_OBJ      := $(U8G_SRC_FULL:.c=.o)
U8G_LIB      := $(U8G_DIR)/libm2u8.a

# Firmware objects
OBJ := $(SRC:.c=.o)

# ---------- Common compiler flags --------------------------------------------
# These match Atmel Studio's defaults for AVR/C projects.
COMMON_CFLAGS := \
    -mmcu=$(MCU)              \
    -DF_CPU=$(F_CPU)          \
    -funsigned-char           \
    -funsigned-bitfields      \
    -fpack-struct             \
    -fshort-enums             \
    -fno-strict-aliasing      \
    -mrelax                   \
    -std=gnu99                \
    -Wall                     \
    -Wstrict-prototypes       \
    -Wmissing-prototypes      \
    -Werror-implicit-function-declaration \
    -Wpointer-arith           \
    -I.                       \
    -I$(U8G_DIR)

# ---------- Config-specific flags --------------------------------------------
# Debug: matches the working Atmel Studio build (DEBUG defined, -g2, no gc-sections)
# Release: smaller, NDEBUG defined
ifeq ($(CONFIG),Debug)
    CFLAGS  := $(COMMON_CFLAGS) -Os -g2 -DDEBUG -DMACHINE_EXPRESSION
    LDFLAGS := -mmcu=$(MCU) -mrelax \
               -Wl,-u,vfprintf -Wl,-u,vfscanf \
               -Wl,-Map=$(TARGET).map,--cref \
               -Wl,--start-group -lprintf_flt -lscanf_flt -lm -L$(U8G_DIR) -lm2u8 -Wl,--end-group
else ifeq ($(CONFIG),Release)
    CFLAGS  := $(COMMON_CFLAGS) -Os -DNDEBUG -DMACHINE_EXPRESSION \
               -ffunction-sections -fdata-sections
    LDFLAGS := -mmcu=$(MCU) -mrelax \
               -Wl,--gc-sections \
               -Wl,-Map=$(TARGET).map,--cref \
               -Wl,--start-group -lm -L$(U8G_DIR) -lm2u8 -Wl,--end-group
else
    $(error CONFIG must be Debug or Release, not '$(CONFIG)')
endif

# u8glib library uses simpler flags — it's a third-party lib, don't apply
# the strict prototype/pointer-arith warnings to it. Just size and the basics.
U8G_CFLAGS := \
    -mmcu=$(MCU)              \
    -DF_CPU=$(F_CPU)          \
    -funsigned-char           \
    -funsigned-bitfields      \
    -fpack-struct             \
    -fshort-enums             \
    -mrelax                   \
    -Os                       \
    -g2                       \
    -Wall                     \
    -I$(U8G_DIR)

# =============================================================================
# Build rules
# =============================================================================

.PHONY: all clean size check prog read_fuses

all: $(TARGET).hex $(TARGET).eep $(TARGET).lss size

# Link firmware against m2u8 static library
$(TARGET).elf: $(OBJ) $(U8G_LIB)
	$(CC) $(OBJ) -o $@ $(LDFLAGS)

# Compile firmware sources
%.o: %.c
	$(CC) -c $(CFLAGS) $< -o $@

# Build the m2u8 static library
$(U8G_LIB): $(U8G_OBJ)
	$(AR) rcs $@ $^

# Compile u8glib sources (uses U8G_CFLAGS, not the firmware CFLAGS)
$(U8G_DIR)/%.o: $(U8G_DIR)/%.c
	$(CC) -c $(U8G_CFLAGS) $< -o $@

# Generate the .hex file (Intel HEX format) — this is what we flash
$(TARGET).hex: $(TARGET).elf
	$(OBJCOPY) -O ihex -R .eeprom -R .fuse -R .lock -R .signature $< $@

# Generate the .eep file for EEPROM contents (if any)
$(TARGET).eep: $(TARGET).elf
	-$(OBJCOPY) -O ihex -j .eeprom \
	    --set-section-flags=.eeprom="alloc,load" \
	    --change-section-lma .eeprom=0 --no-change-warnings $< $@ 2>/dev/null || true

# Generate disassembly listing — invaluable for debugging
$(TARGET).lss: $(TARGET).elf
	$(OBJDUMP) -h -S $< > $@

size: $(TARGET).elf
	@echo
	@echo "=== Firmware size ==="
	@$(SIZE) --format=avr --mcu=$(MCU) $(TARGET).elf
	@echo

clean:
	rm -f $(OBJ) $(TARGET).elf $(TARGET).hex $(TARGET).eep
	rm -f $(TARGET).lss $(TARGET).map
	rm -f $(U8G_OBJ) $(U8G_LIB)
	rm -f *.d $(U8G_DIR)/*.d

# =============================================================================
# Programming (avrdude)
# =============================================================================

# Default to Arduino-as-ISP. Override on the command line:
#   make prog AVRDUDE_PROGRAMMER=usbasp AVRDUDE_PORT=usb
AVRDUDE_MCU         := m1281
AVRDUDE_PROGRAMMER  ?= avrisp
AVRDUDE_PORT        ?= /dev/ttyUSB0
AVRDUDE_BAUD        ?= 19200
AVRDUDE := avrdude -p $(AVRDUDE_MCU) -c $(AVRDUDE_PROGRAMMER) -P $(AVRDUDE_PORT) -b $(AVRDUDE_BAUD)

# Sanity check the ISP wiring — reads chip signature, doesn't write anything
check:
	$(AVRDUDE) -v

# Read fuses
read_fuses:
	$(AVRDUDE) -U lfuse:r:-:h -U hfuse:r:-:h -U efuse:r:-:h

# Flash the firmware
prog: $(TARGET).hex
	$(AVRDUDE) -U flash:w:$(TARGET).hex:i

# Auto-generated header dependency tracking
-include $(OBJ:.o=.d)
-include $(U8G_OBJ:.o=.d)

# Add -MMD -MP to emit .d files alongside .o files. This lets make know
# which .h changes should trigger which .c rebuilds.
CFLAGS      += -MMD -MP
U8G_CFLAGS  += -MMD -MP