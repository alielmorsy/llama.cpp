# mtmd

find_package(Threads REQUIRED)

add_library(mtmd
        mtmd.cpp
        mtmd-audio.cpp
        mtmd.h
        clip.cpp
        clip.h
        clip-impl.h
        mtmd-helper.cpp
        mtmd-helper.h
)

target_link_libraries(mtmd PUBLIC ggml llama)
target_link_libraries(mtmd PRIVATE Threads::Threads)
target_include_directories(mtmd PUBLIC .)
target_include_directories(mtmd PRIVATE ../..)
target_include_directories(mtmd PRIVATE ../../vendor)
target_compile_features(mtmd PRIVATE cxx_std_17)

if (BUILD_SHARED_LIBS)
    set_target_properties(mtmd PROPERTIES POSITION_INDEPENDENT_CODE ON)
    target_compile_definitions(mtmd PRIVATE LLAMA_BUILD)
    target_compile_definitions(mtmd PUBLIC LLAMA_SHARED)
endif ()

set(MTMD_PUBLIC_HEADERS
        ${CMAKE_CURRENT_SOURCE_DIR}/mtmd.h
        ${CMAKE_CURRENT_SOURCE_DIR}/mtmd-helper.h
)

set_target_properties(mtmd
        PROPERTIES
        PUBLIC_HEADER "${MTMD_PUBLIC_HEADERS}")

install(TARGETS mtmd LIBRARY PUBLIC_HEADER)

include(CMakePackageConfigHelpers)

configure_package_config_file(
        ${CMAKE_CURRENT_SOURCE_DIR}/mtmd-config.cmake.in
        ${CMAKE_CURRENT_BINARY_DIR}/mtmd-config.cmake
        INSTALL_DESTINATION ${CMAKE_INSTALL_LIBDIR}/cmake/mtmd
        PATH_VARS LLAMA_INCLUDE_INSTALL_DIR
        LLAMA_LIB_INSTALL_DIR
        LLAMA_BIN_INSTALL_DIR)


write_basic_package_version_file(
        ${CMAKE_CURRENT_BINARY_DIR}/mtmd-version.cmake
        VERSION ${LLAMA_INSTALL_VERSION}
        COMPATIBILITY SameMajorVersion)

install(FILES ${CMAKE_CURRENT_BINARY_DIR}/mtmd-config.cmake
        ${CMAKE_CURRENT_BINARY_DIR}/mtmd-version.cmake
        DESTINATION ${CMAKE_INSTALL_LIBDIR}/cmake/mtmd)

if (NOT MSVC)
    # for stb_image.h and miniaudio.h
    target_compile_options(mtmd PRIVATE -Wno-cast-qual)
endif ()

if (TARGET BUILD_INFO)
    add_dependencies(mtmd BUILD_INFO)
    add_dependencies(mtmd-helper BUILD_INFO)
endif ()


if (LLAMA_STANDALONE)
    add_executable(llama-llava-cli deprecation-warning.cpp)
    add_executable(llama-gemma3-cli deprecation-warning.cpp)
    add_executable(llama-minicpmv-cli deprecation-warning.cpp)
    add_executable(llama-qwen2vl-cli deprecation-warning.cpp)

    set(TARGET llama-mtmd-cli)
    add_executable(${TARGET} mtmd-cli.cpp)
    set_target_properties(${TARGET} PROPERTIES OUTPUT_NAME llama-mtmd-cli)
    install(TARGETS ${TARGET} RUNTIME)
    target_link_libraries(${TARGET} PRIVATE common mtmd Threads::Threads)
    target_compile_features(${TARGET} PRIVATE cxx_std_17)
endif ()
message(STATUS "Public headers: ${MTMD_PUBLIC_HEADERS}")