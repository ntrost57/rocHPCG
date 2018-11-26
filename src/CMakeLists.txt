# Get full git commit SHA-1
set(GIT_COMMAND ${GIT_EXECUTABLE} rev-parse -q HEAD)
execute_process(COMMAND ${GIT_COMMAND}
                WORKING_DIRECTORY ${CMAKE_CURRENT_SOURCE_DIR}
                OUTPUT_VARIABLE rocHPCG_GIT_REV
                OUTPUT_STRIP_TRAILING_WHITESPACE
                ERROR_QUIET)

# Configure a header file to pass the rocALUTION version
configure_file("${CMAKE_CURRENT_SOURCE_DIR}/Version.hpp.in"
               "${PROJECT_BINARY_DIR}/include/Version.hpp"
               @ONLY
)

# HPCG HIP sources
set(HIP_SOURCE
  ComputeDotProduct.cpp
  ComputeMG.cpp
  ComputeSPMV.cpp
  ComputeSYMGS.cpp
  ComputeWAXPBY.cpp
  GenerateProblem.cpp
  SetupHalo.cpp
)

# HPCG sources
set(SOURCE
  CG.cpp
  CG_ref.cpp
  CheckAspectRatio.cpp
  CheckProblem.cpp
  ComputeDotProduct_ref.cpp
  ComputeMG_ref.cpp
  ComputeOptimalShapeXYZ.cpp
  ComputeProlongation_ref.cpp
  ComputeResidual.cpp
  ComputeRestriction_ref.cpp
  ComputeSPMV_ref.cpp
  ComputeSYMGS_ref.cpp
  ComputeWAXPBY_ref.cpp
  ExchangeHalo.cpp
  GenerateCoarseProblem.cpp
  GenerateGeometry.cpp
  GenerateProblem_ref.cpp
  MixedBaseCounter.cpp
  OptimizeProblem.cpp
  OutputFile.cpp
  ReadHpcgDat.cpp
  ReportResults.cpp
  SetupHalo_ref.cpp
  TestCG.cpp
  TestNorms.cpp
  TestSymmetry.cpp
  WriteProblem.cpp
  YAML_Doc.cpp
  YAML_Element.cpp
  finalize.cpp
  init.cpp
  main.cpp
  mytimer.cpp
)

# Flag source files as hip source files
foreach(i ${HIP_SOURCE})
  set_source_files_properties(${i} PROPERTIES HIP_SOURCE_PROPERTY_FORMAT TRUE)
endforeach()

# Target executable
hip_add_executable(rocHPCG ${SOURCE} ${HIP_SOURCE})

# Target compile definitions
target_compile_definitions(rocHPCG PRIVATE __HIP_PLATFORM_HCC__)
if(NOT HPCG_OPENMP)
  target_compile_definitions(rocHPCG PRIVATE HPCG_NO_OPENMP)
endif()

if(NOT HPCG_MPI)
  target_compile_definitions(rocHPCG PRIVATE HPCG_NO_MPI)
endif()

if(HPCG_DEBUG)
  target_compile_definitions(rocHPCG PRIVATE HPCG_DEBUG)
endif()

if(HPCG_DETAILED_DEBUG)
  target_compile_definitions(rocHPCG PRIVATE HPCG_DETAILED_DEBUG)
endif()

if(HPCG_DETAILED_TIMING)
  target_compile_definitions(rocHPCG PRIVATE HPCG_DETAILED_TIMING)
endif()



# Target include directories
target_include_directories(rocHPCG PRIVATE
                             ${PROJECT_BINARY_DIR}/include
                             ${HIP_INCLUDE_DIRS}
                             ${HIPCUB_INCLUDE_DIRS}
                             ${ROCPRIM_INCLUDE_DIRS})
if(HPCG_MPI)
  target_include_directories(rocHPCG PRIVATE ${MPI_CXX_INCLUDE_DIRS})
endif()




# Target link libraries
foreach(target ${AMDGPU_TARGETS})
  target_link_libraries(rocHPCG PRIVATE --amdgpu-target=${target})
endforeach()

if(HPCG_OPENMP)
  target_link_libraries(rocHPCG PRIVATE ${OpenMP_CXX_FLAGS})
endif()

if(HPCG_MPI)
  target_link_libraries(rocHPCG PRIVATE ${MPI_CXX_LIBRARIES})
endif()






# Target compile options
if(HPCG_OPENMP)
  target_compile_options(rocHPCG PRIVATE ${OpenMP_CXX_FLAGS})
endif()

if(HPCG_MPI)
  target_compile_options(rocHPCG PRIVATE ${MPI_COMPILE_OPTIONS})
endif()




# Target properties
set_target_properties(rocHPCG PROPERTIES VERSION ${rocHPCG_VERSION})
set_target_properties(rocHPCG PROPERTIES RUNTIME_OUTPUT_DIRECTORY "${CMAKE_BINARY_DIR}/bin")