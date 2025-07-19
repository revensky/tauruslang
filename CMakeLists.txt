cmake_minimum_required(VERSION 4.0.3)

project(TaurusLang C)
project(TaurusLang VERSION 0.1.0)
project(TaurusLang HOMEPAGE_URL https://github.com/revensky/tauruslang)

set(CMAKE_C_STANDARD 11)
set(CMAKE_C_STANDARD_REQUIRED True)

set(CMAKE_C_COMPILER clang-20)
set(CMAKE_USER_MAKE_RULES_OVERRIDE ${CMAKE_CURRENT_SOURCE_DIR}/ClangOverrides.txt)

find_package(LLVM REQUIRED CONFIG)
find_package(BISON REQUIRED)
find_package(FLEX REQUIRED)

set(MAIN_FILE ${CMAKE_CURRENT_SOURCE_DIR}/src/taurus.c)

include_directories(${LLVM_INCLUDE_DIRS})

add_definitions(${LLVM_DEFINITIONS})

llvm_map_components_to_libnames(
    llvm_libs
    ${LLVM_TARGETS_TO_BUILD}
    support
    core
    executionengine
    mcjit
    interpreter
    analysis
    native
    bitwriter
    irreader
    codegen
    mc
    mcparser
    option
)

bison_target(parser src/parser.y "${CMAKE_CURRENT_SOURCE_DIR}/src/parser.c"
             COMPILE_FLAGS "-Wcounterexamples")

flex_target(lexer src/lexer.l "${CMAKE_CURRENT_SOURCE_DIR}/src/lexer.c")

add_flex_bison_dependency(lexer parser)

add_executable(taurus ${BISON_parser_OUTPUTS} ${FLEX_lexer_OUTPUTS} ${MAIN_FILE})

target_link_libraries(taurus ${FLEX_LIBRARIES} ${llvm_libs})
