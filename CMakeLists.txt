cmake_minimum_required(VERSION 3.14)
project(tracy VERSION 0.6.3)

set(HDR_DIR ${PROJECT_SOURCE_DIR})
set(CLIENT_HDR_DIR ${HDR_DIR}/client)
set(COMMON_HDR_DIR ${HDR_DIR}/common)

set(SRC_DIR ${PROJECT_SOURCE_DIR})
set(CLIENT_SRC_DIR ${SRC_DIR}/client)
set(COMMON_SRC_DIR ${SRC_DIR}/common)

set(HDR_FILES
        ${HDR_DIR}/Tracy.hpp
        ${HDR_DIR}/TracyC.h

        ${CLIENT_HDR_DIR}/tracy_concurrentqueue.h
        ${CLIENT_HDR_DIR}/tracy_rpmalloc.hpp
        ${CLIENT_HDR_DIR}/TracyArmCpuTable.hpp
        ${CLIENT_HDR_DIR}/TracyCallstack.h
        ${CLIENT_HDR_DIR}/TracyCallstack.hpp
        ${CLIENT_HDR_DIR}/TracyDxt1.hpp
        ${CLIENT_HDR_DIR}/TracyFastVector.hpp
        ${CLIENT_HDR_DIR}/TracyLock.hpp
        ${CLIENT_HDR_DIR}/TracyProfiler.hpp
        ${CLIENT_HDR_DIR}/TracyRingBuffer.hpp
        ${CLIENT_HDR_DIR}/TracyScoped.hpp
        ${CLIENT_HDR_DIR}/TracySysTime.hpp
        ${CLIENT_HDR_DIR}/TracySysTrace.hpp
        ${CLIENT_HDR_DIR}/TracySysTracePayload.hpp
        ${CLIENT_HDR_DIR}/TracyThread.hpp

        ${COMMON_HDR_DIR}/tracy_lz4.hpp
        ${COMMON_HDR_DIR}/tracy_lz4hc.hpp
        ${COMMON_HDR_DIR}/TracyAlign.hpp
        ${COMMON_HDR_DIR}/TracyAlloc.hpp
        ${COMMON_HDR_DIR}/TracyApi.h
        ${COMMON_HDR_DIR}/TracyColor.hpp
        ${COMMON_HDR_DIR}/TracyForceInline.hpp
        ${COMMON_HDR_DIR}/TracyMutex.hpp
        ${COMMON_HDR_DIR}/TracyProtocol.hpp
        ${COMMON_HDR_DIR}/TracyQueue.hpp
        ${COMMON_HDR_DIR}/TracySocket.hpp
        ${COMMON_HDR_DIR}/TracySystem.hpp
        )

set(SRC_FILES
        ${SRC_DIR}/TracyClient.cpp

        ${CLIENT_SRC_DIR}/tracy_rpmalloc.cpp
        ${CLIENT_SRC_DIR}/TracyCallstack.cpp
        ${CLIENT_SRC_DIR}/TracyDxt1.cpp
        ${CLIENT_SRC_DIR}/TracyProfiler.cpp
        ${CLIENT_SRC_DIR}/TracySysTime.cpp
        ${CLIENT_SRC_DIR}/TracySysTrace.cpp

        ${COMMON_SRC_DIR}/tracy_lz4.cpp
        ${COMMON_SRC_DIR}/tracy_lz4hc.cpp
        ${COMMON_SRC_DIR}/TracySocket.cpp
        ${COMMON_SRC_DIR}/TracySystem.cpp
        )

add_library(tracy STATIC ${HDR_FILES} ${SRC_FILES})

target_include_directories(tracy PUBLIC $<INSTALL_INTERFACE:include>)
target_compile_definitions(tracy PRIVATE TRACY_ENABLE)
set_target_properties(tracy PROPERTIES POSITION_INDEPENDENT_CODE ON)