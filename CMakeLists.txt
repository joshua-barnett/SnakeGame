cmake_minimum_required(VERSION 3.5)

project(SnakeGame CXX)

file(GLOB_RECURSE SNAKE_GAME_SRCS Src/*.cpp)
file(GLOB_RECURSE SNAKE_GAME_HDRS Src/*.h)

add_executable(${CMAKE_PROJECT_NAME} ${SNAKE_GAME_SRCS} ${SNAKE_GAME_HDRS})

target_compile_features(${CMAKE_PROJECT_NAME} PRIVATE cxx_std_17)

if(DEFINED CMAKE_CONFIGURATION_TYPES)
  if(NOT EXISTS "${CMAKE_BINARY_DIR}/${CMAKE_BUILD_TYPE}/Assets")
    message(STATUS "Copying assets to build directory")
    file(COPY Assets DESTINATION ${CMAKE_BINARY_DIR}/${CMAKE_BUILD_TYPE})
  endif()
else()
  if(NOT EXISTS "${CMAKE_BINARY_DIR}/Assets")
    message(STATUS "Copying assets to build directory")
    file(COPY Assets DESTINATION ${CMAKE_BINARY_DIR})
  endif()
endif()

list(APPEND CMAKE_MODULE_PATH ${CMAKE_BINARY_DIR})
list(APPEND CMAKE_PREFIX_PATH ${CMAKE_BINARY_DIR})

if(NOT EXISTS "${CMAKE_BINARY_DIR}/conan.cmake")
  message(STATUS "Downloading conan.cmake from https://github.com/conan-io/cmake-conan")
  file(DOWNLOAD "https://raw.githubusercontent.com/conan-io/cmake-conan/v0.16.1/conan.cmake"
                "${CMAKE_BINARY_DIR}/conan.cmake"
                EXPECTED_HASH SHA256=396e16d0f5eabdc6a14afddbcfff62a54a7ee75c6da23f32f7a31bc85db23484
                TLS_VERIFY ON)
endif()

include(${CMAKE_BINARY_DIR}/conan.cmake)

if(DEFINED CMAKE_CONFIGURATION_TYPES)
  set(CONAN_GENERATOR cmake_find_package_multi)
else()
  set(CONAN_GENERATOR cmake_find_package)
endif()

conan_cmake_configure(REQUIRES sdl2/2.0.14@bincrafters/stable
                      REQUIRES sdl2_ttf/2.0.15@bincrafters/stable
                      REQUIRES sdl2_image/2.0.5@bincrafters/stable
                      REQUIRES entt/3.0.0@skypjack/stable
                      GENERATORS ${CONAN_GENERATOR})

if(DEFINED CMAKE_CONFIGURATION_TYPES)
  foreach(TYPE ${CMAKE_CONFIGURATION_TYPES})
    conan_cmake_autodetect(settings BUILD_TYPE ${TYPE})
    conan_cmake_install(PATH_OR_REFERENCE .
                        BUILD missing
                        REMOTE conan-center
                        REMOTE bincrafters
                        SETTINGS ${settings})
  endforeach()
else()
  conan_cmake_autodetect(settings)
  conan_cmake_install(PATH_OR_REFERENCE .
                      BUILD missing
                      REMOTE conan-center
                      REMOTE bincrafters
                      SETTINGS ${settings})
endif()

if(DEFINED CMAKE_CONFIGURATION_TYPES)
  find_package(SDL2 CONFIG)
  find_package(SDL2_image CONFIG)
  find_package(SDL2_ttf CONFIG)
  find_package(entt CONFIG)
else()
  find_package(SDL2)
  find_package(SDL2_image)
  find_package(SDL2_ttf)
  find_package(entt)
endif()

target_link_libraries(${CMAKE_PROJECT_NAME} PRIVATE SDL2::SDL2main)
if(DEFINED CMAKE_CONFIGURATION_TYPES)
  target_link_libraries(${CMAKE_PROJECT_NAME} PRIVATE SDL2_image::SDL2_image)
else()
  target_link_libraries(${CMAKE_PROJECT_NAME} PRIVATE SDL2_image::SDL2_image)
endif()
target_link_libraries(${CMAKE_PROJECT_NAME} PRIVATE SDL2_ttf::SDL2_ttf)
target_link_libraries(${CMAKE_PROJECT_NAME} PRIVATE entt::entt)
