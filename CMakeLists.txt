set(CMAKE_VERBOSE_MAKEFILE ON)

cmake_minimum_required(VERSION 3.16)

# set the project name and version
project(EmscriptenDearImGui)

# specify the C++ standard
set(CMAKE_CXX_STANDARD 17)
set(CMAKE_CXX_STANDARD_REQUIRED True)

include(cmake/DearImGui.cmake)

set(CMAKE_EXECUTABLE_SUFFIX ".html")

# add the executable
add_executable(EmscriptenDearImGui)
target_include_directories(EmscriptenDearImGui PRIVATE "imgui")
target_link_libraries(EmscriptenDearImGui imgui)

target_compile_definitions(EmscriptenDearImGui PRIVATE IMGUI_DEFINE_MATH_OPERATORS)

SET(NODE_DIR "imgui-node-editor")
file(GLOB  imgui_node_editor_sources CONFIGURE_DEPENDS ${NODE_DIR}/*.inl ${NODE_DIR}/*.h ${NODE_DIR}/*.cpp)
target_include_directories(EmscriptenDearImGui PRIVATE ${NODE_DIR})
target_sources(EmscriptenDearImGui PRIVATE
        "main.cpp"
        "emscripten_mainloop_stub.h"
        "blueprints-example.h"
        "stb_image.h"
        "renderer.h"
        "renderer_ogl3.cpp"
        "utilities/builders.h"
        "utilities/builders.cpp"
        "utilities/drawing.h"
        "utilities/drawing.cpp"
        "utilities/widgets.h"
        "utilities/widgets.cpp"

        ${imgui_node_editor_sources}
)

if (EMSCRIPTEN)
    target_compile_options(EmscriptenDearImGui PUBLIC "-sUSE_SDL=2;-sUSE_FREETYPE=1;-sUSE_SDL_IMAGE=2;")
    target_link_options(EmscriptenDearImGui PUBLIC "-sUSE_SDL=2;-sUSE_FREETYPE=1;-sUSE_SDL_IMAGE=2;")
    add_link_options("-sASSERTIONS")
    set_target_properties(EmscriptenDearImGui PROPERTIES LINK_FLAGS "--embed-file data")

    add_custom_command(TARGET EmscriptenDearImGui PRE_BUILD
                   COMMAND ${CMAKE_COMMAND} -E copy_directory
                       ${CMAKE_SOURCE_DIR}/data/ $<TARGET_FILE_DIR:EmscriptenDearImGui>/data/)
else()
    find_package(SDL2 REQUIRED)
    include_directories(EmscriptenDearImGui ${SDL2_INCLUDE_DIRS})
    target_link_libraries(EmscriptenDearImGui ${SDL2_LIBRARIES})

    target_link_libraries(EmscriptenDearImGui ${CMAKE_DL_LIBS})

    find_package(OpenGL REQUIRED)
    include_directories(EmscriptenDearImGui ${OPENGL_INCLUDE_DIR})
    target_link_libraries(EmscriptenDearImGui ${OPENGL_LIBRARIES})

    target_sources(EmscriptenDearImGui PRIVATE
        "onnx.proto3.pb.h"
        "onnx.proto3.pb.cc"
    )
    
    find_package(Protobuf REQUIRED)
    include_directories(${Protobuf_INCLUDE_DIRS})
    target_link_libraries(EmscriptenDearImGui ${Protobuf_LIBRARIES})
endif()
