cmake_minimum_required(VERSION 3.8 FATAL_ERROR)
project(openfpm_pdata LANGUAGES C CXX)

enable_testing()

if (POLICY CMP0074)
  cmake_policy(SET CMP0074 NEW)
endif()

list(APPEND CMAKE_MODULE_PATH ${CMAKE_CURRENT_LIST_DIR}/cmake_modules/)

set(BOOST_INCLUDE ${Boost_INCLUDE_DIR} CACHE PATH "Include directory for BOOST")
set(PETSC_ROOT CACHE PATH "If compiling with linear algebra indicate the PETSC root directory")
set(PARMETIS_ROOT CACHE PATH "Parmetis root directory")
set(METIS_ROOT CACHE PATH "Metis root directory")
set(LIBHILBERT_ROOT CACHE PATH "LibHilbert root path")
set(HDF5_ROOT CACHE PATH "HDF5 root path")
set(EIGEN3_ROOT CACHE PATH "Eigen3 include path")
set(LIBHILBERT_ROOT CACHE PATH "LibHilbert root path")
set(SUITESPARSE_ROOT CACHE PATH "The suitesparse root path")
set(TINYOBJLOADER_ROOT CACHE PATH "TinyObjLoader library path")
set(SE_CLASS1 CACHE BOOL "Activate compilation with SE_CLASS1")
set(SE_CLASS2 CACHE BOOL "Activate compilation with SE_CLASS2")
set(SE_CLASS3 CACHE BOOL "Activate compilation with SE_CLASS3")
set(ACTION_ON_ERROR CACHE STRING "Action to perform in case of error")
set(PROFILE_WITH_SCOREP CACHE BOOL "Enable profiling with scorep")
set(ENV{PETSC_DIR} ${PETSC_ROOT})
set(ENV{HDF5_ROOT} ${HDF5_ROOT})
set(ENV{EIGEN3_ROOT} ${EIGEN3_ROOT})
set(METIS_DIR ${METIS_ROOT})
set(PARMETIS_DIR ${PARMETIS_ROOT})
set(OPENBLAS_ROOT CACHE PATH "Root path for blas library")
set(SuiteSparse_ROOT ${SUITESPARSE_ROOT})

set (CMAKE_CXX_STANDARD 11)
set (CMAKE_CUDA_STANDARD 11)

if (OPENBLAS_ROOT)
	set(ENV{LD_LIBRARY_PATH} "$ENV{LD_LIBRARY_PATH}:${OPENBLAS_ROOT}/lib")
	set(ENV{DYLD_LIBRARY_PATH} "$ENV{DYLD_LIBRARY_PATH}:${OPENBLAS_ROOT}/lib")
endif()

set(ENV{PATH} "$ENV{PATH}:${HDF5_ROOT}/bin")
set(HDF5_PREFER_PARALLEL TRUE)

if(ENABLE_GPU)
	enable_language(CUDA)
	find_package(CUDA)

	if (CUDA_VERSION_MAJOR EQUAL 9 AND CUDA_VERSION_MINOR EQUAL 2)
		message("CUDA is compatible")
		set(WARNING_SUPPRESSION_AND_OPTION_NVCC  -Xcudafe "--display_error_number --diag_suppress=611 --diag_suppress=2885 --diag_suppress=2886  --diag_suppress=2887  --diag_suppress=2888 --diag_suppress=186 --diag_suppress=111" --expt-extended-lambda)
		set(WARNING_SUPPRESSION_AND_OPTION_NVCC_TEXT  "-Xcudafe \"--display_error_number --diag_suppress=611 --diag_suppress=2885 --diag_suppress=2886  --diag_suppress=2887  --diag_suppress=2888 --diag_suppress=186 --diag_suppress=111\" --expt-extended-lambda")
	elseif ( CUDA_VERSION_MAJOR EQUAL 10 AND CUDA_VERSION_MINOR EQUAL 1 )
                message("CUDA is compatible")
                set(WARNING_SUPPRESSION_AND_OPTION_NVCC  -Xcudafe "--display_error_number --diag_suppress=2915 --diag_suppress=2914  --diag_suppress=2912 --diag_suppress=2913 --diag_suppress=111 --diag_suppress=186 --diag_suppress=611 " --expt-extended-lambda)
		set(WARNING_SUPPRESSION_AND_OPTION_NVCC_TEXT  "-Xcudafe \"--display_error_number --diag_suppress=2915 --diag_suppress=2914 --diag_suppress=2912 --diag_suppress=2913 --diag_suppress=111 --diag_suppress=186 --diag_suppress=611 \" --expt-extended-lambda")
	else()
		message(FATAL_ERROR "CUDA is incompatible, version 9.2 is only supported")
	endif()

endif()


find_package(Boost 1.68.0 COMPONENTS unit_test_framework iostreams program_options)
find_package(MPI)
find_package(PETSc)
find_package(HDF5)
find_package(Eigen3)
find_package(LibHilbert)
find_package(Metis)
find_package(ParMetis)
find_package(TinyObjLoader )
find_package(BLAS)
find_package(LAPACK)
find_package(Eigen3)
find_package(SuiteSparse OPTIONAL_COMPONENTS UMFPACK)

if(PROFILE_WITH_SCOREP)
	set(CMAKE_CXX_COMPILER_LAUNCHER "scorep")
	set(CMAKE_CC_COMPILER_LAUNCHER "scorep")
	set(CMAKE_CUDA_COMPILER_LAUNCHER "scorep")
endif()

if(CUDA_FOUND)
	set(OPENFPM_INIT_FILE "initialize/initialize_wrapper_cuda.cu")
else()
	set(OPENFPM_INIT_FILE "initialize/initialize_wrapper_cpu.cpp")
endif()

###### CONFIG.h FILE ######

if(SE_CLASS1)
	set(DEFINE_SE_CLASS1 "#define SE_CLASS1")
endif()

if(ACTION_ON_ERROR)
	set(DEFINE_ACTION_ON_ERROR "#define ${ACTION_ON_ERROR}")
endif()

if(SE_CLASS2)
	set(DEFINE_SE_CLASS2 "#define SE_CLASS2")
endif()

if(SE_CLASS3)
	set(DEFINE_SE_CLASS3 "#define SE_CLASS3")
endif()

if(PETSC_FOUND)
	set(DEFINE_HAVE_PETSC "#define HAVE_PETSC")
endif()

if(SCAN_COVERTY)
	set(DEFINE_SCAN_COVERTY "#define COVERTY_SCAN")
endif()

if(TEST_PERFORMANCE)
	set(DEFINE_PERFORMANCE_TEST "#define PERFORMANCE_TEST")
endif()

if(CUDA_FOUND)
	set(DEFINE_CUDA_GPU "#define CUDA_GPU")
endif()

if (METIS_FOUND)
	set(DEFINE_HAVE_METIS "#define HAVE_METIS 1")
else()
	file(WRITE error_code "201")
	message( FATAL_ERROR "Metis is required in order to install OpenFPM" )
endif()

if (PARMETIS_FOUND)
	set(DEFINE_HAVE_PARMETIS "#define HAVE_PARMETIS 1")
else()
	file(WRITE error_code "208")
	message( FATAL_ERROR "ParMetis is required in order to install OpenFPM")
endif()

if(MPI_FOUND)
	set(DEFINE_HAVE_MPI "#define HAVE_MPI")
else()
	file(WRITE error_code "200")
	message( FATAL_ERROR "MPI is required in order to install OpenFPM" )
endif()

if (Boost_FOUND)
	set(DEFINE_HAVE_BOOST "#define HAVE_BOOST")
	set(DEFINE_HAVE_BOOST_IOSTREAMS "#define HAVE_BOOST_IOSTREAMS")
	set(DEFINE_HAVE_BOOST_PROGRAM_OPTIONS "#define HAVE_BOOST_PROGRAM_OPTIONS")
	set(DEFINE_HAVE_BOOST_UNIT_TEST_FRAMEWORK "#define HAVE_BOOST_UNIT_TEST_FRAMEWORK")
else()
	file(WRITE error_code "202")
	message( FATAL_ERROR "BOOST is required in order to install OpenFPM" )
endif()

if(HDF5_FOUND)
	if (HDF5_IS_PARALLEL)
		set(DEFINE_HAVE_HDF5 "#define HAVE_HDF5")
	else()
		file(WRITE error_code "207")
		message( FATAL_ERROR "HDF5 found ${HDF5_INCLUDE_DIRS} does not have parallel support, OpenFPM require it" )
	endif()
else()
	file(WRITE error_code "207")
	message( FATAL_ERROR "HDF5 with parallel support is required in order to install OpenFPM" )
endif()

if(EIGEN3_FOUND)
	set(DEFINE_HAVE_EIGEN "#define HAVE_EIGEN")
endif()

if(LIBHILBERT_FOUND)
	set(DEFINE_HAVE_LIBHILBERT "#define HAVE_LIBHILBERT 1")
else()
	file(WRITE error_code "210")
	message( FATAL_ERROR "LibHilbert is required in order to install OpenFPM")
endif()

if(SUITESPARSE_FOUND AND SuiteSparse_UMFPACK_FOUND)
	set(DEFINE_HAVE_SUITESPARSE "#define HAVE_SUITESPARSE")
endif()

if(APPLE)
	set(DEFINE_HAVE_OSX "#define HAVE_OSX")
endif()

if(TINYOBJLOADER_FOUND)
	set(DEFINE_HAVE_TINYOBJLOADER "#define HAVE_TINYOBJLOADER 1")
endif()

file(WRITE error_code "0")
file(WRITE cuda_lib "${CUDA_cudart_static_LIBRARY} ${CUDA_cudadevrt_LIBRARY}")
file(WRITE cuda_include "-I${CUDA_INCLUDE_DIRS}")
file(WRITE mpi_include "-I${MPI_C_INCLUDE_DIRS}")
file(WRITE mpi_libs "${MPI_C_LINK_FLAGS} ${MPI_C_LIBRARIES}")
file(WRITE cuda_suppress_wr "${WARNING_SUPPRESSION_AND_OPTION_NVCC_TEXT}")

configure_file(${CMAKE_CURRENT_SOURCE_DIR}/src/config/config_cmake.h.in ${CMAKE_CURRENT_SOURCE_DIR}/src/config/config.h)

add_subdirectory (src)
add_subdirectory (openfpm_devices)
add_subdirectory (images)
add_subdirectory (openfpm_data)
add_subdirectory (openfpm_io)
add_subdirectory (openfpm_vcluster)
add_subdirectory (openfpm_numerics)

