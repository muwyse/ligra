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
else
TOOLCHAIN_PREFIX=riscv64-unknown-linux-gnu-
EMULATOR = $(INSTALL_DIR)/bin/qemu-riscv64 -L $(INSTALL_DIR)/sysroot
endif

COMMON= ligra.h graph.h compressedVertex.h vertex.h utils.h IO.h parallel.h gettime.h index_map.h maybe.h sequence.h edgeMap_utils.h binary_search.h quickSort.h blockRadixSort.h transpose.h parseCommandLine.h byte.h byteRLE.h nibble.h byte-pd.h byteRLE-pd.h nibble-pd.h vertexSubset.h encoder.C decoder.C

ALL= encoder decoder BFS BC BellmanFord Components Components-Shortcut Radii PageRank PageRankDelta BFSCC BFS-Bitvector KCore MIS Triangle CF

TOOLS= encoder decoder

BENCHMARKS= BFS BC BellmanFord Components Components-Shortcut Radii PageRank PageRankDelta BFSCC BFS-Bitvector KCore MIS Triangle CF

all: $(ALL)

% : %.C $(COMMON)
	$(TOOLCHAIN_PREFIX)$(PCC) $(PCFLAGS) -o $@.$(EXT) $<

$(COMMON):
	ln -s ../ligra/$@ .

run_all: $(addprefix run_,$(ALL))
	echo "run_all completed"

run_tools: $(addprefix run_,$(TOOLS))
	echo "run_tools completed"

run_benchmarks: $(addprefix run_,$(BENCHMARKS))
	echo "run_benchmarks completed"

run_encoder:
	$(EMULATOR) ./encoder.$(EXT) -s ../inputs/rMatGraph_J_5_100 ../inputs/rMatGraph_J_5_100.compressed \
	2>&1 | tee -i $(LOG_DIR)/ligra-$(EXT)-encoder.log
	echo -e "\nWeighted:" 2>&1 | tee -ia $(LOG_DIR)/ligra-$(EXT)-encoder.log
	$(EMULATOR) ./encoder.$(EXT) -s -w ../inputs/rMatGraph_WJ_5_100 ../inputs/rMatGraph_WJ_5_100.compressed \
	2>&1 | tee -ia $(LOG_DIR)/ligra-$(EXT)-encoder.log

run_decoder:
	$(EMULATOR) ./decoder.$(EXT) -s ../inputs/rMatGraph_J_5_100.compressed ../inputs/rMatGraph_J_5_100_decoded \
	2>&1 | tee -i $(LOG_DIR)/ligra-$(EXT)-decoder.log
	echo -e "\nWeighted:" 2>&1 | tee -ia $(LOG_DIR)/ligra-$(EXT)-decoder.log
	$(EMULATOR) ./decoder.$(EXT) -s -w ../inputs/rMatGraph_WJ_5_100.compressed ../inputs/rMatGraph_WJ_5_100_decoded \
	2>&1 | tee -ia $(LOG_DIR)/ligra-$(EXT)-decoder.log

run_BellmanFord:
	echo -e "Weighted:" 2>&1 | tee -i $(LOG_DIR)/ligra-$(EXT)-BellmanFord.log
	$(EMULATOR) ./BellmanFord.$(EXT) -s ../inputs/rMatGraph_WJ_5_100 \
	2>&1 | tee -ia $(LOG_DIR)/ligra-$(EXT)-BellmanFord.log
	echo -e "\nWeighted & Compressed:" 2>&1 | tee -ia $(LOG_DIR)/ligra-$(EXT)-BellmanFord.log
	$(EMULATOR) ./BellmanFord.$(EXT) -s -c ../inputs/rMatGraph_WJ_5_100.compressed \
	2>&1 | tee -ia $(LOG_DIR)/ligra-$(EXT)-BellmanFord.log

run_CF:
	echo -e "Weighted:" 2>&1 | tee -i $(LOG_DIR)/ligra-$(EXT)-CF.log
	$(EMULATOR) ./CF.$(EXT) -s ../inputs/rMatGraph_WJ_5_100 \
	2>&1 | tee -ia $(LOG_DIR)/ligra-$(EXT)-CF.log
	echo -e "\nWeighted & Compressed:" 2>&1 | tee -ia $(LOG_DIR)/ligra-$(EXT)-CF.log
	$(EMULATOR) ./CF.$(EXT) -s -c ../inputs/rMatGraph_WJ_5_100.compressed \
	2>&1 | tee -ia $(LOG_DIR)/ligra-$(EXT)-CF.log

run_Triangle:
	$(EMULATOR) ./Triangle.$(EXT) -s ../inputs/rMatGraph_J_5_100 \
	2>&1 | tee -i $(LOG_DIR)/ligra-$(EXT)-Triangle.log
	echo -e "\nCompressed:" 2>&1 | tee -ia $(LOG_DIR)/ligra-$(EXT)-Triangle.log
	$(EMULATOR) ./Triangle.$(EXT) -c ../inputs/rMatGraph_J_5_100.compressed \
	2>&1 | tee -ia $(LOG_DIR)/ligra-$(EXT)-Triangle.log

run_%:
	$(EMULATOR) ./$*.$(EXT) -s ../inputs/rMatGraph_J_5_100 \
	2>&1 | tee -i $(LOG_DIR)/ligra-$(EXT)-$*.log
	echo -e "\nCompressed:" 2>&1 | tee -ia $(LOG_DIR)/ligra-$(EXT)-$*.log
	$(EMULATOR) ./$*.$(EXT) -s -c ../inputs/rMatGraph_J_5_100.compressed \
	2>&1 | tee -ia $(LOG_DIR)/ligra-$(EXT)-$*.log
	

.PHONY : clean

clean :
	rm -f *.o $(ALL:=.$(EXT))

cleansrc :
	rm -f *.o $(ALL:=.riscv) $(ALL:=.x86)
	rm $(COMMON)
