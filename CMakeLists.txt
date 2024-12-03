cmake_minimum_required(VERSION 3.5)

project(kinect2)

# Définit les standards C et C++
if(NOT CMAKE_C_STANDARD)
  set(CMAKE_C_STANDARD 99)
endif()

if(NOT CMAKE_CXX_STANDARD)
  set(CMAKE_CXX_STANDARD 17)
endif()

# Options de compilation pour GCC ou Clang
if(CMAKE_COMPILER_IS_GNUCXX OR CMAKE_CXX_COMPILER_ID MATCHES "Clang")
  add_compile_options(-Wall -Wextra -Wpedantic)
endif()

# Inclusion des dépendances
find_package(ament_cmake REQUIRED)
find_package(freenect2 REQUIRED)
find_package(OpenCV REQUIRED)
find_package(rclcpp REQUIRED)
find_package(sensor_msgs REQUIRED)

# Ajout d'un sous-répertoire pour argparse
add_subdirectory(${CMAKE_CURRENT_SOURCE_DIR}/../argparse ${CMAKE_BINARY_DIR}/argparse_build)


# Déclaration de la bibliothèque principale du projet
add_library(${PROJECT_NAME}
  "src/kinect2.cpp"
  "src/utility.cpp"
)

target_include_directories(${PROJECT_NAME} PUBLIC
  $<BUILD_INTERFACE:${CMAKE_CURRENT_SOURCE_DIR}/include>
  $<INSTALL_INTERFACE:include>)

# Lien des bibliothèques externes
target_link_libraries(${PROJECT_NAME} argparse)

ament_target_dependencies(${PROJECT_NAME}
  freenect2 OpenCV rclcpp sensor_msgs)

# Installation de la bibliothèque
install(TARGETS ${PROJECT_NAME}
  EXPORT export_${PROJECT_NAME}
  ARCHIVE DESTINATION "lib"
  LIBRARY DESTINATION "lib"
  RUNTIME DESTINATION "bin"
)

# Déclaration des exécutables
set(EXECUTABLES
  "bin/kinect2_node.cpp")

foreach(EXECUTABLE ${EXECUTABLES})
  get_filename_component(TARGET ${EXECUTABLE} NAME_WE)
  add_executable(${TARGET} ${EXECUTABLE})
  target_link_libraries(${TARGET} ${PROJECT_NAME})
  install(TARGETS ${TARGET}
    DESTINATION "lib/${PROJECT_NAME}")
endforeach()

# Tests et dépendances supplémentaires
if(BUILD_TESTING)
  find_package(ament_lint_auto REQUIRED)
  ament_lint_auto_find_test_dependencies()
endif()

# Déclaration des dépendances exportées
ament_export_dependencies(freenect2 OpenCV rclcpp sensor_msgs)

ament_package()

