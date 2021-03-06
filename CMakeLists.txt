# requires minimum cmake version
cmake_minimum_required(VERSION 3.5)

# produce the cmake var PROJECT_NAME
project(point-cloud-read)

# set some names
set(EXEC_TARGET_NAME "point-cloud-read")
set(${EXEC_TARGET_NAME}_THRIFT src/idl.thrift)
set(${EXEC_TARGET_NAME}_SRC src/main.cpp)

# create a variable for the list of required YARP modules
# this is needed since find_package(ICUBcontrib) overrides the YARP_LIBRARIES set by the first call to find_package(YARP)
set(${EXEC_TARGET_NAME}_YARP_LIBRARIES)

# mandatory use of these packages
find_package(YARP 3.0.101 REQUIRED COMPONENTS 
                                            math 
                                            OS 
                                            sig
                                            dev 
                                            pcl 
                                            idl_tools
)
list(APPEND ${EXEC_TARGET_NAME}_YARP_LIBRARIES ${YARP_LIBRARIES})

find_package(ICUBcontrib REQUIRED)
list(APPEND ${EXEC_TARGET_NAME}_YARP_LIBRARIES ${YARP_LIBRARIES})
list(REMOVE_DUPLICATES ${EXEC_TARGET_NAME}_YARP_LIBRARIES)

# add PCL library
include_directories(${PCL_INCLUDE_DIRS})
link_directories(${PCL_LIBRARY_DIRS})
add_definitions(${PCL_DEFINITIONS})

# Fix some PCL bug related to Ubuntu
list(REMOVE_ITEM PCL_LIBRARIES "vtkproj4")

# extend the current search path used by cmake to load helpers
list(APPEND CMAKE_MODULE_PATH ${ICUBCONTRIB_MODULE_PATH})

# helpers defining certain macros
include(ICUBcontribHelpers)
include(ICUBcontribOptions)

# specify to install in $ICUBcontrib_DIR/bin
icubcontrib_set_default_prefix()

# generate idl source and header
yarp_add_idl(${EXEC_TARGET_NAME}_THRIFT_SRC ${${EXEC_TARGET_NAME}_THRIFT})

# set the executable 
add_executable(${EXEC_TARGET_NAME} 
                                    ${${EXEC_TARGET_NAME}_SRC}
                                    ${${EXEC_TARGET_NAME}_THRIFT} 
                                    ${${EXEC_TARGET_NAME}_THRIFT_SRC}
)

target_link_libraries(${EXEC_TARGET_NAME} 
                                            ${${EXEC_TARGET_NAME}_YARP_LIBRARIES}
                                            ${PCL_LIBRARIES}
)

# install the project executable
install(TARGETS ${EXEC_TARGET_NAME} DESTINATION bin)

# generate ad-hoc project to perform "make uninstall"
icubcontrib_add_uninstall_target()

# installation of YARP application related files
file(GLOB scripts ${CMAKE_SOURCE_DIR}/app/scripts/*.xml)
yarp_install(FILES ${scripts} DESTINATION ${ICUBCONTRIB_APPLICATIONS_INSTALL_DIR})
