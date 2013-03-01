cmake_minimum_required(VERSION 2.8)
project(ndio-raw-plugin)

macro(show v)
  message("${v} is ${${v}}")
endmacro()

################################################################################
# CONFIG
################################################################################
configure_file(${PROJECT_SOURCE_DIR}/config.h.in ${PROJECT_BINARY_DIR}/config.h)
include_directories(${PROJECT_BINARY_DIR})

get_directory_property(ND_SOURCE_DIR PARENT_DIRECTORY)
include_directories(${ND_SOURCE_DIR})

##############################################################################
# SOURCE
##############################################################################

file(GLOB SRCS src/*.c)
file(GLOB HDRS src/*.h)

##############################################################################
# TARGETS
##############################################################################

add_definitions(-fPIC)
add_library(ndio-raw MODULE ${SRCS} ${HDRS})
target_link_libraries(ndio-raw nd)

##############################################################################
#  Testing
##############################################################################

find_package(GTEST CONFIG PATHS cmake NAMES GTest)
file(GLOB TEST_SOURCES test/*.cc)
if(GTEST_FOUND)
  find_package(Threads)
  enable_testing()
  include_directories(${PROJECT_SOURCE_DIR})
  include_directories(${GTEST_INCLUDE_DIR})
  add_executable(test-ndio-raw ${TEST_SOURCES})
  target_link_libraries(test-ndio-raw
    ${GTEST_BOTH_LIBRARIES}
    ${CMAKE_THREAD_LIBS_INIT}
    nd
    )
  add_dependencies(test-ndio-raw gtest)
  gtest_copy_shared_libraries(test-ndio-raw)
  add_test(TestNDIOSeries test-ndio-raw)
  install(TARGETS test-ndio-raw RUNTIME DESTINATION bin)
endif()

##############################################################################
#  Install
##############################################################################
install(TARGETS ndio-raw
  EXPORT ndio-raw-targets
  RUNTIME DESTINATION bin/${NDIO_PLUGIN_PATH}
  LIBRARY DESTINATION bin/${NDIO_PLUGIN_PATH}
)
export(TARGETS ndio-raw FILE ndio-raw-targets.cmake)
install(EXPORT ndio-raw-targets DESTINATION cmake)

################################################################################
#  Documentation
################################################################################

find_package(Doxygen)
if(DOXYGEN_FOUND)
  configure_file(${CMAKE_CURRENT_SOURCE_DIR}/doc/Doxyfile.in
                 ${CMAKE_CURRENT_BINARY_DIR}/doc/Doxyfile
                 @ONLY
                 )
  add_custom_target(ndio-raw-docs
    ${DOXYGEN_EXECUTABLE} ${PROJECT_BINARY_DIR}/doc/Doxyfile
    WORKING_DIRECTORY ${PROJECT_BINARY_DIR}
    COMMENT           "Generating API Docs"
    VERBATIM
    )
endif()

