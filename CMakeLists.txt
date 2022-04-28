cmake_minimum_required (VERSION 3.4)
project (popSTR CXX)

#####################################################
# data setup
#####################################################

add_custom_command(OUTPUT ${CMAKE_BINARY_DIR}/data_created
    COMMAND mkdir -p ${CMAKE_BINARY_DIR}/markerInfo
    COMMAND for i in `/usr/bin/seq 1 22` \; do gunzip < ${CMAKE_CURRENT_SOURCE_DIR}/chr$\${i}markerInfo.gz > ${CMAKE_BINARY_DIR}/markerInfo/chr$\${i}markerInfo \; done \;
    COMMAND cp -a ${CMAKE_CURRENT_SOURCE_DIR}/longRepeats ${CMAKE_BINARY_DIR}/markerInfo/
    COMMAND cp -a ${CMAKE_CURRENT_SOURCE_DIR}/defaultModel ${CMAKE_BINARY_DIR}/markerInfo/
    COMMAND tar -C ${CMAKE_BINARY_DIR} -xvf ${CMAKE_CURRENT_SOURCE_DIR}/kernel.tar.gz > /dev/null
    COMMAND tar -C ${CMAKE_BINARY_DIR} -xvf ${CMAKE_CURRENT_SOURCE_DIR}/panelMarkerInfo.tar.gz > /dev/null
    COMMAND touch ${CMAKE_BINARY_DIR}/data_created
    COMMENT "Creating data files (this may take a moment).")

add_custom_target(create_data DEPENDS ${CMAKE_BINARY_DIR}/data_created)

#####################################################
# find requirements
#####################################################

find_package(ZLIB REQUIRED)

find_package(Boost REQUIRED COMPONENTS iostreams)

if (DEFINED SEQANHTS_DIR)
    list(APPEND MY_INCLUDES "${SEQANHTS_DIR}/include")
    list(APPEND MY_CXXFLAGS -DSEQAN_ENABLE_TESTING=0 -DSEQAN_ENABLE_DEBUG=0 -DSEQAN_HAS_ZLIB=1)
else ()
    message(FATAL_ERROR "You need to set SEQANHTS_DIR to the path of SeqAnHTS.")
endif ()

find_package (Threads)

add_custom_command(OUTPUT liblinear-2.01/README
    COMMAND unzip -qq ${CMAKE_SOURCE_DIR}/liblinear-2.01.zip -d ${CMAKE_BINARY_DIR})
add_custom_target (liblinear DEPENDS liblinear-2.01/README)
list(APPEND MY_INCLUDES "${CMAKE_SOURCE_DIR}")
list(APPEND MY_INCLUDES "${CMAKE_BINARY_DIR}")

#####################################################
# misc flags
#####################################################

#TODO -Wall -Wextra
list(APPEND MY_CXXFLAGS "-Wmaybe-uninitialized")

#####################################################
# define targets
#####################################################

add_executable(popSTR popSTR.cpp liblinear.cpp msGenotyper.cpp computePnSlippage.cpp computeReadAttributes.cpp computePnSlippageDefault.cpp msGenotyperDefault.cpp)

add_dependencies(popSTR create_data liblinear)

target_include_directories(popSTR PRIVATE ${MY_INCLUDES})
target_compile_options(popSTR PRIVATE ${MY_CXXFLAGS})
target_link_libraries (popSTR PRIVATE Threads::Threads ZLIB::ZLIB Boost::iostreams )
target_link_libraries (popSTR PRIVATE hts)

#TODO raise standard
set_property(TARGET popSTR PROPERTY CXX_STANDARD 14)

#####################################################
# copy scripts
#####################################################

configure_file(runSmall.sh runSmall.sh COPYONLY)
