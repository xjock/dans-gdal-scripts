cmake_minimum_required(VERSION 3.16)
project(dans-gdal-scripts VERSION 0.24 LANGUAGES CXX)

set(CMAKE_CXX_STANDARD 11)
set(CMAKE_CXX_STANDARD_REQUIRED ON)

# Compiler flags matching the original autotools build
add_compile_options(-Wall -Wextra -O3 -g)

# Find Boost (header-only usage: foreach, lexical_cast, format)
find_package(Boost REQUIRED)

# Find GDAL via gdal-config (original configure.ac logic)
find_program(GDAL_CONFIG gdal-config)
if(NOT GDAL_CONFIG)
    message(FATAL_ERROR "gdal-config not found. Please install GDAL development files.")
endif()

execute_process(
    COMMAND ${GDAL_CONFIG} --cflags
    OUTPUT_VARIABLE GDAL_CFLAGS
    OUTPUT_STRIP_TRAILING_WHITESPACE
)
execute_process(
    COMMAND ${GDAL_CONFIG} --libs
    OUTPUT_VARIABLE GDAL_RAW_LIBS
    OUTPUT_STRIP_TRAILING_WHITESPACE
)

# Replicate configure.ac logic: if gdal-config --libs returns a libtool .la,
# rebuild proper -L/-l flags from libdir= inside the .la file.
if(GDAL_RAW_LIBS MATCHES "\\.la")
    execute_process(
        COMMAND ${GDAL_CONFIG} --dep-libs
        OUTPUT_VARIABLE GDAL_DEP_LIBS
        OUTPUT_STRIP_TRAILING_WHITESPACE
    )
    # Extract the .la path and read libdir= from it
    string(REGEX MATCH "[^ ]+\\.la" GDAL_LA_FILE "${GDAL_RAW_LIBS}")
    if(GDAL_LA_FILE AND EXISTS ${GDAL_LA_FILE})
        file(STRINGS ${GDAL_LA_FILE} GDAL_LIBDIR_LINE REGEX "^libdir='")
        string(REGEX REPLACE "^libdir='(.+)'$" "\\1" GDAL_LIBDIR "${GDAL_LIBDIR_LINE}")
        set(GDAL_LINK_FLAGS "-Wl,--no-as-needed -L${GDAL_LIBDIR} -lgdal ${GDAL_DEP_LIBS}")
        message(STATUS "gdal-config returned libtool .la; using -L${GDAL_LIBDIR} -lgdal plus --dep-libs")
    else()
        set(GDAL_LINK_FLAGS "${GDAL_RAW_LIBS}")
    endif()
else()
    set(GDAL_LINK_FLAGS "${GDAL_RAW_LIBS}")
endif()

# Convert gdal-config --cflags into include directories and defines
separate_arguments(GDAL_CFLAGS_LIST NATIVE_COMMAND "${GDAL_CFLAGS}")
set(GDAL_INCLUDE_DIRS "")
set(GDAL_DEFINES "")
foreach(flag ${GDAL_CFLAGS_LIST})
    if(flag MATCHES "^-I(.+)$")
        list(APPEND GDAL_INCLUDE_DIRS "${CMAKE_MATCH_1}")
    elseif(flag MATCHES "^-D(.+)$")
        list(APPEND GDAL_DEFINES "${CMAKE_MATCH_1}")
    endif()
endforeach()

# Verify key GDAL headers are reachable
include(CheckIncludeFileCXX)
set(CMAKE_REQUIRED_INCLUDES ${GDAL_INCLUDE_DIRS})
check_include_file_cxx("gdal.h" HAVE_GDAL_H)
check_include_file_cxx("cpl_conv.h" HAVE_CPL_CONV_H)
check_include_file_cxx("cpl_port.h" HAVE_CPL_PORT_H)
check_include_file_cxx("cpl_string.h" HAVE_CPL_STRING_H)
check_include_file_cxx("ogr_api.h" HAVE_OGR_API_H)
check_include_file_cxx("ogr_spatialref.h" HAVE_OGR_SPATIALREF_H)
check_include_file_cxx("ogrsf_frmts.h" HAVE_OGRSF_FRMTS_H)
if(NOT HAVE_GDAL_H)
    message(FATAL_ERROR "Could not find GDAL header files")
endif()

# Generate a minimal config.h replacement
configure_file(
    ${CMAKE_CURRENT_SOURCE_DIR}/cmake/config.h.in
    ${CMAKE_CURRENT_BINARY_DIR}/config.h
)

# Common source files used by multiple targets
set(COMMON_SOURCES src/common.cc)

add_executable(gdal_raw2geotiff src/gdal_raw2geotiff.cc ${COMMON_SOURCES})

add_executable(gdal_dem2rgb
    src/gdal_dem2rgb.cc ${COMMON_SOURCES}
    src/georef.cc src/ndv.cc src/palette.cc src/datatype_conversion.cc
)

add_executable(gdal_list_corners
    src/gdal_list_corners.cc ${COMMON_SOURCES}
    src/polygon.cc src/polygon-rasterizer.cc src/debugplot.cc
    src/georef.cc src/mask.cc src/rectangle_finder.cc
    src/ndv.cc src/datatype_conversion.cc
)

add_executable(gdal_trace_outline
    src/gdal_trace_outline.cc ${COMMON_SOURCES}
    src/polygon.cc src/polygon-rasterizer.cc src/debugplot.cc
    src/georef.cc src/mask.cc src/mask-tracer.cc src/beveler.cc
    src/dp.cc src/ndv.cc src/excursion_pincher2.cc
    src/raster_features.cc src/datatype_conversion.cc
)

add_executable(gdal_contrast_stretch
    src/gdal_contrast_stretch.cc ${COMMON_SOURCES}
    src/ndv.cc src/datatype_conversion.cc
)

add_executable(gdal_landsat_pansharp
    src/gdal_landsat_pansharp.cc ${COMMON_SOURCES}
)

add_executable(gdal_wkt_to_mask
    src/gdal_wkt_to_mask.cc ${COMMON_SOURCES}
    src/polygon.cc src/polygon-rasterizer.cc src/georef.cc
)

add_executable(gdal_get_projected_bounds
    src/gdal_get_projected_bounds.cc ${COMMON_SOURCES}
    src/polygon.cc src/georef.cc src/debugplot.cc
)

add_executable(gdal_merge_simple
    src/gdal_merge_simple.cc ${COMMON_SOURCES}
)

add_executable(gdal_merge_vrt
    src/gdal_merge_vrt.cc ${COMMON_SOURCES}
)

add_executable(gdal_make_ndv_mask
    src/gdal_make_ndv_mask.cc ${COMMON_SOURCES}
    src/ndv.cc src/mask.cc src/debugplot.cc src/datatype_conversion.cc
)

set(TARGETS
    gdal_raw2geotiff gdal_dem2rgb gdal_list_corners gdal_trace_outline
    gdal_contrast_stretch gdal_landsat_pansharp gdal_wkt_to_mask
    gdal_get_projected_bounds gdal_merge_simple gdal_merge_vrt
    gdal_make_ndv_mask
)

foreach(target ${TARGETS})
    target_include_directories(${target} PRIVATE
        ${CMAKE_CURRENT_BINARY_DIR}
        ${GDAL_INCLUDE_DIRS}
        ${Boost_INCLUDE_DIRS}
    )
    target_compile_definitions(${target} PRIVATE ${GDAL_DEFINES})
    # Parse GDAL_LINK_FLAGS into separate library/linker options
    separate_arguments(GDAL_LINK_FLAGS_LIST NATIVE_COMMAND "${GDAL_LINK_FLAGS}")
    target_link_options(${target} PRIVATE ${GDAL_LINK_FLAGS_LIST})
endforeach()

# Tests
enable_testing()
add_test(NAME test1 COMMAND bash test1.sh WORKING_DIRECTORY ${CMAKE_CURRENT_SOURCE_DIR}/src/tests)
add_test(NAME test2 COMMAND bash test2.sh WORKING_DIRECTORY ${CMAKE_CURRENT_SOURCE_DIR}/src/tests)
add_test(NAME test3 COMMAND bash test3.sh WORKING_DIRECTORY ${CMAKE_CURRENT_SOURCE_DIR}/src/tests)
set_tests_properties(test1 test2 test3 PROPERTIES ENVIRONMENT "PATH=${CMAKE_CURRENT_BINARY_DIR}/src:$ENV{PATH}")

# Install
install(TARGETS ${TARGETS} DESTINATION bin)
