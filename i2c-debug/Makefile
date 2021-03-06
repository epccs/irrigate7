# Makefile 
TARGET = i2c-debug
LIBDIR = ../lib
OBJECTS = main.o \
	i2c-scan.o \
	i2c-cmd.o \
	../Uart/id.o \
	$(LIBDIR)/timers.o \
	$(LIBDIR)/uart.o \
	$(LIBDIR)/twi.o \
	$(LIBDIR)/rpu_mgr.o \
	$(LIBDIR)/adc.o \
	$(LIBDIR)/parse.o

## Chip and project-specific global definitions
MCU   =  atmega1284p
F_CPU = 16000000UL  
BAUD  =  38400UL
CPPFLAGS = -DF_CPU=$(F_CPU) -DBAUD=$(BAUD) -I. 

## Cross-compilation
CC = avr-gcc
OBJCOPY = avr-objcopy
OBJDUMP = avr-objdump
SIZE = avr-size

# FTDI's USB to serial bridge shows as /dev/ttyUSB0, 
# Uno's serial bridge (an ATmega16U2) shows as /dev/ttyACM0  (a modem,?)
# Pi Zero on chip hardware serial shows as /dev/ttyAMA0 (hardware UART on a Linux system)
detect_PORT := $(shell sh -c 'ls /dev/ttyAMA0 2>/dev/null || echo not')
ifeq ($(detect_PORT),/dev/ttyAMA0)
	BOOTLOAD_PORT = /dev/ttyAMA0
endif
detect_PORT := $(shell sh -c 'ls /dev/ttyUSB0 2>/dev/null || echo not')
ifeq ($(detect_PORT),/dev/ttyUSB0)
	BOOTLOAD_PORT = /dev/ttyUSB0
endif

## Compiler/linker options
CFLAGS = -Os -g -std=gnu99 -Wall
# CFLAGS += -funsigned-char -funsigned-bitfields 
# CFLAGS += -fpack-struct -fshort-enums 
CFLAGS += -ffunction-sections -fdata-sections 

TARGET_ARCH = -mmcu=$(MCU)

LDFLAGS = -Wl,-Map,$(TARGET).map 
LDFLAGS += -Wl,--gc-sections 

all: $(TARGET).hex $(TARGET).lst

$(TARGET): $(TARGET).hex

$(TARGET).hex: $(TARGET).elf
	$(OBJCOPY) -j .text -j .data -O ihex $< $@

## xboot wants to see the erase command -e and needs safemode fuse checking disabled -u
bootload: $(TARGET).hex
	avrdude -p $(MCU) -c avr109 -P $(BOOTLOAD_PORT) -b 115200 -e -u -U flash:w:$(TARGET).hex

$(TARGET).elf: $(OBJECTS)
	$(CC) $(LDFLAGS) $(TARGET_ARCH) $^ -o $@
	$(SIZE) -C --mcu=$(MCU) $@
	rm -f $(TARGET).o $(OBJECTS)

clean: 
	rm -f $(TARGET).hex $(TARGET).map $(TARGET).elf $(TARGET).lst
 
%.lst: %.elf
	$(OBJDUMP) -h -S $< > $@
