#-----------------------------------------------------------------------------
# CmakeLists.txt for the Digital Bitbox
# Copyright (c) 2015-2017 Lucas Betschart, Douglas J. Bakkum
# MIT License


include_directories(../src)
if(USE_SECP256K1_LIB)
    include_directories(../src/secp256k1)
    include_directories(../src/secp256k1/src)
endif()

#-----------------------------------------------------------------------------
# Build tests_unit
add_executable(tests_unit tests_unit.c)
target_link_libraries(tests_unit bitbox)

#-----------------------------------------------------------------------------
# Generate test vectors for the base64 unit test.
add_custom_command(TARGET tests_unit
  POST_BUILD
  COMMAND python "${CMAKE_CURRENT_SOURCE_DIR}/generate_base64_testcases.py"
  --output "${CMAKE_BINARY_DIR}/bin/base64_testcases.bin"
  --seed 31337 --count 10000 --maxSize 2000
  )


#-----------------------------------------------------------------------------
# Build tests_openssl
find_package(OpenSSL REQUIRED)
include_directories(${OPENSSL_INCLUDE_DIR})
add_executable(tests_openssl tests_openssl.c)
target_link_libraries(tests_openssl bitbox ${OPENSSL_LIBRARIES})


#-----------------------------------------------------------------------------
# Build secp256k1 submodule tests
if(USE_SECP256K1_LIB)
    add_executable(tests_secp256k1 tests_secp256k1.c)
    target_link_libraries(tests_secp256k1 bitbox)
endif()

#-----------------------------------------------------------------------------
# Build yajl submodule tests
add_executable(tests_yajl tests_yajl.c)
target_link_libraries(tests_yajl bitbox)


#-----------------------------------------------------------------------------
# Create hidapi static library for live testing

include_directories(hidapi/hidapi)
if(WIN32)
    set(HIDAPI-SOURCES hidapi/windows/hid.c)
elseif(APPLE)
    set(HIDAPI-SOURCES hidapi/mac/hid.c)
else()
    set(HIDAPI-SOURCES hidapi/linux/hid.c)
endif()
#Disable all the warnings generated by the HID library
set_source_files_properties(${HIDAPI-SOURCES} PROPERTIES COMPILE_FLAGS "-w")
add_library(hidapi STATIC ${HIDAPI-SOURCES})


#-----------------------------------------------------------------------------
# Build tests_api

add_executable(
    tests_api
    tests_api.c
    ${HIDAPI-SOURCES}
)
if(UNIX AND NOT APPLE)
    target_link_libraries(tests_api bitbox hidapi udev)
else()
    target_link_libraries(tests_api bitbox hidapi)
endif()

# Location for sham SD files
execute_process(COMMAND mkdir "-p" "digitalbitbox" WORKING_DIRECTORY ${CMAKE_CURRENT_BINARY_DIR})


#-----------------------------------------------------------------------------
# Build tests_u2f_hid

add_executable(
    tests_u2f_hid
    tests_u2f_hid.c
    u2f/u2f_util_t.c
    ${HIDAPI-SOURCES}
)
if(UNIX AND NOT APPLE)
    target_link_libraries(tests_u2f_hid bitbox hidapi udev)
else()
    target_link_libraries(tests_u2f_hid bitbox hidapi)
endif()


#-----------------------------------------------------------------------------
# Build tests_u2f_standard

add_executable(
    tests_u2f_standard
    tests_u2f_standard.c
    u2f/u2f_util_t.c
    ${HIDAPI-SOURCES}
)
if(UNIX AND NOT APPLE)
    target_link_libraries(tests_u2f_standard bitbox hidapi udev)
else()
    target_link_libraries(tests_u2f_standard bitbox hidapi)
endif()
