ifdef LONG
INTT = -DLONG
endif

ifdef EDGELONG
INTE = -DEDGELONG
endif

ifdef PD
PD = -DPD
endif

ifdef BYTE
CODE = -DBYTE
else ifdef NIBBLE
CODE = -DNIBBLE
else
CODE = -DBYTERLE
endif

ifdef LOWMEM
MEM = -DLOWMEM
endif

#compilers
ifdef CILK
PCC = g++
PCFLAGS = -std=c++14 -fcilkplus -lcilkrts -O3 -DCILK $(INTT) $(INTE) $(CODE) $(PD) $(MEM)
PLFLAGS = -fcilkplus -lcilkrts

else ifdef MKLROOT
PCC = icpc
PCFLAGS = -std=c++14 -O3 -DCILKP $(INTT) $(INTE) $(CODE) $(PD) $(MEM)

else ifdef OPENMP
PCC = g++
PCFLAGS = -std=c++14 -fopenmp -march=native -O3 -DOPENMP $(INTT) $(INTE) $(CODE) $(PD) $(MEM)

else
PCC = g++
PCFLAGS = -std=c++14 -O3 $(INTT) $(INTE) $(CODE) $(PD) $(MEM)
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
TOOLCHAIN_PREFIX=$(RV64_TOOLCHAIN_PREFIX)
EMULATOR = $(INSTALL_DIR)/bin/qemu-riscv64 -L $(INSTALL_DIR)/sysroot
RISCV_FLAG = -DRISCV
endif

compressed ?= 0

COMMON= ligra.h graph.h compressedVertex.h vertex.h utils.h IO.h parallel.h gettime.h index_map.h maybe.h sequence.h edgeMap_utils.h binary_search.h quickSort.h blockRadixSort.h transpose.h parseCommandLine.h byte.h byteRLE.h nibble.h byte-pd.h byteRLE-pd.h nibble-pd.h vertexSubset.h decoder.C hypergraphIO.h hygra.h bucket.h counting_sort.h dyn_arr.h edgeMapReduce.h histogram.h sequentialHT.h hypergraphEncoder.C hypergraphDecoder.C

ALL= hypergraphEncoder hypergraphDecoder HyperReader WeightedHyperReader HyperBFS HyperCC HyperSSSP HyperPageRank HyperBC HyperBPath HyperMIS HyperKCore HyperKCore-Efficient

TOOLS= hypergraphEncoder hypergraphDecoder HyperReader WeightedHyperReader

BENCHMARKS= HyperBFS HyperCC HyperSSSP HyperPageRank HyperBC HyperBPath HyperMIS HyperKCore HyperKCore-Efficient

all: $(ALL)

% : %.C $(COMMON)
	$(TOOLCHAIN_PREFIX)$(PCC) $(RISCV_FLAG) $(PCFLAGS) -o $@.$(EXT) $<

$(COMMON):
	ln -s ../../ligra/$@ .

run_all: $(addprefix run_,$(ALL))
	: run_all completed
	
run_tools: $(addprefix run_,$(TOOLS))
	echo "run_tools completed"

run_benchmarks: $(addprefix run_,$(BENCHMARKS))
	echo "run_benchmarks completed"

run_hypergraphEncoder:
	$(EMULATOR) ./hypergraphEncoder.$(EXT) -s ../../inputs/randHypergraph ../../inputs/randHypergraph.compressed \
	2>&1 | tee -i $(LOG_DIR)/hygra-$(EXT)-hypergraphEncoder.log
	echo -e "\nWeighted:" 2>&1 | tee -ia $(LOG_DIR)/hygra-$(EXT)-hypergraphEncoder.log
	$(EMULATOR) ./hypergraphEncoder.$(EXT) -s -w ../../inputs/randHypergraph_weighted ../../inputs/randHypergraph_weighted.compressed \
	2>&1 | tee -ia $(LOG_DIR)/hygra-$(EXT)-hypergraphEncoder.log


run_hypergraphDecoder:
	$(EMULATOR) ./hypergraphDecoder.$(EXT) -s ../../inputs/randHypergraph.compressed ../../inputs/randHypergraph_decoded \
	2>&1 | tee -i $(LOG_DIR)/hygra-$(EXT)-hypergraphDecoder.log
ifeq ($(compressed), 1)
	echo -e "\nWeighted:" 2>&1 | tee -ia $(LOG_DIR)/hygra-$(EXT)-hypergraphDecoder.log
	$(EMULATOR) ./hypergraphDecoder.$(EXT) -s -w ../../inputs/randHypergraph_weighted.compressed ../../inputs/randHypergraph_weighted_decoded \
	2>&1 | tee -ia $(LOG_DIR)/hygra-$(EXT)-hypergraphDecoder.log
endif


run_WeightedHyperReader:
	$(EMULATOR) ./WeightedHyperReader.$(EXT) -s ../../inputs/randHypergraph_weighted \
	2>&1 | tee -i $(LOG_DIR)/hygra-$(EXT)-WeightedHyperReader.log
ifeq ($(compressed), 1)
	echo -e "\nCompressed:" 2>&1 | tee -ia $(LOG_DIR)/hygra-$(EXT)-WeightedHyperReader.log
	$(EMULATOR) ./WeightedHyperReader.$(EXT) -s -c ../../inputs/randHypergraph_weighted.compressed \
	2>&1 | tee -ia $(LOG_DIR)/hygra-$(EXT)-WeightedHyperReader.log
endif

run_HyperSSSP:
	$(EMULATOR) ./HyperSSSP.$(EXT) -s ../../inputs/randHypergraph_weighted \
	2>&1 | tee -i $(LOG_DIR)/hygra-$(EXT)-HyperSSSP.log
ifeq ($(compressed), 1)
	echo -e "\nCompressed:" 2>&1 | tee -ia $(LOG_DIR)/hygra-$(EXT)-HyperSSSP.log
	$(EMULATOR) ./HyperSSSP.$(EXT) -s -c ../../inputs/randHypergraph_weighted.compressed \
	2>&1 | tee -ia $(LOG_DIR)/hygra-$(EXT)-HyperSSSP.log
endif

run_%:
	$(EMULATOR) ./$*.$(EXT) -s ../../inputs/randHypergraph \
	2>&1 | tee -i $(LOG_DIR)/hygra-$(EXT)-$*.log
ifeq ($(compressed), 1)
	echo -e "\nCompressed:" 2>&1 | tee -ia $(LOG_DIR)/hygra-$(EXT)-$*.log
	$(EMULATOR) ./$*.$(EXT) -s -c ../../inputs/randHypergraph.compressed \
	2>&1 | tee -ia $(LOG_DIR)/hygra-$(EXT)-$*.log
endif

.PHONY : clean

clean :
	rm -f *.o $(ALL:=.$(EXT))

cleansrc :
	rm -f *.o $(ALL:=.riscv) $(ALL:=.x86)
	rm $(COMMON)
