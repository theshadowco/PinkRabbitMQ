# Cmake script for background building 64 bit release project library PinkRabbitMQLinux

CMAKE_MINIMUM_REQUIRED(VERSION 3.16 FATAL_ERROR)


PROJECT(PinkRabbitMQ64)

set (CMAKE_CXX_STANDARD 17)
set (ROOT_DIR ${CMAKE_SOURCE_DIR}/..)
set (CMAKE_SOURCE_DIR ${ROOT_DIR}/src)
set (LIB_DIR ${ROOT_DIR}/lib/linux)
set(CMAKE_C_FLAGS "${CMAKE_C_FLAGS} -fvisibility=hidden -fPIC")
set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -fvisibility=hidden -fvisibility-inlines-hidden -fPIC")

SET(AddInNative_SRC
    ${CMAKE_SOURCE_DIR}/linux/ConnectionImpl.cpp
    ${CMAKE_SOURCE_DIR}/Connection.cpp
    ${CMAKE_SOURCE_DIR}/RabbitMQClient.cpp
    ${CMAKE_SOURCE_DIR}/RabbitMQClientNative.cpp
    ${CMAKE_SOURCE_DIR}/AddInNative.cpp
    ${CMAKE_SOURCE_DIR}/Utils.cpp
    )

include_directories(${CMAKE_SOURCE_DIR})
include_directories(${CMAKE_SOURCE_DIR}/linux/libevent)
include_directories(${CMAKE_SOURCE_DIR}/linux/openssl)

add_definitions(-DVERSION=${VERSION})
add_definitions(-DNAME_POSTFIX=${NAME_POSTFIX})

add_library(${PROJECT_NAME} SHARED ${AddInNative_SRC})

find_library(LIBEVENT NAMES event PATHS ${LIB_DIR} NO_DEFAULT_PATH)
find_library(LIBAMQP NAMES amqpcpp PATHS ${LIB_DIR} NO_DEFAULT_PATH)
find_library(LIBSSL NAMES ssl PATHS ${LIB_DIR} NO_DEFAULT_PATH)
find_library(LIBCRYPTO NAMES crypto PATHS ${LIB_DIR} NO_DEFAULT_PATH)


target_link_libraries(${PROJECT_NAME} ${LIBSSL} ${LIBCRYPTO} ${LIBAMQP} ${LIBEVENT} -lpthread -ldl)

#tests

add_library(python_test SHARED ${ROOT_DIR}/test/python_test.cpp)
target_link_libraries(python_test ${PROJECT_NAME})

enable_testing()

add_test(AllTests python3 -m pytest ../../test -v)
