# Build for srcComplexity

cmake_minimum_required(VERSION 3.14.0)

project(srcComplexity VERSION "1.0.0")

find_package(LibXml2 REQUIRED)

# Build srcComplexity
add_executable(srcComplexity srcComplexity.cpp)
target_link_libraries(srcComplexity PUBLIC srcmlxpath_shared)
install(TARGETS srcComplexity RUNTIME)

# Static srcmlxpath library
add_library(srcmlxpath STATIC srcMLXPathCount.cpp)
target_include_directories(srcmlxpath PUBLIC ${LIBXML2_INCLUDE_DIR})
target_link_libraries(srcmlxpath PUBLIC ${LIBXML2_LIBRARY})
install(TARGETS srcmlxpath LIBRARY)

# Shared srcmlxpath library
add_library(srcmlxpath_shared SHARED srcMLXPathCount.cpp)
target_include_directories(srcmlxpath_shared PUBLIC ${LIBXML2_INCLUDE_DIR})
target_link_libraries(srcmlxpath_shared PUBLIC ${LIBXML2_LIBRARY})
set_target_properties(srcmlxpath_shared PROPERTIES OUTPUT_NAME srcmlxpath)
set_target_properties(srcmlxpath_shared PROPERTIES VERSION ${PROJECT_VERSION})
set_target_properties(srcmlxpath_shared PROPERTIES SOVERSION ${PROJECT_VERSION_MAJOR})
install(TARGETS srcmlxpath_shared LIBRARY)

# run example
set(SRCCOMPLEXITY_TESTFILE srcMLXPathCount.cpp.xml)
configure_file(${SRCCOMPLEXITY_TESTFILE} ${SRCCOMPLEXITY_TESTFILE} COPYONLY)
add_custom_target(run DEPENDS srcComplexity
                      COMMAND ./srcComplexity ${SRCCOMPLEXITY_TESTFILE})

# Strip executables and shared libraries
# Verify that it works
set(CPACK_STRIP_FILES ON)

# Copyright
set(CPACK_RESOURCE_FILE_LICENSE ${CMAKE_SOURCE_DIR}/COPYING.txt)
install(FILES ${CMAKE_SOURCE_DIR}/COPYING.txt DESTINATION share/doc/srccomplexity RENAME copyright)

# Description
set(CPACK_PACKAGE_DESCRIPTION_SUMMARY "Calculate source code complexity")
file(WRITE ${CMAKE_BINARY_DIR}/description.txt
"srcComplexity Hi

")
set(CPACK_PACKAGE_DESCRIPTION_FILE ${CMAKE_BINARY_DIR}/description.txt)

# Changelog file - required by lintian
file(WRITE ${CMAKE_BINARY_DIR}/changelog 
"srcComplexity (1.0.0-1) UNRELEASED; urgency=medium
  * Initial release
 -- Michael L. Collard <collard@uakron.edu>  Mon, 2 Nov 2020 17:00:00 +0400
"
)
execute_process(COMMAND gzip -n -9 -f ${CMAKD_BINARY_DIR}/changelog)
install(FILES ${CMAKE_BINARY_DIR}/changelog.gz DESTINATION share/doc/srccomplexity)

# Dependencies
set(CPACK_DEBIAN_PACKAGE_DEBUG OFF)
set(CPACK_DEBIAN_PACKAGE_SHLIBDEPS ON)
set(CPACK_DEBIAN_PACKAGE_GENERATE_SHLIBS ON)

# Handle ldconfig
set(TRIGGERS_FILE ${CMAKE_CURRENT_BINARY_DIR}/triggers)
file(WRITE ${TRIGGERS_FILE} "activate-noawait ldconfig\n")

# Remove the conf files that CPackDeb installs by default
file(WRITE ${CMAKE_BINARY_DIR}/postinst "#!/bin/bash

set -e

if [ \"$1\" = \"configure\" ]; then
    echo
fi
")
file(WRITE ${CMAKE_BINARY_DIR}/postrm "#!/bin/bash

set -e

if [ \"$1\" = \"configure\" ]; then
    echo
fi
")

# Extra files are triggers, postinst, and postrm
set(CPACK_DEBIAN_PACKAGE_CONTROL_EXTRA ${TRIGGERS_FILE} ${CMAKE_BINARY_DIR}/postinst ${CMAKE_BINARY_DIR}/postrm)

include(GNUInstallDirs)

set(CPACK_PACKAGE_NAME "srccomplexity")
set(CPACK_PACKAGE_VERSION_MAJOR "1")
set(CPACK_PACKAGE_VERSION_MINOR "0")
set(CPACK_PACKAGE_VERSION_PATCH "0")
set(CPACK_PACKAGE_CONTACT "Michael L. Collard <collard@uakron.edu>")
include(CPack)
