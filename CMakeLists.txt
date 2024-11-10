cmake_minimum_required(VERSION 3.24.0 FATAL_ERROR)
set(CMAKE_CXX_STANDARD 20) # Use c++20
set(CMAKE_CXX_STANDARD_REQUIRED TRUE)

set(PROJECT_NAME shv_cmake) # Project name
set(PROJECT_MAP_FILE OFF) # Generate a map file on build
set(PROJECT_OPTIMIZATION OFF) # Enable for prod

cmake_host_system_information(RESULT NCPU QUERY NUMBER_OF_LOGICAL_CORES)

project(${PROJECT_NAME} LANGUAGES CXX ASM_MASM)
set(SRC_DIR "${PROJECT_SOURCE_DIR}/src")

file(GLOB_RECURSE SRC_MAIN
  ${SRC_DIR}/**.c
  ${SRC_DIR}/**.cpp
  ${SRC_DIR}/**.cxx
  ${SRC_DIR}/**.h
  ${SRC_DIR}/**.hpp
  ${SRC_DIR}/**.hxx
  ${SRC_DIR}/**.asm
)

add_compile_options(/bigobj) # Increase Number of Sections in .Obj file
add_compile_options(/cgthreads8) # Code generation threads
add_compile_options(/MP${NCPU}) # Build with multiple processes
add_compile_options(/EHsc) # Exception handling model
add_compile_options(/MD) # Use Run-Time Library
add_compile_options("$<$<C_COMPILER_ID:MSVC>:/utf-8>")
add_compile_options("$<$<CXX_COMPILER_ID:MSVC>:/utf-8>")

add_library(${PROJECT_NAME} MODULE ${SRC_MAIN})
set_target_properties(${PROJECT_NAME} PROPERTIES SUFFIX ".asi") # This is here so we don't have to rename .dll to .asi everytime.
set_property(GLOBAL PROPERTY USE_FOLDERS ON)

source_group(TREE ${SRC_DIR} PREFIX "src" FILES ${SRC_MAIN})

target_include_directories(${PROJECT_NAME} PRIVATE
  "${SRC_DIR}"
  "${PROJECT_SOURCE_DIR}/vendor/shv/inc"
  "${PROJECT_SOURCE_DIR}/vendor/shv/lib"
)

target_precompile_headers(${PROJECT_NAME} PRIVATE 
  "${SRC_DIR}/pch.h"
)

target_link_libraries(${PROJECT_NAME} PRIVATE
  "${PROJECT_SOURCE_DIR}/vendor/shv/lib/ScriptHookV.lib"
)

target_compile_definitions(${PROJECT_NAME} PRIVATE
  "_CRT_SECURE_NO_WARNINGS"
  "NOMINMAX"
  "WIN32_LEAN_AND_MEAN"
)

if(PROJECT_MAP_FILE)
  set_target_properties(${PROJECT_NAME} PROPERTIES LINK_FLAGS "/MAP") # Generate Mapfile
endif(PROJECT_MAP_FILE)

if(PROJECT_OPTIMIZATION)
  add_compile_options(/GL) # Whole program optimization
  add_compile_options(/Oi) # Generate Intrinsic Functions
  add_compile_options(/Gy) # Enable Function-Level Linking
  set_target_properties(${PROJECT_NAME} PROPERTIES LINK_FLAGS "/LTCG /OPT:REF,ICF /GUARD:NO") # Link-time code generation, Optimizations, Guard Checks
endif(PROJECT_OPTIMIZATION)