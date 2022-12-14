#/****************************************************************************
# Copyright (c) 2013-2014 cocos2d-x.org
# Copyright (c) 2021-2022 Bytedance Inc.
#
# https://axis-project.github.io
#
# Permission is hereby granted, free of charge, to any person obtaining a copy
# of this software and associated documentation files (the "Software"), to deal
# in the Software without restriction, including without limitation the rights
# to use, copy, modify, merge, publish, distribute, sublicense, and/or sell
# copies of the Software, and to permit persons to whom the Software is
# furnished to do so, subject to the following conditions:

# The above copyright notice and this permission notice shall be included in
# all copies or substantial portions of the Software.

# THE SOFTWARE IS PROVIDED "AS IS", WITHOUT WARRANTY OF ANY KIND, EXPRESS OR
# IMPLIED, INCLUDING BUT NOT LIMITED TO THE WARRANTIES OF MERCHANTABILITY,
# FITNESS FOR A PARTICULAR PURPOSE AND NONINFRINGEMENT. IN NO EVENT SHALL THE
# AUTHORS OR COPYRIGHT HOLDERS BE LIABLE FOR ANY CLAIM, DAMAGES OR OTHER
# LIABILITY, WHETHER IN AN ACTION OF CONTRACT, TORT OR OTHERWISE, ARISING FROM,
# OUT OF OR IN CONNECTION WITH THE SOFTWARE OR THE USE OR OTHER DEALINGS IN
# THE SOFTWARE.
# ****************************************************************************/

cmake_minimum_required(VERSION 3.20)

set(APP_NAME fish_shooter_lua)

project(${APP_NAME})

set(_AX_TOP_LEVEL_APP FALSE)
if(NOT DEFINED BUILD_ENGINE_DONE) # to test fish_shooter_lua into root project
    set(_AX_TOP_LEVEL_APP TRUE)
    if(XCODE)
        set(CMAKE_XCODE_GENERATE_TOP_LEVEL_PROJECT_ONLY TRUE)
    endif()
    
    # config quick starter batch script run.bat for windows
    if(WIN32)
        file(RELATIVE_PATH CMAKE_BUILD_RELATIVE_DIR "${CMAKE_CURRENT_SOURCE_DIR}" "${PROJECT_BINARY_DIR}")
        configure_file("${CMAKE_CURRENT_SOURCE_DIR}/run.bat.in" "${CMAKE_CURRENT_SOURCE_DIR}/run.bat" @ONLY)
    endif()

    set(AXIS_ROOT "$ENV{AXIS_ROOT}")

    if(NOT (AXIS_ROOT STREQUAL ""))
        set(AXIS_ROOT_PATH "${AXIS_ROOT}")
        file(TO_CMAKE_PATH ${AXIS_ROOT_PATH} AXIS_ROOT_PATH) # string(REPLACE "\\" "/" AXIS_ROOT_PATH ${AXIS_ROOT_PATH})
        message(STATUS "Using system env var AXIS_ROOT=${AXIS_ROOT}")
    else()
        message(FATAL_ERROR "Please run setup.py add system env var 'AXIS_ROOT' to specific the engine root")
    endif()

    set(CMAKE_MODULE_PATH ${AXIS_ROOT_PATH}/cmake/Modules/)

    include(AxisBuildSet)
    set(AX_ENABLE_EXT_LUA ON)
    
    set(_AX_USE_PREBUILT FALSE)
    if (WIN32 AND DEFINED AX_PREBUILT_DIR AND IS_DIRECTORY ${AXIS_ROOT_PATH}/${AX_PREBUILT_DIR})
        set(_AX_USE_PREBUILT TRUE)
    endif()

    if (NOT _AX_USE_PREBUILT)
        add_subdirectory(${AXIS_ROOT_PATH}/core ${ENGINE_BINARY_PATH}/axis/core)
    endif()
endif()

set(RUNTIME_SRC_ROOT ${CMAKE_CURRENT_SOURCE_DIR}/frameworks/runtime-src)

# script and source files info, not need to compile
set(res_res_folders
    "${CMAKE_CURRENT_SOURCE_DIR}/res"
    )
set(res_src_folders
    "${CMAKE_CURRENT_SOURCE_DIR}/src"
    )
set(res_script_folders
    "${AXIS_ROOT_PATH}/extensions/scripting/lua-bindings/script"
    )
if(APPLE OR VS)
    cocos_mark_multi_resources(res_res RES_TO "Resources/res" FOLDERS ${res_res_folders})
    cocos_mark_multi_resources(res_src RES_TO "Resources/src" FOLDERS ${res_src_folders})
    set(cc_common_res ${res_res} ${res_src})
    if (NOT _AX_TOP_LEVEL_APP)
        cocos_mark_multi_resources(res_script RES_TO "Resources/src/cocos" FOLDERS ${res_script_folders})
        set(cc_common_res ${cc_common_res} ${res_script})
    endif()
endif()

# record sources, headers
set(GAME_HEADER ${RUNTIME_SRC_ROOT}/Classes/AppDelegate.h)
set(GAME_SOURCE ${RUNTIME_SRC_ROOT}/Classes/AppDelegate.cpp)
# sources need to compile info, include dirs and source files

###############################################################################################
###############################################################################################
###############################################################################################

set(CMAKE_CXX_STANDARD 20)
include_directories(${AXIS_ROOT_PATH}/../boost/boost)
include_directories(${AXIS_ROOT_PATH}/../yylib/src)

###############################################################################################
###############################################################################################
###############################################################################################

if(ANDROID)
    # the APP_NAME should match on AndroidManifest.xml
    list(APPEND GAME_SOURCE ${RUNTIME_SRC_ROOT}/proj.android/app/jni/hellolua/main.cpp)
elseif(LINUX)
    list(APPEND GAME_SOURCE ${RUNTIME_SRC_ROOT}/proj.linux/main.cpp)
elseif(WINDOWS)
    list(APPEND GAME_HEADER
         ${RUNTIME_SRC_ROOT}/proj.win32/main.h
         )
    list(APPEND GAME_SOURCE
         ${RUNTIME_SRC_ROOT}/proj.win32/main.cpp
         ${cc_common_res}
         )
elseif(APPLE)
    if(IOS)
        list(APPEND GAME_HEADER
             ${RUNTIME_SRC_ROOT}/proj.ios_mac/ios/AppController.h
             ${RUNTIME_SRC_ROOT}/proj.ios_mac/ios/RootViewController.h
             )
        set(APP_UI_RES
            ${RUNTIME_SRC_ROOT}/proj.ios_mac/ios/LaunchScreen.storyboard
            ${RUNTIME_SRC_ROOT}/proj.ios_mac/ios/LaunchScreenBackground.png
            ${RUNTIME_SRC_ROOT}/proj.ios_mac/ios/Images.xcassets
            )
        list(APPEND GAME_SOURCE
             ${RUNTIME_SRC_ROOT}/proj.ios_mac/ios/main.m
             ${RUNTIME_SRC_ROOT}/proj.ios_mac/ios/AppController.mm
             ${RUNTIME_SRC_ROOT}/proj.ios_mac/ios/RootViewController.mm
             ${RUNTIME_SRC_ROOT}/proj.ios_mac/ios/Prefix.pch
             ${RUNTIME_SRC_ROOT}/proj.ios_mac/ios/build-cfg.json
             ${APP_UI_RES}
             )
    elseif(MACOSX)
        list(APPEND GAME_HEADER
             )
        set(APP_UI_RES
            ${RUNTIME_SRC_ROOT}/proj.ios_mac/mac/Icon.icns
            ${RUNTIME_SRC_ROOT}/proj.ios_mac/mac/Info.plist
            )
        list(APPEND GAME_SOURCE
             ${RUNTIME_SRC_ROOT}/proj.ios_mac/mac/main.cpp
             ${RUNTIME_SRC_ROOT}/proj.ios_mac/mac/Prefix.pch
             ${APP_UI_RES}
             )
    endif()
    list(APPEND GAME_SOURCE ${cc_common_res})
endif()

set(APP_SRC ${GAME_HEADER} ${GAME_SOURCE})

# mark app complie info and libs info
if(NOT ANDROID)
    add_executable(${APP_NAME} ${APP_SRC})
else()
    add_library(${APP_NAME} SHARED ${APP_SRC})
    
    # whole archive for jni
    add_subdirectory(${AXIS_ROOT_PATH}/core/platform/android ${ENGINE_BINARY_PATH}/core/platform)
    target_link_libraries(${APP_NAME} -Wl,--whole-archive cpp_android_spec -Wl,--no-whole-archive)
    add_subdirectory(${AXIS_ROOT_PATH}/extensions/scripting/lua-bindings/proj.android ${ENGINE_BINARY_PATH}/extensions/lua-android)
    target_link_libraries(${APP_NAME} -Wl,--whole-archive lua_android_spec -Wl,--no-whole-archive)

    config_android_shared_libs("org.cocos2dx.lib" "${CMAKE_CURRENT_SOURCE_DIR}/frameworks/runtime-src/proj.android/app/src")
endif()

if (NOT _AX_USE_PREBUILT)
    target_link_libraries(${APP_NAME} ${AXIS_LUA_LIB})
endif()

target_include_directories(${APP_NAME} PRIVATE ${RUNTIME_SRC_ROOT}/Classes)

# mark app resources, resource will be copy auto after mark
setup_cocos_app_config(${APP_NAME})
if(APPLE)
    set_target_properties(${APP_NAME} PROPERTIES RESOURCE "${APP_UI_RES}")
    set_xcode_property(${APP_NAME} INSTALL_PATH "\$(LOCAL_APPS_DIR)")
    set_xcode_property(${APP_NAME} PRODUCT_BUNDLE_IDENTIFIER "org.axis.hellolua")
    if(MACOSX)
        set_target_properties(${APP_NAME} PROPERTIES
                              MACOSX_BUNDLE_INFO_PLIST "${RUNTIME_SRC_ROOT}/proj.ios_mac/mac/Info.plist"
                              )
        set_target_properties(${APP_NAME} PROPERTIES
            LINK_FLAGS "-pagezero_size 10000 -image_base 100000000"
        )
    elseif(IOS)
        set_target_properties(${APP_NAME} PROPERTIES MACOSX_BUNDLE_INFO_PLIST "${RUNTIME_SRC_ROOT}/proj.ios_mac/ios/Info.plist")
        set_xcode_property(${APP_NAME} ASSETCATALOG_COMPILER_APPICON_NAME "AppIcon")
        set_xcode_property(${APP_NAME} DEVELOPMENT_TEAM "")
        set_xcode_property(${APP_NAME} CODE_SIGN_IDENTITY "iPhone Developer")
    endif()
elseif(WINDOWS)
    if(NOT _AX_USE_PREBUILT)
        cocos_copy_target_dll(${APP_NAME})
        cocos_copy_lua_dlls(${APP_NAME})
    endif()
endif()
# copy resource on linux or WINDOWS
if((WINDOWS AND (CMAKE_GENERATOR STREQUAL "Ninja")) OR LINUX)
    cocos_get_resource_path(APP_RES_DIR ${APP_NAME})
    cocos_copy_target_res(${APP_NAME} LINK_TO ${APP_RES_DIR}/res FOLDERS ${res_res_folders})
    cocos_copy_lua_scripts(${APP_NAME} ${res_src_folders} ${APP_RES_DIR}/src )
    cocos_copy_lua_scripts(${APP_NAME} ${res_script_folders} ${APP_RES_DIR}/src/cocos)
elseif(WINDOWS)
    set(my_res_folder "${CMAKE_CURRENT_SOURCE_DIR}")
    cocos_copy_lua_scripts(${APP_NAME} ${res_script_folders} "${my_res_folder}/src/cocos")
    set_property(TARGET ${APP_NAME} PROPERTY VS_DEBUGGER_WORKING_DIRECTORY "${my_res_folder}")
    if(NOT DEFINED BUILD_ENGINE_DONE)
        set_property(DIRECTORY PROPERTY VS_STARTUP_PROJECT ${APP_NAME})
    endif()
endif()

# The optional thirdparties(not dependent by engine)
if (AX_WITH_YAML_CPP)
    target_include_directories(${APP_NAME} PRIVATE ${AXIS_ROOT_PATH}/thirdparty/yaml-cpp/include)
    target_link_libraries(${APP_NAME} yaml-cpp)
endif()

if (_AX_USE_PREBUILT) # support windows only
    include(${AXIS_ROOT_PATH}/cmake/Modules/AxisConfigDefine.cmake)
    use_axis_compile_define(${APP_NAME})

    include(${AXIS_ROOT_PATH}/cmake/Modules/AxisLinkHelpers.cmake)
    axis_link_lua_prebuilt(${APP_NAME} ${AXIS_ROOT_PATH} ${AX_PREBUILT_DIR})
endif()
