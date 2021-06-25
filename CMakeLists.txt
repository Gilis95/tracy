cmake_minimum_required(VERSION 3.14)

project(tracy VERSION 0.6.3 LANGUAGES CXX)

set(HDR_DIR ${PROJECT_SOURCE_DIR})
set(CLIENT_HDR_DIR ${HDR_DIR}/client)
set(COMMON_HDR_DIR ${HDR_DIR}/common)

set(SRC_DIR ${PROJECT_SOURCE_DIR})
set(CLIENT_SRC_DIR ${SRC_DIR}/client)
set(COMMON_SRC_DIR ${SRC_DIR}/common)

option(TRACY_INSTALL "Generate installation target" OFF)

set(TRACY_HEADERS
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

set(TRACY_SOURCES
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

add_library(tracy STATIC ${TRACY_HEADERS} ${TRACY_SOURCES})

target_include_directories(tracy PUBLIC
        $<BUILD_INTERFACE:${HDR_DIR}>
        $<INSTALL_INTERFACE:include>)
target_compile_definitions(tracy PRIVATE TRACY_ENABLE)
set_target_properties(tracy PROPERTIES POSITION_INDEPENDENT_CODE ON)

# Install
if(TRACY_INSTALL)
    # cmake install dirs
    include(GNUInstallDirs)

    set(version_config "${CMAKE_CURRENT_BINARY_DIR}/tracyConfigVersion.cmake")
    set(project_config "${CMAKE_CURRENT_BINARY_DIR}/tracyConfig.cmake")
    set(targets_export_name "tracyTargets")
    set(namespace "tracy::")

    # Include module with function 'write_basic_package_version_file'
    include(CMakePackageConfigHelpers)

    # Configure 'gladConfigVersion.cmake'
    # PROJECT_VERSION is used as a VERSION
    write_basic_package_version_file("${version_config}" COMPATIBILITY SameMajorVersion)

    # Configure 'tracyConfig.cmake'
    # Uses targets_export_name variable.
    configure_package_config_file(
            "Config.cmake.in"
            "${project_config}"
            INSTALL_DESTINATION ${CMAKE_INSTALL_FULL_LIBDIR}/cmake/tracy)

    # Targets:
    install(
            TARGETS tracy
            EXPORT "${targets_export_name}"
            LIBRARY DESTINATION ${CMAKE_INSTALL_FULL_LIBDIR}
            ARCHIVE DESTINATION ${CMAKE_INSTALL_FULL_LIBDIR}
            RUNTIME DESTINATION ${CMAKE_INSTALL_FULL_BINDIR})

    install(FILES ${ROOT_HEADERS} ${ROOT_INLINE}
            DESTINATION ${CMAKE_INSTALL_FULL_INCLUDEDIR}/tracy)

    install(FILES ${COMMON_HEADERS} ${COMMON_INLINE}
            DESTINATION ${CMAKE_INSTALL_FULL_INCLUDEDIR}/tracy/common)

    install(FILES ${CLIENT_HEADERS} ${CLIENT_INLINE}
            DESTINATION ${CMAKE_INSTALL_FULL_INCLUDEDIR}/tracy/client)

    # Install tracyConfig.cmake, tracyConfigVersion.cmake
    install(
            FILES "${project_config}" "${version_config}"
            DESTINATION ${CMAKE_INSTALL_FULL_LIBDIR}/cmake/tracy)

    # Create and install tracyTargets.cmake
    install(
            EXPORT "${targets_export_name}"
            NAMESPACE "${namespace}"
            DESTINATION ${CMAKE_INSTALL_FULL_LIBDIR}/cmake/tracy)

endif()
