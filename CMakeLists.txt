cmake_minimum_required(VERSION 3.13)

find_package(OpenEnclave CONFIG REQUIRED)

set(CMAKE_CXX_STANDARD 11)

add_subdirectory(enclave)
add_subdirectory(host)

# Generate key
add_custom_command(OUTPUT private.pem public.pem
        COMMAND openssl genrsa -out private.pem -3 3072
        COMMAND openssl rsa -in private.pem -pubout -out public.pem)
message(STATUS "tagggggggggggggggg11111")
# Sign enclave
add_custom_command(OUTPUT enclave/enclave.signed
        DEPENDS enclave enclave/helloworld.conf private.pem
        COMMAND openenclave::oesign sign -e $<TARGET_FILE:enclave> -c ${CMAKE_SOURCE_DIR}/enclave/helloworld.conf -k private.pem)

message(STATUS "tagggggggggggggggg22222")
add_custom_target(sign ALL DEPENDS enclave/enclave.signed)

if ((NOT DEFINED ENV{OE_SIMULATION}) OR (NOT $ENV{OE_SIMULATION}))
    add_custom_target(run
            DEPENDS helloworld_host sign
            COMMAND helloworld_host ${CMAKE_SOURCE_DIR}/enclave/enclave.signed)
endif ()

add_custom_target(simulate
        DEPENDS helloworld_host sign
        COMMAND helloworld_host ${CMAKE_SOURCE_DIR}/enclave/enclave.signed --simulate)
