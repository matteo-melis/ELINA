include ../Makefile.config

PREFIX = /usr/local
HOMEBREW_PREFIX = /opt/homebrew

FCONV_INCLUDE_FILES = fconv.h utils.h fp_mat.h mpq.h dynamic_bitset.h octahedron.h asrt.h S_curve.h S_curve2.h pdd.h split_in_quadrants.h sparse_cover.h octahedron_resources.h relaxation.h decomposition.h quadrants.h
FCONV_C_FILES = fconv.cpp utils.cpp fp_mat.cpp mpq.cpp dynamic_bitset.cpp octahedron.cpp S_curve.cpp S_curve2.cpp pdd.cpp split_in_quadrants.cpp sparse_cover.cpp relaxation.cpp decomposition.cpp quadrants.cpp
FCONV_OBJS = $(subst .cpp,.o,$(FCONV_C_FILES))

# OS
UNAME_S := $(shell uname -s)

CXXFLAGS += -std=c++14 -O3 -march=native -fPIC -Wall -Wextra -DGMPRATIONAL
CXXFLAGS += -I$(HOMEBREW_PREFIX)/include -I$(HOMEBREW_PREFIX)/include/cdd

ifeq ($(UNAME_S),Darwin)
    # macOS Specifics
    SDKROOT := $(shell xcrun --show-sdk-path)
    SHARED_LIB = libfconv.dylib
    
    # We create the dylib, but we MUST create a .so symlink for Python later
    SHARED_FLAGS = -dynamiclib -install_name $(PREFIX)/lib/$(SHARED_LIB)
    
    # Linker locations
    LDFLAGS += -L/usr/local/lib -L$(HOMEBREW_PREFIX)/lib
    
    # SDK and Stdlib linkage
    CXXFLAGS += -stdlib=libc++ -isysroot $(SDKROOT) -I$(SDKROOT)/usr/include/c++/v1
    LDFLAGS += -stdlib=libc++ -isysroot $(SDKROOT)
else
    SHARED_LIB = libfconv.so
    SHARED_FLAGS = -shared -fPIC
endif

LIBS = -lgmp -lmpfr -lcdd
all: $(SHARED_LIB) libfconv.so

$(SHARED_LIB): $(FCONV_OBJS)
	$(CXX) $(SHARED_FLAGS) -o $@ $^ $(LDFLAGS) $(LIBS)

# Create a .so symlink so Python can find it (macOS only)
libfconv.so: $(SHARED_LIB)
ifeq ($(UNAME_S),Darwin)
	ln -sf $(SHARED_LIB) libfconv.so
endif

%.o: %.cpp $(FCONV_INCLUDE_FILES)
	$(CXX) -c $(CXXFLAGS) -o $@ $<

install:
	mkdir -p $(PREFIX)/include/fconv
	mkdir -p $(PREFIX)/lib
	cp $(FCONV_INCLUDE_FILES) $(PREFIX)/include/fconv
	cp $(SHARED_LIB) $(PREFIX)/lib
ifeq ($(UNAME_S),Darwin)
	# Install the symlink too
	ln -sf $(PREFIX)/lib/$(SHARED_LIB) $(PREFIX)/lib/libfconv.so
endif

clean:
	rm -f *.o $(SHARED_LIB) libfconv.so libfconv.dylib

.PHONY: all install clean