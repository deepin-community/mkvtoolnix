cmake_minimum_required(VERSION 3.1.2)

project(matroska VERSION 1.7.1)

include(CMakeDependentOption)

include( FeatureSummary )
function(feature_info_on_off)
  set(option_name ${ARGV0})
  if(${option_name})
    add_feature_info(${option_name} ${option_name} ${ARGV1})
  else()
    add_feature_info(${option_name} $<NOT:${option_name}> ${ARGV2})
  endif()
endfunction(feature_info_on_off)

cmake_dependent_option(BUILD_SHARED_LIBS "Build libmatroska as a shared library (except Windows)" OFF "NOT WIN32" OFF)
feature_info_on_off(BUILD_SHARED_LIBS "will build as a dynamic library" "will build as a static library")

option(DISABLE_PKGCONFIG "Disable PkgConfig module generation" OFF)
feature_info_on_off(DISABLE_PKGCONFIG "won't generate pkgconfig config file" "will generate pkgconfig config file")

option(DISABLE_CMAKE_CONFIG "Disable CMake package config module generation" OFF)
feature_info_on_off(DISABLE_CMAKE_CONFIG "won't generate CMake config file" "will generate CMake config file")

option(BUILD_EXAMPLES "Build example code" OFF)
feature_info_on_off(BUILD_EXAMPLES "will build example code" "will build without example code")

find_package(EBML 1.4.3 REQUIRED)
set_package_properties(EBML PROPERTIES DESCRIPTION "libebml")

include(GNUInstallDirs)

set(CMAKE_CXX_STANDARD 14)
set(CMAKE_CXX_STANDARD_REQUIRED ON)

set(libmatroska_SOURCES
  src/FileKax.cpp
  src/KaxAttached.cpp
  src/KaxAttachments.cpp
  src/KaxBlock.cpp
  src/KaxBlockData.cpp
  src/KaxCluster.cpp
  src/KaxContexts.cpp
  src/KaxCues.cpp
  src/KaxCuesData.cpp
  src/KaxInfoData.cpp
  src/KaxSeekHead.cpp
  src/KaxSegment.cpp
  src/KaxSemantic.cpp
  src/KaxTracks.cpp
  src/KaxVersion.cpp)

set(libmatroska_PUBLIC_HEADERS
  matroska/FileKax.h
  matroska/KaxAttached.h
  matroska/KaxAttachments.h
  matroska/KaxBlockData.h
  matroska/KaxBlock.h
  matroska/KaxChapters.h
  matroska/KaxClusterData.h
  matroska/KaxCluster.h
  matroska/KaxConfig.h
  matroska/KaxContentEncoding.h
  matroska/KaxContexts.h
  matroska/KaxCuesData.h
  matroska/KaxCues.h
  matroska/KaxDefines.h
  matroska/KaxInfoData.h
  matroska/KaxInfo.h
  matroska/KaxSeekHead.h
  matroska/KaxSegment.h
  matroska/KaxSemantic.h
  matroska/KaxTag.h
  matroska/KaxTags.h
  matroska/KaxTrackAudio.h
  matroska/KaxTrackEntryData.h
  matroska/KaxTracks.h
  matroska/KaxTrackVideo.h
  matroska/KaxTypes.h
  matroska/KaxVersion.h)

set (libmatroska_C_PUBLIC_HEADERS
  matroska/c/libmatroska.h
  matroska/c/libmatroska_t.h)

add_library(matroska ${libmatroska_SOURCES} ${libmatroska_PUBLIC_HEADERS} ${libmatroska_C_PUBLIC_HEADERS})
target_link_libraries(matroska PUBLIC EBML::ebml)
set_target_properties(matroska PROPERTIES
  VERSION 7.0.0
  SOVERSION 7
  CXX_VISIBILITY_PRESET hidden
  VISIBILITY_INLINES_HIDDEN ON
)
target_include_directories(matroska
  PRIVATE
    $<BUILD_INTERFACE:${CMAKE_CURRENT_BINARY_DIR}>
	PUBLIC
    $<BUILD_INTERFACE:${CMAKE_CURRENT_SOURCE_DIR}>
    $<INSTALL_INTERFACE:${CMAKE_INSTALL_INCLUDEDIR}>)
if(MSVC)
  target_compile_definitions(matroska PRIVATE _CRT_SECURE_NO_WARNINGS)
endif()

include(GenerateExportHeader)
generate_export_header(matroska EXPORT_MACRO_NAME MATROSKA_DLL_API)
target_sources(matroska
  PRIVATE
    ${CMAKE_CURRENT_BINARY_DIR}/matroska_export.h
)

if(NOT BUILD_SHARED_LIBS)
  target_compile_definitions(matroska PUBLIC MATROSKA_STATIC_DEFINE)
endif()

if(BUILD_EXAMPLES)
  add_executable(test00 test/ebml/test00.cpp)
  target_link_libraries(test00 matroska)
  target_include_directories(test00 PRIVATE $<BUILD_INTERFACE:${CMAKE_CURRENT_BINARY_DIR}>)

  add_executable(test6 test/mux/test6.cpp)
  target_link_libraries(test6 matroska)
  target_include_directories(test6 PRIVATE $<BUILD_INTERFACE:${CMAKE_CURRENT_BINARY_DIR}>)

  add_executable(test8 test/mux/test8.cpp)
  target_link_libraries(test8 matroska)
  target_include_directories(test8 PRIVATE $<BUILD_INTERFACE:${CMAKE_CURRENT_BINARY_DIR}>)
endif()

install(TARGETS matroska
  EXPORT MatroskaTargets
  RUNTIME DESTINATION ${CMAKE_INSTALL_BINDIR}
  ARCHIVE DESTINATION ${CMAKE_INSTALL_LIBDIR}
  LIBRARY DESTINATION ${CMAKE_INSTALL_LIBDIR})

install(FILES ${libmatroska_PUBLIC_HEADERS} DESTINATION ${CMAKE_INSTALL_INCLUDEDIR}/matroska)
install(FILES ${libmatroska_C_PUBLIC_HEADERS} DESTINATION ${CMAKE_INSTALL_INCLUDEDIR}/matroska/c)
install(FILES ${CMAKE_CURRENT_BINARY_DIR}/matroska_export.h DESTINATION ${CMAKE_INSTALL_INCLUDEDIR}/matroska)

if(NOT DISABLE_PKGCONFIG)
  set(prefix ${CMAKE_INSTALL_PREFIX})
  set(exec_prefix "\$\{prefix\}")
  if (IS_ABSOLUTE ${CMAKE_INSTALL_LIBDIR})
    set(libdir "${CMAKE_INSTALL_LIBDIR}")
  else()
    set(libdir "\$\{prefix\}/${CMAKE_INSTALL_LIBDIR}")
  endif()
  if (IS_ABSOLUTE ${CMAKE_INSTALL_INCLUDEDIR})
    set(includedir "${CMAKE_INSTALL_INCLUDEDIR}")
  else()
    set(includedir "\$\{prefix\}/${CMAKE_INSTALL_INCLUDEDIR}")
  endif()
  set(PACKAGE_VERSION ${PROJECT_VERSION})
  configure_file(libmatroska.pc.in libmatroska.pc @ONLY)
  install(FILES ${CMAKE_CURRENT_BINARY_DIR}/libmatroska.pc DESTINATION ${CMAKE_INSTALL_LIBDIR}/pkgconfig)
endif()

if(NOT DISABLE_CMAKE_CONFIG)
  set(CMAKE_INSTALL_PACKAGEDIR ${CMAKE_INSTALL_LIBDIR}/cmake/Matroska)
  include(CMakePackageConfigHelpers)
  configure_package_config_file(MatroskaConfig.cmake.in ${CMAKE_CURRENT_BINARY_DIR}/MatroskaConfig.cmake
    INSTALL_DESTINATION ${CMAKE_INSTALL_PACKAGEDIR})
  write_basic_package_version_file(MatroskaConfigVersion.cmake COMPATIBILITY SameMajorVersion)
  install(EXPORT MatroskaTargets NAMESPACE Matroska:: DESTINATION ${CMAKE_INSTALL_PACKAGEDIR})
  install(FILES
    ${CMAKE_CURRENT_BINARY_DIR}/MatroskaConfig.cmake
    ${CMAKE_CURRENT_BINARY_DIR}/MatroskaConfigVersion.cmake
    DESTINATION ${CMAKE_INSTALL_PACKAGEDIR})
endif()

feature_summary(DESCRIPTION "libmatroska options:" WHAT ENABLED_FEATURES)
feature_summary(DESCRIPTION "libmatroska disabled options:" WHAT DISABLED_FEATURES)
feature_summary(DESCRIPTION "libmatroska packages used:" WHAT PACKAGES_FOUND)
