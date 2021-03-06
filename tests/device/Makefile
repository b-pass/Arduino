SHELL := /bin/bash
V ?= 0
TEST_LIST ?= $(wildcard test_*/*.ino)
ESP8266_CORE_PATH ?= ../..
BUILD_DIR ?= $(PWD)/.build
HARDWARE_DIR ?= $(PWD)/.hardware
ESPTOOL ?= $(ESP8266_CORE_PATH)/tools/esptool/esptool
UPLOAD_PORT ?= $(shell ls /dev/tty* | grep -m 1 -i USB)
UPLOAD_BAUD ?= 921600
UPLOAD_BOARD ?= nodemcu
BS_DIR ?= libraries/BSTest
DEBUG_LEVEL ?= DebugLevel=None____
FQBN ?= esp8266com:esp8266:generic:CpuFrequency=80,FlashFreq=40,FlashMode=DIO,UploadSpeed=115200,FlashSize=4M1M,ResetMethod=none,Debug=Serial,$(DEBUG_LEVEL)
BUILD_TOOL = $(ARDUINO_IDE_PATH)/arduino-builder
TEST_CONFIG = libraries/test_config/test_config.h

ifeq ("$(UPLOAD_PORT)","")
$(error "Failed to detect upload port, please export UPLOAD_PORT manually")
endif

ifeq ("$(ARDUINO_IDE_PATH)","")
$(error "Please export ARDUINO_IDE_PATH")
endif

ifneq ("$(V)","1")
	SILENT = @
else
	BUILDER_DEBUG_FLAG = -verbose
	RUNNER_DEBUG_FLAG = -d
	UPLOAD_VERBOSE_FLAG = -v
endif


all: count tests

count:
	@echo Running $(words $(TEST_LIST)) tests

tests: $(BUILD_DIR) $(HARDWARE_DIR) virtualenv $(TEST_CONFIG) $(TEST_LIST)

$(TEST_LIST): LOCAL_BUILD_DIR=$(BUILD_DIR)/$(notdir $@)

$(TEST_LIST):
	$(SILENT)mkdir -p $(LOCAL_BUILD_DIR)
ifneq ("$(NO_BUILD)","1")
	@echo Compiling $(notdir $@)
	$(SILENT)$(BUILD_TOOL) -compile -logger=human \
		-libraries "$(PWD)/libraries" \
		-core-api-version="10608" \
		-warnings=none \
		$(BUILDER_DEBUG_FLAG) \
		-build-path $(LOCAL_BUILD_DIR) \
		-tools $(ARDUINO_IDE_PATH)/tools-builder \
		-hardware $(HARDWARE_DIR)\
		-hardware $(ARDUINO_IDE_PATH)/hardware \
		-fqbn=$(FQBN) \
		$@
endif
ifneq ("$(NO_UPLOAD)","1")
	$(SILENT)$(ESPTOOL) $(UPLOAD_VERBOSE_FLAG)  \
		-cp $(UPLOAD_PORT) \
		-cb $(UPLOAD_BAUD) \
		-cd $(UPLOAD_BOARD) \
		-cf $(LOCAL_BUILD_DIR)/$(notdir $@).bin
endif
ifneq ("$(NO_RUN)","1")
	@echo Running tests
	$(SILENT)$(ESPTOOL) $(UPLOAD_VERBOSE_FLAG) -cp $(UPLOAD_PORT) -cd $(UPLOAD_BOARD) -cr
	@source $(BS_DIR)/virtualenv/bin/activate && \
		python $(BS_DIR)/runner.py \
			$(RUNNER_DEBUG_FLAG) \
			-p $(UPLOAD_PORT) \
			-n $(basename $(notdir $@)) \
			-o $(LOCAL_BUILD_DIR)/test_result.xml
endif

$(BUILD_DIR):
	mkdir -p $(BUILD_DIR)

$(HARDWARE_DIR):
	mkdir -p $(HARDWARE_DIR)/esp8266com
	cd $(HARDWARE_DIR)/esp8266com && ln -s $(realpath $(ESP8266_CORE_PATH)) esp8266

virtualenv:
	make -C $(BS_DIR) virtualenv

clean:
	rm -rf $(BUILD_DIR)
	rm -rf $(HARDWARE_DIR)

$(TEST_CONFIG):
	@echo "******    "
	@echo "******    libraries/test_config/test_config.h does not exist"
	@echo "******    Create one from libraries/test_config/test_config.h.template"
	@echo "******    "
	false

.PHONY: tests all count venv $(BUILD_DIR) $(TEST_LIST) 
