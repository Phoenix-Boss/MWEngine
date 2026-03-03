# The MIT License (MIT)
#
# Copyright (c) 2013-2022 Igor Zinken - https://www.igorski.nl
#
# Permission is hereby granted, free of charge, to any person obtaining a copy of
# this software and associated documentation files (the "Software"), to deal in
# the Software without restriction, including without limitation the rights to
# use, copy, modify, merge, publish, distribute, sublicense, and/or sell copies of
# the Software, and to permit persons to whom the Software is furnished to do so,
# subject to the following conditions:
#
# The above copyright notice and this permission notice shall be included in all
# copies or substantial portions of the Software.
#
# THE SOFTWARE IS PROVIDED "AS IS", WITHOUT WARRANTY OF ANY KIND, EXPRESS OR
# IMPLIED, INCLUDING BUT NOT LIMITED TO THE WARRANTIES OF MERCHANTABILITY, FITNESS
# FOR A PARTICULAR PURPOSE AND NONINFRINGEMENT. IN NO EVENT SHALL THE AUTHORS OR
# COPYRIGHT HOLDERS BE LIABLE FOR ANY CLAIM, DAMAGES OR OTHER LIABILITY, WHETHER
# IN AN ACTION OF CONTRACT, TORT OR OTHERWISE, ARISING FROM, OUT OF OR IN
# CONNECTION WITH THE SOFTWARE OR THE USE OR OTHER DEALINGS IN THE SOFTWARE.
#
cmake_minimum_required(VERSION 3.22.1)

# policies for SWIG (EAS compatible)
cmake_policy(SET CMP0086 NEW)
cmake_policy(SET CMP0078 NEW)

Project(MWEngine)
set(target mwengine)

# architecture-specific compiler flags
if (${ANDROID_ABI} MATCHES "x86_64")
    SET(CMAKE_C_FLAGS "${CMAKE_C_FLAGS} -march=x86-64 -msse4.2 -mpopcnt -m64")
endif()

# source folders
set(CPP_SRC  ${CMAKE_CURRENT_SOURCE_DIR}/src/main/cpp)
set(JAVA_SRC ${CMAKE_CURRENT_SOURCE_DIR}/src/main/java)

# Include location of the header files
include_directories(${CPP_SRC}/)

# core engine
set(MWENGINE_CORE_SOURCES ${CPP_SRC}/global.cpp
                          ${CPP_SRC}/audioengine.cpp
                          ${CPP_SRC}/audiobuffer.cpp
                          ${CPP_SRC}/audiochannel.cpp
                          ${CPP_SRC}/channelgroup.cpp
                          ${CPP_SRC}/processingchain.cpp
                          ${CPP_SRC}/resizable_audiobuffer.cpp
                          ${CPP_SRC}/resizable_buffergroup.cpp
                          ${CPP_SRC}/ringbuffer.cpp
                          ${CPP_SRC}/sequencer.cpp
                          ${CPP_SRC}/sequencercontroller.cpp
                          ${CPP_SRC}/wavetable.cpp
                          ${CPP_SRC}/definitions/libraries.cpp
                          ${CPP_SRC}/drivers/adapter.cpp
                          ${CPP_SRC}/drivers/aaudio_io.cpp
                          ${CPP_SRC}/drivers/opensl_io.c
                          ${CPP_SRC}/events/baseaudioevent.cpp
                          ${CPP_SRC}/events/basecacheableaudioevent.cpp
                          ${CPP_SRC}/events/sampleevent.cpp
                          ${CPP_SRC}/generators/envelopegenerator.cpp
                          ${CPP_SRC}/generators/wavegenerator.cpp
                          ${CPP_SRC}/instruments/baseinstrument.cpp
                          ${CPP_SRC}/instruments/sampledinstrument.cpp
                          ${CPP_SRC}/messaging/notifier.cpp
                          ${CPP_SRC}/messaging/observer.cpp
                          ${CPP_SRC}/modules/envelopefollower.cpp
                          ${CPP_SRC}/modules/lfo.cpp
                          ${CPP_SRC}/modules/routeableoscillator.cpp
                          ${CPP_SRC}/processors/baseprocessor.cpp
                          ${CPP_SRC}/services/library_loader.cpp
                          ${CPP_SRC}/utilities/audiorenderer.cpp
                          ${CPP_SRC}/utilities/bufferutility.cpp
                          ${CPP_SRC}/utilities/levelutility.cpp
                          ${CPP_SRC}/utilities/bulkcacher.cpp
                          ${CPP_SRC}/utilities/diskwriter.cpp
                          ${CPP_SRC}/utilities/debug.cpp
                          ${CPP_SRC}/utilities/samplemanager.cpp
                          ${CPP_SRC}/utilities/bufferpool.cpp
                          ${CPP_SRC}/utilities/tablepool.cpp
                          ${CPP_SRC}/utilities/fastmath.cpp
                          ${CPP_SRC}/utilities/wavereader.cpp
                          ${CPP_SRC}/utilities/wavewriter.cpp
                          ${CPP_SRC}/utilities/utils.cpp)

# Mikrowave specific sources (drum machine specific, you can also create drum machines using sampledinstrument)
set(MWENGINE_MIKROWAVE_SOURCES ${CPP_SRC}/drumpattern.cpp
                               ${CPP_SRC}/events/drumevent.cpp
                               ${CPP_SRC}/instruments/druminstrument.cpp)

# synthesis (can be omitted if your use case only concerns sample based playback)
set(MWENGINE_SYNTH_SOURCES ${CPP_SRC}/definitions/pitch.cpp
                           ${CPP_SRC}/events/basesynthevent.cpp
                           ${CPP_SRC}/events/synthevent.cpp
                           ${CPP_SRC}/modules/adsr.cpp
                           ${CPP_SRC}/modules/arpeggiator.cpp
                           ${CPP_SRC}/instruments/oscillatorproperties.cpp
                           ${CPP_SRC}/instruments/synthinstrument.cpp
                           ${CPP_SRC}/generators/synthesizer.cpp)

# effects processors (can be omitted if your use case only concerns raw audio)
set(MWENGINE_PROCESSORS ${CPP_SRC}/processors/basedynamicsprocessor.cpp
                        ${CPP_SRC}/processors/bitcrusher.cpp
                        ${CPP_SRC}/processors/compressor.cpp
                        ${CPP_SRC}/processors/dcoffsetfilter.cpp
                        ${CPP_SRC}/processors/decimator.cpp
                        ${CPP_SRC}/processors/delay.cpp
                        ${CPP_SRC}/processors/filter.cpp
                        ${CPP_SRC}/processors/flanger.cpp
                        ${CPP_SRC}/processors/fm.cpp
                        ${CPP_SRC}/processors/formantfilter.cpp
                        ${CPP_SRC}/processors/gain.cpp
                        ${CPP_SRC}/processors/gate.cpp
                        ${CPP_SRC}/processors/glitcher.cpp
                        ${CPP_SRC}/processors/limiter.cpp
                        ${CPP_SRC}/processors/lowpassfilter.cpp
                        ${CPP_SRC}/processors/lpfhpfilter.cpp
                        ${CPP_SRC}/processors/phaser.cpp
                        ${CPP_SRC}/processors/pitchshifter.cpp
                        ${CPP_SRC}/processors/reverb.cpp
                        ${CPP_SRC}/processors/reverbsm.cpp
                        ${CPP_SRC}/processors/tremolo.cpp
                        ${CPP_SRC}/processors/waveshaper.cpp)

# when using the library solely from C++ you can omit bundling JNI_SOURCES (or SWIG wrapping)
set(JNI_SOURCES ${CPP_SRC}/jni/javabridge.cpp
                ${CPP_SRC}/jni/javautilities.cpp)

# build the mock driver into the library when unit test execution is enabled
if (${RUN_TESTS} MATCHES "true")
    SET(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -DMOCK_ENGINE")
    set(MWENGINE_CORE_SOURCES ${MWENGINE_CORE_SOURCES}
                              ${CPP_SRC}/drivers/mock_io.cpp)
endif()

# include the AAudio library into the MWEngine compilation when configured
if (${INCLUDE_AAUDIO_LIBRARY} MATCHES "true")
    SET(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -DINCLUDE_AAUDIO_LIBRARY")
endif()

# Build SHARED library for Expo/EAS compatibility (instead of STATIC)
add_library(${target} SHARED
                     ${MWENGINE_CORE_SOURCES}
                     ${MWENGINE_SYNTH_SOURCES}
                     ${MWENGINE_PROCESSORS}
                     ${MWENGINE_MIKROWAVE_SOURCES}
                     ${JNI_SOURCES})

# Compiler optimizations for audio processing
target_compile_options(${target} PRIVATE 
    -O3 
    -ffast-math 
    -funsafe-math-optimizations
)

# link all the NDK libraries used by the project
find_library(log-lib log)
find_library(android-lib android)
find_library(atomic-lib atomic)
find_library(opensles-lib OpenSLES)
find_library(aaudio-lib aaudio)

target_link_libraries(${target} 
    ${android-lib} 
    ${atomic-lib} 
    ${log-lib} 
    ${opensles-lib}
)

# if AAudio library inclusion is configured in the build, link it here
if (${INCLUDE_AAUDIO_LIBRARY} MATCHES "true")
    target_link_libraries(${target} PRIVATE ${aaudio-lib})
endif()

#####################################
## SWIG JNI to Java class wrapping ##
#####################################

# Make SWIG COMPLETELY OPTIONAL for EAS servers
find_package(SWIG QUIET)
if(SWIG_FOUND)
    message(STATUS "SWIG found - generating Java bindings")
    include(${SWIG_USE_FILE})

    set_property(SOURCE ${CPP_SRC}/mwengine.i PROPERTY CPLUSPLUS ON)
    set(CMAKE_SWIG_FLAGS -DNO_RTIO -c++ -lcarrays.i -package nl.igorski.mwengine.core)
    set(CMAKE_SWIG_OUTDIR ${JAVA_SRC}/nl/igorski/mwengine/core)

    # Export a wrapper file to Java, and link with the created C++ library
    swig_add_library(${target}_wrapped LANGUAGE java SOURCES ${CPP_SRC}/mwengine.i)
    swig_link_libraries(${target}_wrapped ${target})
else()
    message(STATUS "SWIG not found - skipping Java bindings (using direct JNI)")
endif()

################
## UNIT TESTS ##
################

if (${RUN_TESTS} MATCHES "true")
    # only run for a single architecture, assume x86_64
    if (${ANDROID_ABI} MATCHES "x86_64")
        set(GTEST_DIR ${ANDROID_NDK}/sources/third_party/googletest)

        add_library(gtest STATIC 
            ${GTEST_DIR}/src/gtest_main.cc 
            ${GTEST_DIR}/src/gtest-all.cc
        )
        target_include_directories(gtest PRIVATE ${GTEST_DIR})
        target_include_directories(gtest PUBLIC ${GTEST_DIR}/include)

        add_executable(${target}_unittest ${CPP_SRC}/tests/main.cpp)
        target_link_libraries(${target}_unittest ${target} gtest)

        # Push and execute tests as post-build event (disabled for EAS)
        set(TARGET_TEST_DIR /data/local/tmp/${target}_unittest_${ANDROID_ABI})
        add_custom_command(
            TARGET ${target}_unittest POST_BUILD
            COMMENT "Running unit tests requires device/emulator - skipping for EAS"
        )
    endif()
endif()
