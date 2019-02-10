cmake_minimum_required(VERSION 3.10 FATAL_ERROR)
project(hello-world LANGUAGES CXX)

add_executable(hello-world "")

target_compile_definitions(hello-world PUBLIC "COMPILER_NAME=\"${CMAKE_CXX_COMPILER_ID}\"")
if(CMAKE_CXX_COMPILER_ID MATCHES Intel)
  target_compile_definitions(hello-world PUBLIC "IS_INTEL_CXX_COMPILER")
elseif(CMAKE_CXX_COMPILER_ID MATCHES GNU)
  target_compile_definitions(hello-world PUBLIC "IS_GNU_CXX_COMPILER")
elseif(CMAKE_CXX_COMPILER_ID MATCHES PGI)
  target_compile_definitions(hello-world PUBLIC "IS_PGI_CXX_COMPILER")
elseif(CMAKE_CXX_COMPILER_ID MATCHES XL)
  target_compile_definitions(hello-world PUBLIC "IS_XL_CXX_COMPILER")
endif()

target_sources(hello-world
        PRIVATE
        hello-world.cpp
        )

target_include_directories(hello-world
        PRIVATE
        ${PROJECT_BINARY_DIR}
        )

foreach(key
        IN ITEMS
        NUMBER_OF_LOGICAL_CORES
        NUMBER_OF_PHYSICAL_CORES
        TOTAL_VIRTUAL_MEMORY
        AVAILABLE_VIRTUAL_MEMORY
        TOTAL_PHYSICAL_MEMORY
        AVAILABLE_PHYSICAL_MEMORY
        IS_64BIT
        HAS_FPU
        HAS_MMX
        HAS_MMX_PLUS
        HAS_SSE
        HAS_SSE2
        HAS_SSE_FP
        HAS_SSE_MMX
        HAS_AMD_3DNOW
        HAS_AMD_3DNOW_PLUS
        HAS_IA64
        OS_NAME
        OS_RELEASE
        OS_VERSION
        OS_PLATFORM
        )
    cmake_host_system_information(RESULT _${key} QUERY ${key})
endforeach()

configure_file(config.h.in config.h @ONLY)
