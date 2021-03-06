cmake_minimum_required (VERSION 2.6)
project (Tick)
 
# The version number.
set (Tick_VERSION_MAJOR 1)
set (Tick_VERSION_MINOR 2)

include(CheckCXXCompilerFlag)
enable_language(CXX)

if(CMAKE_HOST_APPLE)
    list(APPEND CXX_EXTRA_FLAGS -stdlib=libc++)
endif()

set(ENABLE_CXXFLAGS_TO_CHECK 
    -std=gnu++1y 
    -std=c++1y
    -std=gnu++11 
    -std=c++11
    -std=gnu++0x 
    -std=c++0x)

foreach(flag ${ENABLE_CXXFLAGS_TO_CHECK})
    string(REPLACE "-std=" "_" flag_var ${flag})
    string(REPLACE "+" "x" flag_var ${flag_var})
    check_cxx_compiler_flag("${flag}" COMPILER_HAS_CXX_FLAG${flag_var})
    if(COMPILER_HAS_CXX_FLAG${flag_var})
        list(APPEND CXX_EXTRA_FLAGS ${flag})
        break()
    endif()
endforeach()

install (DIRECTORY tick DESTINATION include)

configure_file(tick.pc.in tick.pc)
install(FILES tick.pc DESTINATION lib/pkgconfig)


# Moved down, so everything above is common
if (BIICODE) 
    ADD_BII_TARGETS()
    target_compile_options(${BII_LIB_TARGET} INTERFACE ${CXX_EXTRA_FLAGS})
    return() #finish processing, no need to run cmake below
endif()

add_custom_target(check COMMAND ${CMAKE_CTEST_COMMAND})

macro(add_test_executable TEST_NAME_)
    set(TEST_NAME "${TEST_NAME_}")
    add_executable (${TEST_NAME} EXCLUDE_FROM_ALL test/${TEST_NAME}.cpp)
    target_compile_options(${TEST_NAME} PUBLIC ${CXX_EXTRA_FLAGS})
    if(WIN32)
        add_test(NAME ${TEST_NAME} WORKING_DIRECTORY ${LIBRARY_OUTPUT_PATH} COMMAND ${TEST_NAME}${CMAKE_EXECUTABLE_SUFFIX})
    else()
        add_test(NAME ${TEST_NAME} COMMAND ${TEST_NAME})
    endif()
    add_dependencies(check ${TEST_NAME})
    set_tests_properties(${TEST_NAME} PROPERTIES FAIL_REGULAR_EXPRESSION "FAILED")
endmacro(add_test_executable)

include(CTest)

include_directories(.)

add_test_executable(builder)
add_test_executable(fold)
add_test_executable(integral_constant)
add_test_executable(matches)
add_test_executable(requires)
add_test_executable(set)
add_test_executable(tag)
add_test_executable(trait_check)
add_test_executable(traits)

####################
# add include headers to IDE

file(GLOB tick_include
  RELATIVE "${CMAKE_CURRENT_SOURCE_DIR}" 
  "${CMAKE_CURRENT_SOURCE_DIR}/tick/*.h"
)
add_custom_target(tick_include SOURCES ${tick_include})

file(GLOB tick_include
  RELATIVE "${CMAKE_CURRENT_SOURCE_DIR}"
  "${CMAKE_CURRENT_SOURCE_DIR}/tick/detail/*.h"
)
add_custom_target(tick_include_detail SOURCES ${tick_include})

# end headers in IDE
####################

