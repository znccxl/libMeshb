

################################
# GLOBAL CONFIGURATION VARIABLES
################################

cmake_minimum_required (VERSION 3.7.2)
set (libMeshb_VERSION_MAJOR 7)
set (libMeshb_VERSION_MINOR 54)
project(libMeshb VERSION ${libMeshb_VERSION_MAJOR}.${libMeshb_VERSION_MINOR} LANGUAGES C)

include (CheckLanguage)
check_language (Fortran)
if(CMAKE_Fortran_COMPILER)
  enable_language(Fortran)
  set(CMAKE_Fortran_MODULE_DIRECTORY ${CMAKE_BINARY_DIR}/modules)
  if (CMAKE_Fortran_COMPILER_ID STREQUAL GNU)
     set (CMAKE_Fortran_FLAGS "${CMAKE_Fortran_FLAGS} -std=legacy")
  endif ()
endif()

if (NOT CMAKE_BUILD_TYPE)
  set(CMAKE_BUILD_TYPE Release)
endif ()

if(CMAKE_INSTALL_PREFIX_INITIALIZED_TO_DEFAULT)
   if (WIN32)
      set(CMAKE_INSTALL_PREFIX "$ENV{HOMEPATH}/local" CACHE PATH "..." FORCE)
   else ()
      set(CMAKE_INSTALL_PREFIX "$ENV{HOME}/local" CACHE PATH "..." FORCE)
   endif()
endif ()

if(AIO)
   set (CMAKE_C_FLAGS "${CMAKE_C_FLAGS} -DWITH_AIO")
   IF(${CMAKE_SYSTEM_NAME} MATCHES Linux)
      set(AIO_LIBRARIES rt)
   endif()
endif()


#######################################
# SET FILES AND DIRECTORIES TO BE BUILT
#######################################

include_directories (${PROJECT_SOURCE_DIR}/sources)
add_subdirectory (sources)
add_subdirectory (examples)
add_subdirectory (utilities)

install (FILES LICENSE.txt copyright.txt DESTINATION share/libMeshb)
install (DIRECTORY sample_meshes DESTINATION share/libMeshb)


######################################
# SET PACKAGE AND DEPLOYMENT VARIABLES
######################################

if (CPACK)
   include (InstallRequiredSystemLibraries)
   set (CPACK_GENERATOR TXZ)
   set (CPACK_RESOURCE_FILE_LICENSE ${CMAKE_CURRENT_SOURCE_DIR}/copyright.txt)
   set (CPACK_PACKAGE_VERSION_MAJOR ${libMeshb_VERSION_MAJOR})
   set (CPACK_PACKAGE_VERSION_MINOR ${libMeshb_VERSION_MINOR})
   set(CPACK_COMPONENTS_ALL applications examples libraries headers)
   include (CPack)
endif ()


##################################
# EXPORT CMAKE PACKAGE INFORMATION
##################################

include(CMakePackageConfigHelpers)
file(WRITE ${PROJECT_BINARY_DIR}/${PROJECT_NAME}Config.cmake
"include(\${CMAKE_CURRENT_LIST_DIR}/meshb-target.cmake)
include(\${CMAKE_CURRENT_LIST_DIR}/libMeshb-target.cmake)
set(libMeshb_INCLUDE_DIRS ${CMAKE_INSTALL_PREFIX}/include)
set(libMeshb_LINK_DIRS ${CMAKE_INSTALL_PREFIX}/lib)
set(libMeshb_LIBRARIES Meshb.7 ${AIO_LIBRARIES})
set(libMeshb_Fortran_LIBRARIES Meshbf.7 ${AIO_LIBRARIES})
set(libMeshb_FOUND TRUE)
")
write_basic_package_version_file(
   "${PROJECT_BINARY_DIR}/${PROJECT_NAME}ConfigVersion.cmake"
   VERSION ${PROJECT_VERSION}
   COMPATIBILITY SameMajorVersion)
install(FILES
   "${PROJECT_BINARY_DIR}/${PROJECT_NAME}Config.cmake"
   "${PROJECT_BINARY_DIR}/${PROJECT_NAME}ConfigVersion.cmake"
   DESTINATION lib/cmake/${PROJECT_NAME})
