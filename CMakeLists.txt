cmake_minimum_required(VERSION 3.14)
project(tracy VERSION 0.6.3)

set(SOURCE_SUBFOLDER ${CMAKE_CURRENT_SOURCE_DIR})

add_library(tracy SHARED ${SOURCE_SUBFOLDER}/TracyClient.cpp)
target_include_directories(tracy PUBLIC $<INSTALL_INTERFACE:include>)
target_compile_definitions(tracy PRIVATE TRACY_ENABLE)
set_target_properties(tracy PROPERTIES POSITION_INDEPENDENT_CODE ON)

# NOTE: Library requires builtin imgui
find_package(glfw REQUIRED)
find_package(Freetype REQUIRED)
find_package(Threads REQUIRED)

add_executable(profiler
        ${SOURCE_SUBFOLDER}/common/TracySocket.cpp
        ${SOURCE_SUBFOLDER}/common/TracySystem.cpp
        ${SOURCE_SUBFOLDER}/common/tracy_lz4.cpp
        ${SOURCE_SUBFOLDER}/common/tracy_lz4hc.cpp
        ${SOURCE_SUBFOLDER}/imguicolortextedit/TextEditor.cpp
        ${SOURCE_SUBFOLDER}/imgui/imgui.cpp
        ${SOURCE_SUBFOLDER}/imgui/imgui_demo.cpp
        ${SOURCE_SUBFOLDER}/imgui/imgui_draw.cpp
        ${SOURCE_SUBFOLDER}/imgui/imgui_widgets.cpp
        ${SOURCE_SUBFOLDER}/nfd/nfd_common.c
        ${SOURCE_SUBFOLDER}/server/TracyBadVersion.cpp
        ${SOURCE_SUBFOLDER}/server/TracyMemory.cpp
        ${SOURCE_SUBFOLDER}/server/TracyMmap.cpp
        ${SOURCE_SUBFOLDER}/server/TracyPrint.cpp
        ${SOURCE_SUBFOLDER}/server/TracyStorage.cpp
        ${SOURCE_SUBFOLDER}/server/TracyTaskDispatch.cpp
        ${SOURCE_SUBFOLDER}/server/TracyTexture.cpp
        ${SOURCE_SUBFOLDER}/server/TracyThreadCompress.cpp
        ${SOURCE_SUBFOLDER}/server/TracyUserData.cpp
        ${SOURCE_SUBFOLDER}/server/TracyView.cpp
        ${SOURCE_SUBFOLDER}/server/TracyWorker.cpp
        ${SOURCE_SUBFOLDER}/zstd/debug.c
        ${SOURCE_SUBFOLDER}/zstd/entropy_common.c
        ${SOURCE_SUBFOLDER}/zstd/error_private.c
        ${SOURCE_SUBFOLDER}/zstd/fse_compress.c
        ${SOURCE_SUBFOLDER}/zstd/fse_decompress.c
        ${SOURCE_SUBFOLDER}/zstd/hist.c
        ${SOURCE_SUBFOLDER}/zstd/huf_compress.c
        ${SOURCE_SUBFOLDER}/zstd/huf_decompress.c
        ${SOURCE_SUBFOLDER}/zstd/pool.c
        ${SOURCE_SUBFOLDER}/zstd/threading.c
        ${SOURCE_SUBFOLDER}/zstd/xxhash.c
        ${SOURCE_SUBFOLDER}/zstd/zstdmt_compress.c
        ${SOURCE_SUBFOLDER}/zstd/zstd_common.c
        ${SOURCE_SUBFOLDER}/zstd/zstd_compress.c
        ${SOURCE_SUBFOLDER}/zstd/zstd_compress_literals.c
        ${SOURCE_SUBFOLDER}/zstd/zstd_compress_sequences.c
        ${SOURCE_SUBFOLDER}/zstd/zstd_compress_superblock.c
        ${SOURCE_SUBFOLDER}/zstd/zstd_ddict.c
        ${SOURCE_SUBFOLDER}/zstd/zstd_decompress.c
        ${SOURCE_SUBFOLDER}/zstd/zstd_decompress_block.c
        ${SOURCE_SUBFOLDER}/zstd/zstd_double_fast.c
        ${SOURCE_SUBFOLDER}/zstd/zstd_fast.c
        ${SOURCE_SUBFOLDER}/zstd/zstd_lazy.c
        ${SOURCE_SUBFOLDER}/zstd/zstd_ldm.c
        ${SOURCE_SUBFOLDER}/zstd/zstd_opt.c
        ${SOURCE_SUBFOLDER}/profiler/libs/gl3w/GL/gl3w.c
        ${SOURCE_SUBFOLDER}/profiler/src/imgui_freetype.cpp
        ${SOURCE_SUBFOLDER}/profiler/src/imgui_impl_glfw.cpp
        ${SOURCE_SUBFOLDER}/profiler/src/imgui_impl_opengl3.cpp
        ${SOURCE_SUBFOLDER}/profiler/src/main.cpp
        ${SOURCE_SUBFOLDER}/profiler/src/ResolvService.cpp
        )
if(APPLE)
    target_sources(profiler PRIVATE ${SOURCE_SUBFOLDER}/nfd/nfd_cocoa.m)
elseif(UNIX)
    target_sources(profiler PRIVATE ${SOURCE_SUBFOLDER}/nfd/nfd_gtk.c)
    find_package(GTK2 REQUIRED)
    target_include_directories(profiler PRIVATE ${GTK2_INCLUDE_DIRS})
    target_link_libraries(profiler PRIVATE dl ${GTK2_LIBRARIES} GL)
elseif(WIN32)
    target_sources(profiler PRIVATE ${SOURCE_SUBFOLDER}/nfd/nfd_win.cpp)
else()
    message(FATAL_ERROR "Unknown platform")
endif()

target_compile_definitions(profiler PRIVATE TRACY_FILESELECTOR TRACY_EXTENDED_FONT TRACY_ROOT_WINDOW IMGUI_IMPL_OPENGL_LOADER_GL3W)
target_include_directories(profiler PRIVATE
        ${SOURCE_SUBFOLDER}/profiler/libs/gl3w
        ${SOURCE_SUBFOLDER}/imgui)
target_link_libraries(profiler PRIVATE Freetype::Freetype glfw Threads::Threads CONAN_PKG::tbb)
target_compile_features(profiler PRIVATE cxx_std_17)



include(GNUInstallDirs)
install(TARGETS tracy profiler
        EXPORT tracy-export
        LIBRARY DESTINATION ${CMAKE_INSTALL_LIBDIR}
        ARCHIVE DESTINATION ${CMAKE_INSTALL_LIBDIR}
        PUBLIC_HEADER DESTINATION ${CMAKE_INSTALL_INCLUDEDIR}
        )
install(DIRECTORY ${SOURCE_SUBFOLDER} DESTINATION ${CMAKE_INSTALL_INCLUDEDIR} FILES_MATCHING PATTERN "*.hpp" PATTERN "*.h")
# For projects consuming the library via CMake
install(EXPORT tracy-export
        FILE tracyConfig.cmake
        NAMESPACE tracy::
        DESTINATION ${CMAKE_INSTALL_LIBDIR}/cmake/tracy)

# For projects including the entire library source tree
export(TARGETS tracy NAMESPACE tracy:: FILE tracyConfig.cmake)