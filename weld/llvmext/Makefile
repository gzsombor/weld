OS = $(shell uname -s)
ifeq (, $(shell which llvm-config))
  $(info No llvm-config on the path, try to use llvmenv)
  LLVM_PREFIX = $(shell llvmenv prefix)
  $(info LLVM is installed into $(LLVM_PREFIX))
  LLVM_CONFIG = $(LLVM_PREFIX)/bin/llvm-config
  LLVM_VERSION = $(shell $(LLVM_CONFIG) --version | cut -d . -f 1,2)
else
  LLVM_CONFIG = llvm-config
  LLVM_VERSION = $(shell llvm-config --version | cut -d . -f 1,2)
endif

CFLAGS = -O3 -fno-use-cxa-atexit
CFLAGS += $(shell $(LLVM_CONFIG) --cppflags) $(shell $(LLVM_CONFIG) --cxxflags)
LIBDIR = $(shell $(LLVM_CONFIG) --libdir)
LLVMLIBS = $(shell $(LLVM_CONFIG) --system-libs) $(shell $(LLVM_CONFIG) --libs)
LIB = libllvmext.a

ifeq ($(OS), Darwin)
  CLANG ?= clang++
else ifeq ($(OS), Linux)
  ifeq (, $(shell which clang++-$(LLVM_VERSION)))
    CLANG ?= $(LLVM_PREFIX)/bin/clang++
  else
    CLANG ?= clang++-$(LLVM_VERSION)
  endif
else
  $(error Unsupported platform: $(OS))
endif

.PHONY: all test

all: $(LIB)

test: $(LIB) test.cpp
	$(CLANG) $(CFLAGS) -L$(LIBDIR) $(LLVMLIBS) $(LIB) test.cpp -o $(OUT_DIR)/test

llvmext.o: llvmext.cpp
	$(CLANG) $(CFLAGS) -c $< -o $(OUT_DIR)/$@

$(LIB): llvmext.o
	ar rcs $(OUT_DIR)/$@ $(OUT_DIR)/$^

clean:
	rm -f $(OUT_DIR)/*.o $(OUT_DIR)/$(LIB)
