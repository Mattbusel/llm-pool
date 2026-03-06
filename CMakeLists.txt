cmake_minimum_required(VERSION 3.16)
project(llm_pool_examples CXX)
set(CMAKE_CXX_STANDARD 17)
set(CMAKE_CXX_STANDARD_REQUIRED ON)

if(MSVC)
    add_compile_options(/W4 /WX)
else()
    add_compile_options(-Wall -Wextra -Werror)
endif()

find_package(Threads REQUIRED)

foreach(ex basic_pool priority_queue rate_limited)
    add_executable(${ex} examples/${ex}.cpp)
    target_include_directories(${ex} PRIVATE include)
    target_link_libraries(${ex} Threads::Threads)
endforeach()
