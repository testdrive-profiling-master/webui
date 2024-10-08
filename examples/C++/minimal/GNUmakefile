# WebUI C++ Example

# == 1. VARIABLES =============================================================

MAKEFILE_PATH := $(abspath $(lastword $(MAKEFILE_LIST)))
PROJECT_DIR := $(dir $(MAKEFILE_PATH))/../../../
LIB_DIR := $(PROJECT_DIR)/dist
INCLUDE_DIR := $(PROJECT_DIR)/include
WEBUI_LIB_NAME = webui-2
ifeq ($(WEBUI_USE_TLS), 1)
WEBUI_LIB_NAME = webui-2-secure
endif

# ARGS
# Set a compiler when running on Linux via `make CC=g++` / `make CC=clang`
CXX = g++
# Build the WebUI library if running via `make BUILD_LIB=true`
BUILD_LIB ?=

# BUILD FLAGS
STATIC_BUILD_FLAGS = -std=c++17 -lstdc++ main.cpp -I"$(INCLUDE_DIR)" -L"$(LIB_DIR)"
DYN_BUILD_FLAGS = -std=c++17 -lstdc++ main.cpp -I"$(INCLUDE_DIR)" -L"$(LIB_DIR)"

# Platform conditions
ifeq ($(OS),Windows_NT)
	# Windows
	PLATFORM := windows
	SHELL := CMD
	STATIC_BUILD_FLAGS += -l$(WEBUI_LIB_NAME)-static -lws2_32 -Wall -Wl,-subsystem=console -luser32 -lOle32 -ldwmapi -lgdi32 -static
	DYN_BUILD_FLAGS += "$(LIB_DIR)/$(WEBUI_LIB_NAME).dll" -lws2_32 -Wall -Wl,-subsystem=console -luser32 -lOle32 -ldwmapi -lgdi32
	STATIC_OUT := main.exe
	DYN_OUT := main-dyn.exe
	LWS2_OPT := -lws2_32
	STRIP_OPT := --strip-all
else
	STATIC_BUILD_FLAGS += -lpthread -lm -l$(WEBUI_LIB_NAME)-static
	DYN_BUILD_FLAGS += -lpthread -lm
	STATIC_OUT := main
	DYN_OUT := main-dyn
	ifeq ($(shell uname),Darwin)
		# MacOS
		PLATFORM := macos
		DYN_BUILD_FLAGS += "$(LIB_DIR)/$(WEBUI_LIB_NAME).dylib"
		CXX = clang
	else
		# Linux
		PLATFORM := linux
		STATIC_BUILD_FLAGS += -ldl
		DYN_BUILD_FLAGS += -ldl "$(LIB_DIR)/$(WEBUI_LIB_NAME).so"
		STRIP_OPT := --strip-all
		ifeq ($(CXX),clang)
			LLVM_OPT := llvm-
		endif
	endif
endif

# == 2.TARGETS ================================================================

all: release

debug: LIB_DIR := $(LIB_DIR)/debug
debug: --validate-args
ifeq ($(BUILD_LIB),true)
	@cd "$(PROJECT_DIR)" && $(MAKE) debug
endif
#	Static with Debug info
ifneq ($(WEBUI_USE_TLS), 1)
	@echo "Build C Example ($(CXX) debug static)..."
	@$(CXX) -g $(STATIC_BUILD_FLAGS) $(LWS2_OPT) -o $(STATIC_OUT)
endif
#	Dynamic with Debug info
	@echo "Build C Example ($(CXX) debug dynamic)..."
	@$(CXX) -g $(DYN_BUILD_FLAGS) $(LWS2_OPT) -o $(DYN_OUT)
#	Clean
ifeq ($(PLATFORM),windows)
	@- del *.o >nul 2>&1
else
	@- rm -f *.o
	@- rm -rf *.dSYM # macOS
endif
	@echo "Done."

release: --validate-args
ifeq ($(BUILD_LIB),true)
	@cd "$(PROJECT_DIR)" && $(MAKE)
endif
#	Static Release
ifneq ($(WEBUI_USE_TLS), 1)
	@echo "Build C Example ($(CXX) release static)..."
	@$(CXX) -Os $(STATIC_BUILD_FLAGS) $(LWS2_OPT) -o $(STATIC_OUT)
	@$(LLVM_OPT)strip $(STRIP_OPT) $(STATIC_OUT)
endif
#	Dynamic Release
	@echo "Build C Example ($(CXX) release dynamic)..."
	@$(CXX) $(DYN_BUILD_FLAGS) $(LWS2_OPT) -o $(DYN_OUT)
	@$(LLVM_OPT)strip $(STRIP_OPT) $(DYN_OUT)
#	Clean
ifeq ($(PLATFORM),windows)
	@- del *.o >nul 2>&1
else
	@- rm -f *.o
	@- rm -rf *.dSYM # macOS
endif
	@echo "Done."

clean: --clean-$(PLATFORM)

# INTERNAL TARGETS

--validate-args:
ifneq ($(filter $(CXX),g++ clang),$(CXX))
	$(error Invalid compiler specified: `$(CXX)`)
endif

--clean-linux: --clean-unix

--clean-macos: --clean-unix

--clean-unix:
	- rm -f *.o
	- rm -f *.a
	- rm -f *.so
	- rm -f *.dylib
	- rm -rf *.dSYM

--clean-windows:
	- del *.o >nul 2>&1
	- del *.dll >nul 2>&1
	- del *.a >nul 2>&1
	- del *.exe >nul 2>&1
