OS=$(shell uname -s)
ifeq (, $(shell which llvm-config))
  $(info No llvm-config on the path, try to use llvmenv)
  LLVM_PREFIX = $(shell llvmenv prefix)
  LLVM_CONFIG = $(LLVM_PREFIX)/bin/llvm-config
  LLVM_VERSION = $(shell $(LLVM_CONFIG) --version | cut -d . -f 1,2)
else
  LLVM_CONFIG = llvm-config
  LLVM_VERSION = $(shell llvm-config --version | cut -d . -f 1,2)
endif

PYTHON_HEADER_INCLUDE = $(shell python-config --includes)
NUMPY_HEADER_INCLUDE = -I$(shell python -c "import numpy; print numpy.get_include()" 2>/dev/null)

ifndef EXEC
  EXEC=python
endif

PYTHON_LDFLAGS=-L$(shell $(EXEC) -c "from distutils import sysconfig; print sysconfig.get_config_var('LIBDIR')" 2>/dev/null) $(shell python-config --libs)

ifeq (${NUMPY_HEADER_INCLUDE}, -I)
  $(error Error: NumPy installation not found)
endif

ifeq (${OS}, Darwin)
  # OS X
  CLANG ?= clang
  CLANGPP ?= clang++
  FLAGS=-march=native -O3 -flto
  DFLAGS=-dynamiclib
  DYLIB_SUFFIX=.dylib
  PYTHON_INCLUDES=${PYTHON_HEADER_INCLUDE} ${NUMPY_HEADER_INCLUDE}
else ifeq (${OS}, Linux)
  # Linux
  ifeq (, $(shell which clang-$(LLVM_VERSION)))
    CLANG ?= $(LLVM_PREFIX)/bin/clang
  else
    CLANG ?= clang-${LLVM_VERSION}
  endif
  ifeq (, $(shell which clang++-$(LLVM_VERSION)))
    CLANGPP ?= $(LLVM_PREFIX)/bin/clang++
  else
    CLANGPP ?= clang++-${LLVM_VERSION}
  endif
  FLAGS=-mavx2 -fuse-ld=gold -flto
  DFLAGS=-shared -fPIC
  DYLIB_SUFFIX=.so
  PYTHON_INCLUDES=${PYTHON_HEADER_INCLUDE} ${NUMPY_HEADER_INCLUDE}
else
  $(error Unsupported platform: ${OS})
endif

convertor:
	${CLANG} ${DFLAGS} -w -march=native $(PYTHON_LDFLAGS) numpy_weld_convertor.cpp -o numpy_weld_convertor${DYLIB_SUFFIX}  ${PYTHON_INCLUDES}

clean:
	rm -rf run numpy_weld_convertor${DYLIB_SUFFIX} *.pyc
