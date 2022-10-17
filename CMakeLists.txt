# CMakeLists.txt for:
# https://gitlab.mpcdf.mpg.de/mtr/pocketfft

cmake_minimum_required(VERSION 3.5)

project(pocketfft LANGUAGES C)

option(BUILD_TESTS "Build tests." OFF)

include(CheckSymbolExists)
include(GNUInstallDirs)

set(sources pocketfft.h pocketfft.c)

if(WIN32 AND BUILD_SHARED_LIBS)
    list(APPEND sources exports.def)
endif()

add_library(pocketfft ${sources})

# https://stackoverflow.com/questions/32816646/can-cmake-detect-if-i-need-to-link-to-libm-when-using-pow-in-c
if(NOT HAVE_SQRT AND NOT NEED_LINKING_AGAINST_LIBM)
    check_symbol_exists(sqrt "math.h" HAVE_SQRT)
    if(NOT HAVE_SQRT)
        unset(HAVE_SQRT CACHE)
        list(APPEND CMAKE_REQUIRED_LIBRARIES m)
        check_symbol_exists(sqrt "math.h" HAVE_SQRT)
        if(HAVE_SQRT)
            set(NEED_LINKING_AGAINST_LIBM True CACHE BOOL "" FORCE)
        else()
            message(FATAL_ERROR "sqrt() not found.")
        endif()
    endif()
endif()

if(NEED_LINKING_AGAINST_LIBM)
    target_link_libraries(pocketfft PUBLIC m)
endif()

set_target_properties(pocketfft PROPERTIES PUBLIC_HEADER pocketfft.h)

install(TARGETS pocketfft
    EXPORT pocketfft-targets
    PUBLIC_HEADER DESTINATION "${CMAKE_INSTALL_INCLUDEDIR}"
    ARCHIVE DESTINATION "${CMAKE_INSTALL_LIBDIR}"
    LIBRARY DESTINATION "${CMAKE_INSTALL_LIBDIR}"
    RUNTIME DESTINATION "${CMAKE_INSTALL_BINDIR}"
    INCLUDES DESTINATION "${CMAKE_INSTALL_INCLUDEDIR}"
)

if(MSVC AND BUILD_SHARED_LIBS)
    install(FILES $<TARGET_PDB_FILE:pocketfft>
        DESTINATION "${CMAKE_INSTALL_BINDIR}" OPTIONAL
    )
endif()

install(EXPORT pocketfft-targets
    FILE pocketfft-config.cmake
    NAMESPACE pocketfft::
    DESTINATION "share/pocketfft"
)

if(BUILD_TESTS)
    add_executable(ffttest ffttest.c)
    target_link_libraries(ffttest PRIVATE pocketfft)
endif()


# CMake package usage:
#     find_package(pocketfft REQUIRED)
#     add_executable(ffttest ffttest.c)
#     target_link_libraries(ffttest PRIVATE pocketfft::pocketfft)
