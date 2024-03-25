cmake_minimum_required(VERSION 3.0)

find_package(Trilinos REQUIRED)

#Use same compilers and flags as Trilinos
set(CMAKE_CXX_COMPILER ${Trilinos_CXX_COMPILER})
message("Compiler: ${CMAKE_CXX_COMPILER}")
set(CMAKE_C_COMPILER ${Trilinos_C_COMPILER})
set(CMAKE_Fortran_COMPILER ${Trilinos_Fortran_COMPILER})
# THIS IS A HACK!
# For some reason, the Trilinos CXX flag do not include -fopenmp
# even when it's enabled
set(CMAKE_CXX_FLAGS "${Trilinos_CXX_COMPILER_FLAGS} ${Trilinos_C_COMPILER_FLAGS} -pg")
set(CMAKE_C_FLAGS  "${Trilinos_C_COMPILER_FLAGS} ${CMAKE_C_FLAGS}")
set(CMAKE_Fortran_FLAGS  "${Trilinos_Fortran_COMPILER_FLAGS} ${Kokkos_Fortran_COMPILER_FLAGS} ${CMAKE_Fortran_FLAGS}")

if (CMAKE_BUILD_TYPE MATCHES DEBUG)
  ADD_DEFINITIONS(-DDEBUG)
  ADD_DEFINITIONS(-DHPCG_DEBUG)
endif()

if (NOT DEFINED KHPCG_EXECSPACE)
  message(WARNING "KHPCG_EXECSPACE not defined, defaulting to Serial. Choose one of {OpenMP, Serial, Cuda}")
  set(KHPCG_EXECSPACE Serial)
endif()

if (NOT DEFINED KHPCG_SYMGS)
  message(WARNING "KHPCG_SYMGS not defined, defaulting to Level. Choose one of {Color, Level, Inexact}")
  set(KHPCG_SYMGS Level)
endif()

# This is a string instead of boolean for consistency
if (NOT DEFINED KHPCG_KOKKOS_TEAM)
  message(WARNING "KHPCG_KOKKOS_TEAM not defined, defaulting to On. Choose one of {On, Off}")
  set(KHPCG_KOKKOS_TEAM On)
endif()

project(KokkosHPCG)

# MPI check 
MESSAGE("-- Checking if MPI is enabled in Trilinos:")
LIST(FIND Trilinos_TPL_LIST MPI MPI_List_ID)
IF (MPI_List_ID GREATER -1)
  MESSAGE("-- Checking if MPI is enabled in Trilinos: MPI ENABLED")
  SET(HPCG_NO_MPI OFF)
ELSE()
  MESSAGE("-- Checking if MPI is enabled in Trilinos: MPI NOT ENABLED")
  SET(HPCG_NO_MPI ON)
  ADD_DEFINITIONS(-DHPCG_NO_MPI)
ENDIF()

# KokkosCore check will exit if Kokkos is not found
message("-- Looking for Kokkos:") 
list(FIND Trilinos_PACKAGE_LIST Kokkos Kokkos_List_ID)
IF (Kokkos_List_ID GREATER -1)
  message("-- Looking for Kokkos: -- found. Proceeding...")
ELSE()
  message(FATAL_ERROR "Kokkos not found... CMake exiting")
ENDIF()

if(${KHPCG_EXECSPACE} STREQUAL "OpenMP")
  ADD_DEFINITIONS(-DHPCG_Kokkos_OpenMP)
elseif(${KHPCG_EXECSPACE} STREQUAL "Cuda")
  ADD_DEFINITIONS(-DHPCG_Kokkos_Cuda)
  ADD_DEFINITIONS(-DHPCG_NO_OPENMP)
elseif(${KHPCG_EXECSPACE} STREQUAL "Serial")
  ADD_DEFINITIONS(-DHPCG_Kokkos_Serial)
  ADD_DEFINITIONS(-DHPCG_NO_OPENMP)
else()
  message(FATAL_ERROR "Unknown KHPCG_EXECSPACE. Choose one of {OpenMP, Cuda or Serial}")
endif()
message("Execution Space: ${KHPCG_EXECSPACE}")

if(${KHPCG_SYMGS} STREQUAL "Color")
  ADD_DEFINITIONS(-DSYMGS_Color)
elseif(${KHPCG_SYMGS} STREQUAL "Level")
  ADD_DEFINITIONS(-DSYMGS_LEVEL)
elseif(${KHPCG_SYMGS} STREQUAL "Inexact")
  ADD_DEFINITIONS(-DSYMGS_INEXACT)
else()
  message(FATAL_ERROR "Unknown KHPCG_SYMGS. Choose one of {Color, Level or Inexact}")
endif()
message("HPCG Gause Seidel: ${KHPCG_SYMGS}")

if(${KHPCG_KOKKOS_TEAM} STREQUAL "On")
  ADD_DEFINITIONS(-DKOKKOS_TEAM)
elseif(${KHPCG_KOKKOS_TEAM} STREQUAL "Off")
else()
  message(FATAL_ERROR "Unknown KHPCG_KOKKOS_TEAM. Choose one of {On, Off}")
endif()
message("Use Kokkos Team: ${KHPCG_KOKKOS_TEAM}")

include_directories("${Trilinos_PREFIX}/include")
add_subdirectory(src)
