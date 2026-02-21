# This file is part of yt-media-storage, a tool for encoding media.
# Copyright (C) Brandon Li <https://brandonli.me/>
#
# This program is free software: you can redistribute it and/or modify
# it under the terms of the GNU General Public License as published by
# the Free Software Foundation, either version 3 of the License, or
# (at your option) any later version.
#
# This program is distributed in the hope that it will be useful,
# but WITHOUT ANY WARRANTY; without even the implied warranty of
# MERCHANTABILITY or FITNESS FOR A PARTICULAR PURPOSE.  See the
# GNU General Public License for more details.
#
# You should have received a copy of the GNU General Public License
# along with this program.  If not, see <https://www.gnu.org/licenses/>.

cmake_minimum_required(VERSION 3.22)
project(media_storage LANGUAGES CXX)

set(CMAKE_CXX_STANDARD 23)
set(CMAKE_CXX_STANDARD_REQUIRED ON)

find_package(OpenMP REQUIRED)
find_package(Qt6 REQUIRED COMPONENTS Core Widgets)

# Enable Qt MOC, UIC, and RCC for GUI only
set(CMAKE_AUTOMOC OFF)
set(CMAKE_AUTOUIC OFF)
set(CMAKE_AUTORCC OFF)

# Find FFmpeg components
find_path(AVCODEC_INCLUDE_DIR libavcodec/avcodec.h)
find_library(AVCODEC_LIBRARY avcodec)
find_path(AVFORMAT_INCLUDE_DIR libavformat/avformat.h)
find_library(AVFORMAT_LIBRARY avformat)
find_path(AVUTIL_INCLUDE_DIR libavutil/avutil.h)
find_library(AVUTIL_LIBRARY avutil)
find_path(SWSCALE_INCLUDE_DIR libswscale/swscale.h)
find_library(SWSCALE_LIBRARY swscale)
find_path(SWRESAMPLE_INCLUDE_DIR libswresample/swresample.h)
find_library(SWRESAMPLE_LIBRARY swresample)

# Find libsodium
find_path(SODIUM_INCLUDE_DIR sodium.h)
find_library(SODIUM_LIBRARY libsodium)

# Create imported targets to mimic PkgConfig targets
add_library(PkgConfig::AVCODEC INTERFACE IMPORTED)
target_include_directories(PkgConfig::AVCODEC INTERFACE ${AVCODEC_INCLUDE_DIR})
target_link_libraries(PkgConfig::AVCODEC INTERFACE ${AVCODEC_LIBRARY})

add_library(PkgConfig::AVFORMAT INTERFACE IMPORTED)
target_include_directories(PkgConfig::AVFORMAT INTERFACE ${AVFORMAT_INCLUDE_DIR})
target_link_libraries(PkgConfig::AVFORMAT INTERFACE ${AVFORMAT_LIBRARY})

add_library(PkgConfig::AVUTIL INTERFACE IMPORTED)
target_include_directories(PkgConfig::AVUTIL INTERFACE ${AVUTIL_INCLUDE_DIR})
target_link_libraries(PkgConfig::AVUTIL INTERFACE ${AVUTIL_LIBRARY})

add_library(PkgConfig::SWSCALE INTERFACE IMPORTED)
target_include_directories(PkgConfig::SWSCALE INTERFACE ${SWSCALE_INCLUDE_DIR})
target_link_libraries(PkgConfig::SWSCALE INTERFACE ${SWSCALE_LIBRARY})

add_library(PkgConfig::SWRESAMPLE INTERFACE IMPORTED)
target_include_directories(PkgConfig::SWRESAMPLE INTERFACE ${SWRESAMPLE_INCLUDE_DIR})
target_link_libraries(PkgConfig::SWRESAMPLE INTERFACE ${SWRESAMPLE_LIBRARY})

add_library(PkgConfig::SODIUM INTERFACE IMPORTED)
target_include_directories(PkgConfig::SODIUM INTERFACE ${SODIUM_INCLUDE_DIR})
target_link_libraries(PkgConfig::SODIUM INTERFACE ${SODIUM_LIBRARY})

add_executable(media_storage)
add_executable(media_storage_gui)

add_subdirectory(src)

target_link_libraries(media_storage PRIVATE
        PkgConfig::AVCODEC
        PkgConfig::AVFORMAT
        PkgConfig::AVUTIL
        PkgConfig::SWSCALE
        PkgConfig::SWRESAMPLE
        PkgConfig::SODIUM
        OpenMP::OpenMP_CXX
)

target_link_libraries(media_storage_gui PRIVATE
        PkgConfig::AVCODEC
        PkgConfig::AVFORMAT
        PkgConfig::AVUTIL
        PkgConfig::SWSCALE
        PkgConfig::SWRESAMPLE
        PkgConfig::SODIUM
        OpenMP::OpenMP_CXX
        Qt6::Core
        Qt6::Widgets
)

# Enable Qt MOC only for GUI target
set_target_properties(media_storage_gui PROPERTIES AUTOMOC ON AUTOUIC ON)

# Check for OpenMP
if (MSVC)
    target_compile_options(media_storage PRIVATE
            $<$<CONFIG:Release>:/O2>
            /arch:AVX2
    )
    target_compile_options(media_storage_gui PRIVATE
            $<$<CONFIG:Release>:/O2>
            /arch:AVX2
    )
else ()
    target_compile_options(media_storage PRIVATE -march=native)
    if (CMAKE_SYSTEM_PROCESSOR MATCHES "x86_64|AMD64|i386")
        target_compile_options(media_storage_gui PRIVATE -O2 -mssse3)
    else ()
        target_compile_options(media_storage_gui PRIVATE -O2 -march=native)
    endif ()
endif ()
