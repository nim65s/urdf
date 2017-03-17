include(GNUInstallDirs)
cmake_minimum_required(VERSION 2.8.3)
project(urdf)

find_package(Boost REQUIRED thread)
find_package(urdfdom REQUIRED)
find_package(urdfdom_headers REQUIRED)
find_package(PkgConfig)
pkg_check_modules(TinyXML tinyxml)

# declare ModelInterface's SharedPtr
set(URDFDOM_DECLARE_TYPES 1)
set(generated_compat_header "${PROJECT_BINARY_DIR}/include/${PROJECT_NAME}/urdfdom_compatibility.h")
include_directories(include ${PROJECT_BINARY_DIR}/include)
configure_file(urdfdom_compatibility.h.in "${generated_compat_header}" @ONLY)

install(FILES ${generated_compat_header} DESTINATION ${CMAKE_INSTALL_INCLUDEDIR}/${PROJECT_NAME})


include_directories(
  include
  ${Boost_INCLUDE_DIR}
  ${urdfdom_INCLUDE_DIRS}
  ${urdfdom_headers_INCLUDE_DIRS}
  ${TinyXML_INCLUDE_DIRS}
  )

link_directories(${Boost_LIBRARY_DIRS})

add_library(${PROJECT_NAME} SHARED src/model.cpp)
target_link_libraries(${PROJECT_NAME} ${TinyXML_LIBRARIES} ${urdfdom_LIBRARIES})

if(APPLE)
  set_target_properties(${PROJECT_NAME} PROPERTIES LINK_FLAGS "-undefined dynamic_lookup")
endif(APPLE)

set(PKG_NAME ${PROJECT_NAME})
set(PKG_VERSION 1.50.0)
set(PKG_LIBS -lurdf)
set(PKG_DEPENDS "urdfdom urdfdom_headers")
set(cmake_conf_file "cmake/pkgconfig/urdf.pc")
configure_file("${CMAKE_CURRENT_SOURCE_DIR}/${cmake_conf_file}.in" "${CMAKE_BINARY_DIR}/${cmake_conf_file}" @ONLY)
install(FILES ${CMAKE_BINARY_DIR}/${cmake_conf_file}
  DESTINATION ${CMAKE_INSTALL_LIBDIR}/pkgconfig)

install(TARGETS ${PROJECT_NAME} 
  ARCHIVE DESTINATION ${CMAKE_INSTALL_LIBDIR}
  LIBRARY DESTINATION ${CMAKE_INSTALL_LIBDIR}
  RUNTIME DESTINATION ${CMAKE_INSTALL_BINDIR})

install(DIRECTORY include/${PROJECT_NAME}/
  DESTINATION ${CMAKE_INSTALL_INCLUDEDIR}/${PROJECT_NAME})

install(FILES package.xml DESTINATION
  ${CMAKE_INSTALL_DATAROOTDIR}/${PROJECT_NAME})
