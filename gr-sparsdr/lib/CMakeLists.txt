# Copyright 2011,2012,2016 Free Software Foundation, Inc.
#
# This file is a part of gr-sparsdr
#
# GNU Radio is free software; you can redistribute it and/or modify
# it under the terms of the GNU General Public License as published by
# the Free Software Foundation; either version 3, or (at your option)
# any later version.
#
# GNU Radio is distributed in the hope that it will be useful,
# but WITHOUT ANY WARRANTY; without even the implied warranty of
# MERCHANTABILITY or FITNESS FOR A PARTICULAR PURPOSE.  See the
# GNU General Public License for more details.
#
# You should have received a copy of the GNU General Public License
# along with GNU Radio; see the file COPYING.  If not, write to
# the Free Software Foundation, Inc., 51 Franklin Street,
# Boston, MA 02110-1301, USA.

########################################################################
# Setup library
########################################################################
include(GrPlatform) #define LIB_SUFFIX

list(APPEND sparsdr_sources
    average_detector_impl.cc
    real_time_receiver_impl.cc
    multi_sniffer_impl.cc
    reconstruct_impl.cc
    compressing_usrp_source_impl.cc
    gui/average_waterfall_impl.cc
    gui/stream_average_model.cc
    gui/average_model.cpp
    gui/average_waterfall_view.cpp
    sample_distributor_impl.cc
)

set(sparsdr_sources "${sparsdr_sources}" PARENT_SCOPE)
if(NOT sparsdr_sources)
    MESSAGE(STATUS "No C++ sources... skipping lib/")
    return()
endif(NOT sparsdr_sources)

add_library(gnuradio-sparsdr SHARED ${sparsdr_sources})
target_link_libraries(gnuradio-sparsdr
    gnuradio::gnuradio-runtime
    gnuradio::gnuradio-blocks
    gnuradio::gnuradio-fft
    gnuradio::gnuradio-filter
    gnuradio::gnuradio-uhd
    UHD::UHD
    Python::Python	
    ${Boost_LIBRARIES}
    Qt5::Widgets
)
target_include_directories(gnuradio-sparsdr
    PUBLIC ${Boost_INCLUDE_DIR}
    PUBLIC ${PYTHON_INCLUDE_DIRS}
    PUBLIC $<BUILD_INTERFACE:${CMAKE_CURRENT_SOURCE_DIR}/../include>
    PUBLIC $<INSTALL_INTERFACE:include>
  )
set_target_properties(gnuradio-sparsdr PROPERTIES DEFINE_SYMBOL "gnuradio_sparsdr_EXPORTS")

if(APPLE)
    set_target_properties(gnuradio-sparsdr PROPERTIES
        INSTALL_NAME_DIR "${CMAKE_INSTALL_PREFIX}/lib"
    )
endif(APPLE)

########################################################################
# Install built library files
########################################################################
include(GrMiscUtils)
GR_LIBRARY_FOO(gnuradio-sparsdr)

########################################################################
# Print summary
########################################################################
message(STATUS "Using install prefix: ${CMAKE_INSTALL_PREFIX}")
message(STATUS "Building for version: ${VERSION} / ${LIBVER}")

########################################################################
# Build and register unit test
########################################################################
include(GrTest)

# If your unit tests require special include paths, add them here
#include_directories()
# List all files that contain Boost.UTF unit tests here
list(APPEND test_sparsdr_sources
)
# Anything we need to link to for the unit tests go here
list(APPEND GR_TEST_TARGET_DEPS gnuradio-sparsdr)

if(NOT test_sparsdr_sources)
    MESSAGE(STATUS "No C++ unit tests... skipping")
    return()
endif(NOT test_sparsdr_sources)

foreach(qa_file ${test_sparsdr_sources})
    GR_ADD_CPP_TEST("sparsdr_${qa_file}"
        ${CMAKE_CURRENT_SOURCE_DIR}/${qa_file}
    )
endforeach(qa_file)
