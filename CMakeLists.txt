cmake_minimum_required(VERSION 3.14)

project(tracy VERSION 0.6.3 LANGUAGES CXX)

set(HDR_DIR ${PROJECT_SOURCE_DIR})
set(CLIENT_HDR_DIR ${HDR_DIR}/client)
set(COMMON_HDR_DIR ${HDR_DIR}/common)

set(SRC_DIR ${PROJECT_SOURCE_DIR})
set(CLIENT_SRC_DIR ${SRC_DIR}/client)
set(COMMON_SRC_DIR ${SRC_DIR}/common)

option(TRACY_INSTALL "Generate installation target" OFF)

file(GLOB_RECURSE CLIENT_HEADER ${CLIENT_HDR_DIR}/*.h)
file(GLOB_RECURSE CLIENT_SOURCE ${CLIENT_SRC_DIR}/*.cpp)
file(GLOB_RECURSE CLIENT_TEMPLATE_HEADER ${CLIENT_HDR_DIR}/*.hpp)

file(GLOB_RECURSE COMMON_HEADER ${COMMON_HDR_DIR}/*.h)
file(GLOB_RECURSE COMMON_SOURCE ${COMMON_SRC_DIR}/*.cpp)
file(GLOB_RECURSE COMMON_TEMPLATE_HEADER ${COMMON_HDR_DIR}/*.hpp)

file(GLOB ROOT_HEADER ${HDR_DIR}/*.h)
file(GLOB ROOT_SOURCE ${SRC_DIR}/*.cpp)
file(GLOB ROOT_TEMPLATE_HEADER ${HDR_DIR}/*.hpp)

set(TRACY_HEADERS ${CLIENT_HEADER} ${COMMON_HEADER} ${ROOT_HEADER})
set(TRACY_SOURCES ${CLIENT_SOURCE} ${COMMON_SOURCE} ${ROOT_SOURCE})
set(TRACY_TEMPLATE_HEADERS ${CLIENT_TEMPLATE_HEADER} ${COMMON_TEMPLATE_HEADER} ${ROOT_TEMPLATE_HEADER})

add_library(tracy STATIC ${TRACY_HEADERS} ${TRACY_TEMPLATE_HEADERS} ${TRACY_SOURCES})

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

    # Include module with fuction 'write_basic_package_version_file'
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

    install(FILES ${TRACY_HEADERS}
            DESTINATION ${CMAKE_INSTALL_FULL_INCLUDEDIR}/tracy)

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