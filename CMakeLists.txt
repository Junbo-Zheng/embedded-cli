cmake_minimum_required(VERSION 3.8...3.17)
project(embedded_cli C)
list(APPEND CMAKE_MODULE_PATH "${CMAKE_CURRENT_LIST_DIR}/cmake-modules")

set(CMAKE_C_STANDARD 11)

set(CMAKE_EXPORT_COMPILE_COMMANDS ON)

option(BUILD_TESTS "Build and run tests" OFF)
option(TESTS_COV "Run coverage on tests" OFF)
option(BUILD_SINGLE_HEADER "Build single-header version" OFF)
option(BUILD_EXAMPLES "Builds example applications" OFF)

if (BUILD_TESTS OR BUILD_EXAMPLES)
    # C++ is only used in tests and examples
    enable_language(CXX)
    set(CMAKE_CXX_STANDARD 20)
endif ()

if (${BUILD_TESTS})
    if (${TESTS_COV})
        include(CodeCoverage)
        append_coverage_compiler_flags()
    endif ()
endif ()

add_subdirectory(lib)

if (BUILD_EXAMPLES)
    if (WIN32)
        add_subdirectory(examples/win32-example)
    endif (WIN32)
    IF(${CMAKE_SYSTEM_NAME} MATCHES "Linux")
        add_subdirectory(examples/linux-example)
    ENDIF()
endif ()

if (${BUILD_TESTS})
    include(CTest)
    add_subdirectory(deps/catch2)
    add_subdirectory(tests)
    add_test(CliTests tests/embedded_cli_tests)
    if (${TESTS_COV})
        setup_target_for_coverage_gcovr_xml(
                NAME coverage_xml
                EXECUTABLE ctest
                DEPENDENCIES embedded_cli_tests
                EXCLUDE "tests/*")
    endif ()
endif ()
