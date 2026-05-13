cmake_minimum_required(VERSION 3.25...4.2)

project(waveshare_commander
    VERSION 1.0.0
    DESCRIPTION "CLI commander for generic Modbus TCP devices"
    LANGUAGES CXX
)

set(CMAKE_CXX_STANDARD 23)
set(CMAKE_CXX_STANDARD_REQUIRED ON)
set(CMAKE_CXX_EXTENSIONS OFF)

include(GNUInstallDirs)
include(FetchContent)

option(WAVESHARE_ENABLE_CPACK "Enable CPack packaging support" ${PROJECT_IS_TOP_LEVEL})

if(NOT TARGET waveshare)
    set(LIBWAVESHARE_ENABLE_CPACK OFF CACHE BOOL "" FORCE)
    FetchContent_Declare(
        libwaveshare_proj
        GIT_REPOSITORY https://github.com/daixtrose/waveshare-lib.git
        GIT_TAG v0.2.2
        GIT_SHALLOW TRUE
    )
    FetchContent_MakeAvailable(libwaveshare_proj)
endif()

if(NOT TARGET CLI11::CLI11)
    FetchContent_Declare(
        cli11_proj
        GIT_REPOSITORY https://github.com/CLIUtils/CLI11.git
        GIT_TAG v2.6.1
        GIT_SHALLOW TRUE
    )
    FetchContent_MakeAvailable(cli11_proj)
endif()

add_executable(waveshare_commander
    ${CMAKE_CURRENT_LIST_DIR}/src/waveshare_commander.cpp
    ${CMAKE_CURRENT_LIST_DIR}/src/cli_parser.cpp
)

set_target_properties(waveshare_commander PROPERTIES
    OUTPUT_NAME waveshare-commander
    RUNTIME_OUTPUT_DIRECTORY ${CMAKE_BINARY_DIR}/bin
)

target_include_directories(waveshare_commander
    PRIVATE
        ${CMAKE_CURRENT_LIST_DIR}/include
)

target_link_libraries(waveshare_commander
    PRIVATE
        waveshare
        CLI11::CLI11
)

target_compile_definitions(waveshare_commander PRIVATE
    PROJECT_VERSION="${PROJECT_VERSION}"
)

target_compile_features(waveshare_commander PRIVATE cxx_std_23)

if(CMAKE_CXX_COMPILER_ID MATCHES "GNU|Clang")
    target_compile_options(waveshare_commander PRIVATE
        -Wall
        -Wextra
        -Wpedantic
    )
elseif(MSVC)
    target_compile_options(waveshare_commander PRIVATE
        /W4
    )
endif()

# Statically link MinGW runtime (libstdc++, libgcc, libwinpthread) so the
# resulting .exe is fully self-contained and runs without MSYS2/MinGW installed.
if(MINGW)
    target_link_options(waveshare_commander PRIVATE -static)
endif()

# Trixie ships GCC 14 libstdc++, so no static-libstdc++ kludge needed
# any more — the deb declares libstdc++6 as a runtime Depends:.

include(${CMAKE_CURRENT_LIST_DIR}/CMakeListsCPackConfiguration.txt)
