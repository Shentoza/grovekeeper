#
# Copyright (C) The Android Open Source Project
#
# Licensed under the Apache License, Version 2.0 (the "License");
# you may not use this file except in compliance with the License.
# You may obtain a copy of the License at
#
#      http://www.apache.org/licenses/LICENSE-2.0
#
# Unless required by applicable law or agreed to in writing, software
# distributed under the License is distributed on an "AS IS" BASIS,
# WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
# See the License for the specific language governing permissions and
# limitations under the License.
#

cmake_minimum_required(VERSION 3.4.1)

# build native_app_glue as a static lib
set(${CMAKE_C_FLAGS}, "${CMAKE_C_FLAGS}")
set(sfml_dir ${CMAKE_SOURCE_DIR}/../../../../sfml)
set(sfml_lib ${sfml_dir}/lib/${ANDROID_ABI})
set(sfml_include ${sfml_dir}/include)

include_directories(${sfml_include})

#--- ADD LIBS: prebuilt libraries (3rd party)
add_library(sfml-main STATIC IMPORTED)
set_target_properties(sfml-main PROPERTIES IMPORTED_LOCATION ${sfml_lib}/libsfml-main.a)

add_library(sfml-activity SHARED IMPORTED)
set_target_properties(sfml-activity PROPERTIES IMPORTED_LOCATION ${sfml_lib}/libsfml-activity.so)

add_library(sfml-system SHARED IMPORTED)
set_target_properties(sfml-system PROPERTIES IMPORTED_LOCATION ${sfml_lib}/libsfml-system.so)

add_library(sfml-window SHARED IMPORTED)
set_target_properties(sfml-window PROPERTIES IMPORTED_LOCATION ${sfml_lib}/libsfml-window.so)

add_library(sfml-graphics SHARED IMPORTED)
set_target_properties(sfml-graphics PROPERTIES IMPORTED_LOCATION ${sfml_lib}/libsfml-graphics.so)

add_library(sfml-audio SHARED IMPORTED)
set_target_properties(sfml-audio PROPERTIES IMPORTED_LOCATION ${sfml_lib}/libsfml-audio.so)

add_library(openal SHARED IMPORTED)
set_target_properties(openal PROPERTIES IMPORTED_LOCATION ${sfml_lib}/libopenal.so)

add_library(native_app_glue STATIC
    ${ANDROID_NDK}/sources/android/native_app_glue/android_native_app_glue.c)

# now build app's shared lib
set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -std=gnu++17 -Wall -Werror")

# Export ANativeActivity_onCreate(),
# Refer to: https://github.com/android-ndk/ndk/issues/381.
set(CMAKE_SHARED_LINKER_FLAGS
    "${CMAKE_SHARED_LINKER_FLAGS} -u ANativeActivity_onCreate")

add_library(com.shentoza.grovekeeper SHARED main.cpp)

target_include_directories(com.shentoza.grovekeeper PRIVATE
    ${sfml_include})

# add lib dependencies
target_link_libraries(com.shentoza.grovekeeper # specifies the target lib
        # link the target libs

        custom_native_app_glue
        sfml-main

        android
        log

        EGL
        GLESv1_CM

        openal
        sfml-audio

        sfml-system
        sfml-window
        sfml-graphics
        sfml-activity

        # neat one, thanks Alia5
        -Wl,--whole-archive sfml-main -Wl,--no-whole-archive
        )