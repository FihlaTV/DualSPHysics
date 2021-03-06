cmake_minimum_required(VERSION 2.8)

PROJECT(DualSPHysics)

# CPU Objects

set(OBJXML JXml.cpp tinystr.cpp tinyxml.cpp tinyxmlerror.cpp tinyxmlparser.cpp)
set(OBJSPHMOTION JMotion.cpp JMotionList.cpp JMotionMov.cpp JMotionObj.cpp JMotionPos.cpp JSphMotion.cpp)
set(OBCOMMON Functions.cpp FunctionsGeo3d.cpp JAppInfo.cpp JBinaryData.cpp JDataArrays.cpp JException.cpp JLinearValue.cpp JLog2.cpp JMeanValues.cpp JObject.cpp JOutputCsv.o JRadixSort.cpp JRangeFilter.cpp JReadDatafile.cpp JSaveCsv2.cpp JTimeControl.cpp randomc.cpp)
set(OBCOMMONDSPH JDsphConfig.cpp JPartDataBi4.cpp JPartDataHead.cpp JPartFloatBi4.cpp JPartOutBi4Save.cpp JSpaceCtes.cpp JSpaceEParms.cpp JSpaceParts.cpp JSpaceProperties.cpp JSpaceVtkOut.cpp)
set(OBSPH JArraysCpu.cpp JCellDivCpu.cpp JCfgRun.cpp JDamping.cpp JGaugeItem.cpp JGaugeSystem.cpp JPartsOut.cpp JSaveDt.cpp JShifting.cpp JSph.cpp JSphAccInput.cpp JSphCpu.cpp JSphInitialize.cpp JSphMk.cpp JSphPartsInit.cpp JSphDtFixed.cpp JSphVisco.cpp JTimeOut.cpp JWaveSpectrumGpu.cpp main.cpp)
set(OBSPHSINGLE JCellDivCpuSingle.cpp JPartsLoad4.cpp JSphCpuSingle.cpp)

# GPU Objects

set(OBCOMMONGPU FunctionsCuda.cpp JObjectGpu.cpp)
set(OBSPHGPU JArraysGpu.cpp JDebugSphGpu.cpp JCellDivGpu.cpp JSphGpu.cpp )
set(OBSPHSINGLEGPU JCellDivGpuSingle.cpp JSphGpuSingle.cpp)
set(OBCUDA JCellDivGpu_ker.cu JCellDivGpuSingle_ker.cu JGauge_ker.cu JReduSum_ker.cu JShifting_ker.cu JSphAccInput_ker.cu JSphGpu_ker.cu JSphGpuSimple_ker.cu JWaveOrder2_ker.cu)

# Other Objects

set(OBWAVERZ JMLPistonsGpu.cpp JRelaxZonesGpu.cpp)
set(OBWAVERZCUDA JRelaxZone_ker.cu)
set(OBCHRONO JChronoObjects.cpp)
set(OBINOUT JSphCpu_InOut.cpp JSphCpuSingle_InOut.cpp JSphBoundCorr.cpp JSphInOut.cpp JSphInOutGridData.cpp JSphInOutPoints.cpp JSimpleNeigs.cpp)
set(OBINOUTGPU JSphGpuSingle_InOut.cpp)

find_package(CUDA QUIET)

# NVCC Flags

if(CUDA_FOUND)
  if(CUDA_VERSION VERSION_LESS "7.5")
    message("Using cuda version <7.5")
    list(APPEND CUDA_NVCC_FLAGS "-use_fast_math -O3 -gencode=arch=compute_20,code=\"sm_20,compute_20\"")
  elseif(CUDA_VERSION VERSION_GREATER "7.4" AND CUDA_VERSION VERSION_LESS "9.1")
    message("Using cuda version >=7.5 and <9.1")
    list(APPEND CUDA_NVCC_FLAGS "-use_fast_math -O3  -gencode=arch=compute_20,code=\"sm_20,compute_20\" -gencode=arch=compute_30,code=\"sm_30,compute_30\" -gencode=arch=compute_35,code=\"sm_35,compute_35\" -gencode=arch=compute_37,code=\"sm_37,compute_37\" -gencode=arch=compute_50,code=\"sm_50,compute_50\" -gencode=arch=compute_52,code=\"sm_52,compute_52\"")
  else()
    message("Using cuda version >=9.1")
    list(APPEND CUDA_NVCC_FLAGS "-use_fast_math -O3  -gencode=arch=compute_30,code=\"sm_30,compute_30\" -gencode=arch=compute_35,code=\"sm_35,compute_35\" -gencode=arch=compute_37,code=\"sm_37,compute_37\" -gencode=arch=compute_50,code=\"sm_50,compute_50\" -gencode=arch=compute_52,code=\"sm_52,compute_52\" -gencode=arch=compute_61,code=\"sm_61,compute_61\" -gencode=arch=compute_70,code=\"sm_70,compute_70\"")
  endif()
else()
  message("CUDA Libraries were not found.")
endif()

# Static libraries linker path

if(CMAKE_CXX_COMPILER_ID STREQUAL "GNU")
    message("Using libraries for gcc")
    LINK_DIRECTORIES(${CMAKE_CURRENT_SOURCE_DIR}/../lib/linux_gcc)
elseif(MSVC)
  message("Windows version")

  #if(MSVC_VERSION VERSION_EQUAL 1900)
    LINK_DIRECTORIES(${CMAKE_CURRENT_SOURCE_DIR}/../lib/vs2015)
  #endif()
endif()

# OpenMP flags

find_package(OpenMP)

if (OPENMP_FOUND)
  set (CMAKE_C_FLAGS "${CMAKE_C_FLAGS} ${OpenMP_C_FLAGS}")
  set (CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} ${OpenMP_CXX_FLAGS}")
endif()

# Don't use C++11 ABI for compatibility between GCC v4.9 and >=5

set (CMAKE_C_FLAGS "${CMAKE_C_FLAGS} -D_GLIBCXX_USE_CXX11_ABI=0 -fPIC")
set (CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -D_GLIBCXX_USE_CXX11_ABI=0 -fPIC")

# Binaries

add_executable(DualSPHysics4.4CPU ${OBJXML} ${OBJSPHMOTION} ${OBCOMMON} ${OBCOMMONDSPH} ${OBSPH} ${OBSPHSINGLE} ${OBWAVERZ} ${OBCHRONO} ${OBINOUT})
install(TARGETS	DualSPHysics4.4CPU DESTINATION ${CMAKE_CURRENT_SOURCE_DIR}/../../bin/linux)

if (CUDA_FOUND)
   cuda_add_executable(DualSPHysics4.4 ${OBJXML} ${OBJSPHMOTION} ${OBCOMMON} ${OBCOMMONDSPH} ${OBSPH} ${OBSPHSINGLE} ${OBCOMMONGPU} ${OBSPHGPU} ${OBSPHSINGLEGPU} ${OBCUDA} ${OBWAVERZ} ${OBWAVERZCUDA} ${OBCHRONO} ${OBINOUT} ${OBINOUTGPU})
   install(TARGETS DualSPHysics4.4 DESTINATION ${CMAKE_CURRENT_SOURCE_DIR}/../../bin/linux)
endif()

# Linker flags

if(CMAKE_CXX_COMPILER_ID STREQUAL "GNU")
  target_link_libraries(DualSPHysics4.4CPU jvtklib_64 jwavegen_64 ChronoEngine dsphchrono)
  set_target_properties(DualSPHysics4.4CPU PROPERTIES COMPILE_FLAGS "-use_fast_math -O3 -D_GLIBCXX_USE_CXX11_ABI=0 -fPIC")
  
  if (CUDA_FOUND)
     target_link_libraries(DualSPHysics4.4 jvtklib_64 jwavegen_64 ChronoEngine dsphchrono)
     set_target_properties(DualSPHysics4.4 PROPERTIES COMPILE_FLAGS "-use_fast_math -O3 -D_WITHGPU -D_GLIBCXX_USE_CXX11_ABI=0 -fPIC")
  endif()
  
elseif(MSVC)
  set_target_properties(DualSPHysics4.4 PROPERTIES COMPILE_FLAGS "/D _WITHGPU")

  #if(MSVC_VERSION VERSION_EQUAL 1900)
    # MSVC 2015
    target_link_libraries(DualSPHysics4.4CPU LibJVtkLib_x64_v140_Release LibJWaveGen_x64_v140_Release dsphchrono)
    target_link_libraries(DualSPHysics4.4 LibJVtkLib_x64_v140_Release LibJWaveGen_x64_v140_Release dsphchrono)
  #endif()
  
  SET(CUDA_PROPAGATE_HOST_FLAGS OFF CACHE BOOL "Propagate C/CXX Flags and friends to the host compiler in NVCC via -Xompile  " FORCE)
  
  foreach(CPP_FLAGS CMAKE_CXX_FLAGS CMAKE_CXX_FLAGS_DEBUG CMAKE_CXX_FLAGS_RELEASE CMAKE_CXX_FLAGS_MINSIZEREL CMAKE_CXX_FLAGS_RELWITHDEBINFO)
    if(${CPP_FLAGS} MATCHES "/MD")
      string(REGEX REPLACE "/MD" "/MT" ${CPP_FLAGS} "${${CPP_FLAGS}}")
    endif(${CPP_FLAGS} MATCHES "/MD")
  endforeach(CPP_FLAGS)
endif()
