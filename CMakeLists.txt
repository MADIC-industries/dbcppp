
cmake_minimum_required(VERSION 3.12)

project("libdbcppp" VERSION 3.2.6)

option(build_tests "Build tests" ON)
option(build_examples "Build examples" ON)
option(build_libxml2 "Build libxml2" ON)
option(use_internal_boost "Build boost" ON)
option(use_internal_cxxopts "Use internal cxxopts" ON)
option(BUILD_SHARED_LIBS "Build shared library" ON)

set(CMAKE_CXX_STANDARD 17)

if (CMAKE_CXX_COMPILER_ID MATCHES "MSVC")
	add_definitions("/bigobj")
endif()

include_directories("include")
if(use_internal_boost)
  include_directories("third-party/boost")
endif(use_internal_boost)

if(build_libxml2)
  include_directories("third-party/libxml2/include")
  set(LIBXML2_WITH_ICONV OFF)
  set(LIBXML2_WITH_LZMA OFF)
  set(LIBXML2_WITH_PYTHON OFF)
  set(LIBXML2_WITH_ZLIB OFF)
  set(LIBXML2_WITH_TESTS OFF)

  add_subdirectory(third-party/libxml2)
else()
  include(FindPkgConfig)
  find_package(LibXml2 REQUIRED)
  include_directories(${LIBXML2_INCLUDE_DIR})
endif(build_libxml2)

if(use_internal_cxxopts)
  include_directories("third-party/cxxopts/include")
endif(use_internal_cxxopts)

include_directories("third-party/libxmlmm/libxmlmm")
file(GLOB libxmlmm_header
    "third-party/libxmlmm/libxmlmm/*.h"
)
file(GLOB libxmlmm_src
    "third-party/libxmlmm/libxmlmm/*.cpp"
)

add_library(libxmlmm "")
target_link_libraries(libxmlmm LibXml2)

target_sources(libxmlmm
    PRIVATE ${libxmlmm_header}
    PRIVATE ${libxmlmm_src}
)

if (build_libxml2)
install(
    TARGETS "libxmlmm" "LibXml2" EXPORT ${PROJECT_NAME}Targets
    DESTINATION ${CMAKE_INSTALL_LIBDIR})
else()
install(
    TARGETS "libxmlmm" EXPORT ${PROJECT_NAME}Targets
    DESTINATION ${CMAKE_INSTALL_LIBDIR})
endif()

install(
    DIRECTORY "libxmlmm"
    DESTINATION ${CMAKE_INSTALL_INCLUDEDIR}/libxmlmm
    FILES_MATCHING PATTERN "*.h")

add_subdirectory(src)
if (build_tests)
  add_subdirectory(tests)
endif()
if (build_examples)
  add_subdirectory(examples)
endif()
