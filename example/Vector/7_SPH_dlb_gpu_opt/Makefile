include ../../example.mk

### This is a trick to avoid "Command not found if you no not have NVCC compiler". In practice the normal C++ compiler is used
### internally the example disable with the preprocessor its code if not compiled with nvcc 
CUDA_CC=
CUDA_CC_LINK=
ifeq (, $(shell which nvcc))
        CUDA_CC=mpic++ -x c++ $(INCLUDE_PATH)
        INCLUDE_PATH_NVCC=
        CUDA_CC_LINK=mpic++
	CUDA_OPTIONS=
else
        CUDA_CC=nvcc -ccbin=mpic++
        CUDA_CC_LINK=nvcc -ccbin=mpic++
	CUDA_OPTIONS=-use_fast_math  -arch=sm_61 -lineinfo
endif

ifeq ($(PROFILE),ON)
        CUDA_CC=scorep --nocompiler  --cuda --mpp=mpi nvcc
        CUDA_CC_LINK=scorep --nocompiler  --cuda --mpp=mpi nvcc
else
	CUDA_CC:=$(CUDA_CC)
	CUDA_CC_LINK:=$(CUDA_CC_LINK)
endif

LDIR =
OPT=

OBJ = main.o

sph_dlb:
sph_dlb_test: OPT += -DTEST_RUN
sph_dlb_test: sph_dlb

%.o: %.cu
	$(CUDA_CC) -O3 $(OPT) $(CUDA_OPTIONS)  -g -c --std=c++11 -o $@ $< $(INCLUDE_PATH_NVCC)

%.o: %.cpp
	$(CC) -O3 $(OPT) -g -c --std=c++11 -o $@ $< $(INCLUDE_PATH)

sph_dlb: $(OBJ)
	$(CUDA_CC_LINK) -o $@ $^ $(CFLAGS) $(LIBS_PATH) $(LIBS)

sph_dlb2: $(OBJ)
	$(CUDA_CC_LINK) -o $@ $^ $(CFLAGS) $(LIBS_PATH) $(LIBS)

all: sph_dlb

run: sph_dlb_test
	mpirun -np 2 ./sph_dlb

.PHONY: clean all run

clean:
	rm -f *.o *~ core sph_dlb

