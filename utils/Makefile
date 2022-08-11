ifdef LONG
INTT = -DLONG
endif

ifdef EDGELONG
INTE = -DEDGELONG
endif

#compilers
ifdef CILK
PCC = g++
PCFLAGS = -std=c++14 -fcilkplus -lcilkrts -O3 -DCILK $(INTT) $(INTE)
PLFLAGS = -fcilkplus -lcilkrts

else ifdef MKLROOT
PCC = icpc
PCFLAGS = -std=c++14 -O3 -DCILKP $(INTT) $(INTE)

else ifdef OPENMP
PCC = g++
PCFLAGS = -std=c++14 -fopenmp -march=native -O3 -DOPENMP $(INTT) $(INTE)

else
PCC = g++
PCFLAGS = -std=c++14 -O3 $(INTT) $(INTE)
endif

TOP := $(shell git rev-parse --show-toplevel)/../..
INSTALL_DIR ?= $(TOP)/install
LOG_DIR ?= $(TOP)/outputs

EXT ?= riscv
ifeq ($(EXT),x86)
TOOLCHAIN_PREFIX=
EMULATOR =
RISCV_FLAG =
else
TOOLCHAIN_PREFIX=riscv64-unknown-linux-gnu-
EMULATOR = $(INSTALL_DIR)/bin/qemu-riscv64 -L $(INSTALL_DIR)/sysroot
RISCV_FLAG = -DRISCV
endif

COMMON = utils.h parseCommandLine.h parallel.h quickSort.h blockRadixSort.h transpose.h
LOCAL_COMMON = graphIO.h
GENERATORS = rMatGraph gridGraph randLocalGraph SNAPtoAdj wghSNAPtoAdj adjGraphAddWeights adjToBinary communityToHyperAdj hyperAdjToBinary adjHypergraphAddWeights randHypergraph KONECTtoHyperAdj KONECTtoClique communityToClique communityToMESH KONECTtoMESH

.PHONY: all clean
all: $(GENERATORS)

$(COMMON):
	ln -s ../ligra/$@ .

% : %.C $(COMMON) $(LOCAL_COMMON)
	$(TOOLCHAIN_PREFIX)$(PCC) $(RISCV_FLAG) $(PCFLAGS) -o $@.$(EXT) $<

clean :
	rm -f *.o $(GENERATORS:=.$(EXT))

cleansrc :
	rm -f *.o $(GENERATORS:=.riscv) $(GENERATORS:=.x86)
	rm -f $(COMMON)
