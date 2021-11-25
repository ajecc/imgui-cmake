cmake_minimum_required(VERSION 3.16.3)

project(imgui-cmake LANGUAGES C CXX)

set(CPM_DOWNLOAD_VERSION 0.34.0)
if(CPM_SOURCE_CACHE)
  set(CPM_DOWNLOAD_LOCATION "${CPM_SOURCE_CACHE}/cpm/CPM_${CPM_DOWNLOAD_VERSION}.cmake")
elseif(DEFINED ENV{CPM_SOURCE_CACHE})
  set(CPM_DOWNLOAD_LOCATION "$ENV{CPM_SOURCE_CACHE}/cpm/CPM_${CPM_DOWNLOAD_VERSION}.cmake")
else()
  set(CPM_DOWNLOAD_LOCATION "${CMAKE_BINARY_DIR}/cmake/CPM_${CPM_DOWNLOAD_VERSION}.cmake")
endif()
if(NOT (EXISTS ${CPM_DOWNLOAD_LOCATION}))
  message(STATUS "Downloading CPM.cmake to ${CPM_DOWNLOAD_LOCATION}")
  file(DOWNLOAD
       https://github.com/TheLartians/CPM.cmake/releases/download/v${CPM_DOWNLOAD_VERSION}/CPM.cmake
       ${CPM_DOWNLOAD_LOCATION}
  )
endif()
include(${CPM_DOWNLOAD_LOCATION})



set(IMGUI_DIR ${CMAKE_CURRENT_SOURCE_DIR}/imgui)
add_library(IMGUI STATIC)

target_sources( IMGUI
                PRIVATE
                    ${IMGUI_DIR}/imgui_demo.cpp
                    ${IMGUI_DIR}/imgui_draw.cpp
                    ${IMGUI_DIR}/imgui_tables.cpp
                    ${IMGUI_DIR}/imgui_widgets.cpp
                    ${IMGUI_DIR}/imgui.cpp

                PRIVATE
                    ${IMGUI_DIR}/backends/imgui_impl_opengl3.cpp
                    ${IMGUI_DIR}/backends/imgui_impl_sdl.cpp
                )

target_include_directories( IMGUI
                            PUBLIC ${IMGUI_DIR}
                            PUBLIC ${IMGUI_DIR}/backends
                            )


if(RASPBIAN)
    target_compile_definitions(IMGUI PUBLIC -D__RASPBIAN__)
    set(USE_GLES ON) 
else()
    CPMAddPackage(
            NAME SDL 
            GITHUB_REPOSITORY ajecc/SDL
            VERSION 0.1.0
    )
endif()


if(USE_GLES)
    target_compile_definitions(IMGUI PUBLIC -DIMGUI_IMPL_OPENGL_ES2)

    if(RASPBIAN AND USE_BROADCOM_DRIVER)
        target_link_directories(IMGUI PUBLIC /opt/vc/lib)
        target_link_libraries(IMGUI PUBLIC brcmGLESv2 brcmEGL)
    else()
        target_link_libraries(IMGUI PUBLIC GLESv2)
    endif()
else()
    find_package(OpenGL REQUIRED)
    target_link_libraries(IMGUI PUBLIC SDL2main SDL2 ${OPENGL_LIBRARIES})
endif()


target_link_libraries(IMGUI PUBLIC ${CMAKE_DL_LIBS})



if(BUILD_EXAMPLES)
    add_executable(example_sdl_opengl3)
    target_sources(example_sdl_opengl3 PUBLIC ${IMGUI_DIR}/examples/example_sdl_opengl3/main.cpp)
    target_link_libraries(example_sdl_opengl3 IMGUI)
    set_target_properties(example_sdl_opengl3 PROPERTIES RUNTIME_OUTPUT_DIRECTORY ${CMAKE_BINARY_DIR})
endif()
