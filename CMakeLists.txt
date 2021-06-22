cmake_minimum_required(VERSION 3.14)

project(tracy VERSION 0.6.3 LANGUAGES CXX)

set(HDR_DIR ${PROJECT_SOURCE_DIR})
set(CLIENT_HDR_DIR ${HDR_DIR}/client)
set(COMMON_HDR_DIR ${HDR_DIR}/common)

set(SRC_DIR ${PROJECT_SOURCE_DIR})
set(CLIENT_SRC_DIR ${SRC_DIR}/client)
set(COMMON_SRC_DIR ${SRC_DIR}/common)

option(TRACY_INSTALL "Generate installation target" OFF)

file(GLOB_RECURSE CLIENT_HEADERS ${CLIENT_HDR_DIR}/*.h)
file(GLOB_RECURSE CLIENT_SOURCE ${CLIENT_SRC_DIR}/*.cpp)
file(GLOB_RECURSE CLIENT_INLINE ${CLIENT_HDR_DIR}/*.hpp)

file(GLOB_RECURSE COMMON_HEADERS ${COMMON_HDR_DIR}/*.h)
file(GLOB_RECURSE COMMON_INLINE ${COMMON_HDR_DIR}/*.hpp)
file(GLOB_RECURSE COMMON_SOURCE ${COMMON_SRC_DIR}/*.cpp)

file(GLOB ROOT_HEADERS ${HDR_DIR}/*.h)
file(GLOB ROOT_INLINE ${HDR_DIR}/*.hpp)
file(GLOB ROOT_SOURCE ${SRC_DIR}/*.cpp)

set(TRACY_HEADERS ${CLIENT_HEADERS} ${CLIENT_INLINE} ${COMMON_HEADERS} ${COMMON_INLINE} ${ROOT_HEADERS} ${ROOT_INLINE})
set(TRACY_SOURCES ${CLIENT_SOURCE} ${COMMON_SOURCE} ${ROOT_SOURCE})

add_library(tracy STATIC ${TRACY_HEADERS} ${TRACY_SOURCES})

target_include_directories(tracy PUBLIC $<INSTALL_INTERFACE:include>)
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
