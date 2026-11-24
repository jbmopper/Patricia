.PHONY: build datagen aarch64 cross-aarch64 cross-aarch64-clang macos macos-arm64 macos-x86_64 macos-universal

EXE := patricia
SOURCES := src/patricia.cpp src/fathom/src/tbprobe.c

CXX := clang++
CXXFLAGS := -O3 -march=native -std=c++20 -ffast-math
LINKER :=

ifeq ($(OS), Windows_NT)
	DETECTED_OS := Windows
	SUFFIX := .exe
else
	DETECTED_OS := $(shell uname -s)
	LINKER := -lm
	CXXFLAGS += -pthread
	SUFFIX :=
endif

OUT := $(EXE)$(SUFFIX)

# --- Native build ----------------------------------------------------------
build: $(SOURCES)
	$(CXX) $^ $(CXXFLAGS) -o $(OUT) $(LINKER)

default: build

datagen: datagen/datagen.cpp
	$(CXX) $^ $(CXXFLAGS) -o data $(LINKER)

# --- Native AArch64 build (on ARM host) -----------------------------------
aarch64: CXX := clang++
aarch64: CXXFLAGS := -O3 -std=c++20 -ffast-math -mcpu=native -pthread
ifneq ($(DETECTED_OS), Darwin)
aarch64: CXXFLAGS += -static-libstdc++ -static-libgcc
endif
aarch64: OUT := $(EXE)_aarch64$(SUFFIX)
aarch64: $(SOURCES)
	$(CXX) $^ $(CXXFLAGS) -o $(OUT) $(LINKER)

# --- Cross-compile for AArch64 (fully static) -----------------------------
# Usage: make cross-aarch64 CROSS=aarch64-linux-gnu
cross-aarch64: CROSS ?= aarch64-linux-gnu
cross-aarch64: CXX := $(CROSS)-g++
cross-aarch64: CXXFLAGS := -O3 -std=c++20 -ffast-math -march=armv8-a -pthread -static -static-libstdc++ -static-libgcc
cross-aarch64: OUT := $(EXE)_aarch64$(SUFFIX)
cross-aarch64: $(SOURCES)
	$(CXX) $(CXXFLAGS) $^ -o $(OUT)

# --- Cross-compile for AArch64 using clang (requires proper sysroot) -----
cross-aarch64-clang: CXX := clang++
cross-aarch64-clang: CXXFLAGS := --target=aarch64-linux-gnu -O3 -std=c++20 -ffast-math -march=armv8-a -pthread -static
cross-aarch64-clang: OUT := $(EXE)_aarch64$(SUFFIX)
cross-aarch64-clang: $(SOURCES)
	$(CXX) $(CXXFLAGS) $^ -o $(OUT)

# --- macOS ----------------------------------------------------------------
# Apple clang cannot link libstdc++/libgcc statically, and libc++ ships with
# macOS, so these binaries only depend on system libraries.
#   make macos            portable Apple Silicon binary (NEON, any M-series)
#   make macos-universal  arm64 (NEON) + x86_64 (AVX2/BMI2) fat binary
MACOS_MIN := 11.0
MACOS_FLAGS := -O3 -std=c++20 -ffast-math -pthread -mmacosx-version-min=$(MACOS_MIN) \
               -DINCBIN_SILENCE_BITCODE_WARNING

macos: macos-arm64

macos-arm64: $(SOURCES)
	clang++ $^ -arch arm64 -mcpu=apple-m1 $(MACOS_FLAGS) -o $(EXE)-macos-arm64

macos-x86_64: $(SOURCES)
	clang++ $^ -arch x86_64 -march=haswell -mbmi2 $(MACOS_FLAGS) -o $(EXE)-macos-x86_64

macos-universal: macos-arm64 macos-x86_64
	lipo -create -output $(EXE)-macos-universal $(EXE)-macos-arm64 $(EXE)-macos-x86_64
