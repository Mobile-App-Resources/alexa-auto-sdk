cmake_minimum_required(VERSION 3.6)

set(CMAKE_CXX_STANDARD 11)

project(AACECarControlAndroid LANGUAGES CXX)

set(AAC_HOME ${ANDROID_PLATFORM_DIR}/native-build/${ANDROID_ABI}-${ANDROID_PLATFORM}/opt/AAC)
set(CORE_MODULE_DIR ${ANDROID_PLATFORM_DIR}/modules/core)
string(TOLOWER ${CMAKE_BUILD_TYPE} NDK_BUILD_TYPE)

if(NOT EXISTS ${AAC_HOME})
    message(FATAL_ERROR "AAC build for ${ANDROID_ABI}/${ANDROID_PLATFORM} ${AAC_HOME} does not exist!")
endif()

### CAR CONTROL MODULE ###
include(${AAC_HOME}/share/cmake/AACECore.cmake)
include(${AAC_HOME}/share/cmake/AACEAlexa.cmake)
include(${AAC_HOME}/share/cmake/AACECarControl.cmake)

set(CARCONTROL_CPP_DIR ${CMAKE_CURRENT_SOURCE_DIR}/src/main/cpp)
set(CARCONTROL_SOURCE
    ${CARCONTROL_CPP_DIR}/src/CarControl/CarControlBinder.cpp
    ${CARCONTROL_CPP_DIR}/src/CarControl/CarControlConfigurationBinder.cpp
)
set(CARCONTROL_DEPENDENCIES
    AACECarControlEngine
    AACECarControlPlatform
    ${CORE_MODULE_DIR}/build/intermediates/cmake/${NDK_BUILD_TYPE}/obj/${ANDROID_ABI}/libAACECoreAndroid.so
)

add_library(AACECarControlAndroid SHARED
    ${CARCONTROL_SOURCE}
)

target_include_directories(AACECarControlAndroid PUBLIC
    ${CARCONTROL_CPP_DIR}/include
    ${CORE_MODULE_DIR}/src/main/cpp/include
)

target_link_libraries(AACECarControlAndroid
    ${CARCONTROL_DEPENDENCIES}
    android
    log
)

# Copy all AAC dependencies
add_custom_command(
    TARGET AACECarControlAndroid
    POST_BUILD
    COMMAND ${CMAKE_COMMAND} -E copy
        $<TARGET_FILE:AACECarControlPlatform>
        $<TARGET_FILE:AACECarControlEngine>
        $<TARGET_FILE_DIR:AACECarControlAndroid>
)
