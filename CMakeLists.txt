cmake_minimum_required(VERSION 3.15)
project(tinyengine CXX C)
set(CMAKE_EXPORT_COMPILE_COMMANDS ON)

# Set C++ standard
set(CMAKE_CXX_STANDARD 17)

include_directories(src lib/glfw/include lib/glm lib/glad/include lib/imgui lib/imgui/backends)

# Add subdirectories for dependencies
add_subdirectory(lib/glfw)
add_subdirectory(lib/glm)

# Add glad library
add_library(glad STATIC lib/glad/src/glad.c) 
target_include_directories(glad PUBLIC lib/glad/include)

# add imgui library
file(GLOB IMGUI_SOURCES "lib/imgui/*.cpp" "lib/imgui/backends/imgui_impl_glfw.cpp" "lib/imgui/backends/imgui_impl_opengl3.cpp")
add_library(imgui STATIC ${IMGUI_SOURCES})
target_include_directories(imgui PUBLIC lib/imgui lib/imgui/backends)

# Add sources for the tinyengine project
file(GLOB_RECURSE SOURCES "src/*.cpp" "src/*.hpp")

add_executable(tinyengine ${SOURCES})

# Link libraries
target_link_libraries(tinyengine glad glm glfw imgui m)

if(APPLE)
    target_link_libraries(tinyengine
        "-framework OpenGL"
        "-framework IOKit"
        "-framework CoreVideo"
        "-framework Cocoa"
    )
elseif(UNIX)
    target_link_libraries(tinyengine GL dl pthread)
endif()

# Set the output directory for the executable
set_target_properties(tinyengine PROPERTIES RUNTIME_OUTPUT_DIRECTORY "${CMAKE_BINARY_DIR}/bin")

# Ensure bin directory exists before build
add_custom_command(
    TARGET tinyengine PRE_BUILD
    COMMAND ${CMAKE_COMMAND} -E make_directory "${CMAKE_BINARY_DIR}/bin"
)

# Copy assets and shaders to the bin directory after build
add_custom_command(
    TARGET tinyengine POST_BUILD
    COMMAND ${CMAKE_COMMAND} -E copy_directory "${CMAKE_SOURCE_DIR}/data/assets" "${CMAKE_BINARY_DIR}/bin/assets"
    COMMAND ${CMAKE_COMMAND} -E copy_directory "${CMAKE_SOURCE_DIR}/src/shaders" "${CMAKE_BINARY_DIR}/bin/shaders"
)
