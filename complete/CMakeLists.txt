cmake_minimum_required (VERSION 2.6)
project ( complete )

#grab all the cpp files from the root directory
file( GLOB_RECURSE SOURCES "*.cpp" )

option( LLVM_INCLUDE_DIRS "Location of the clang include location" )
option( LLVM_LIBRARY_DIRS "Location of the clang lib location" )

add_library(${PROJECT_NAME} SHARED  ${SOURCES})

if( NOT CLANG_LIB_DIR )

	set(CLANG_FIND_REQUIRED "yes" )

	include( "FindLLVM.cmake" )
	include( "FindClang.cmake" )

endif()

get_cmake_property(_variableNames VARIABLES)
foreach (_variableName ${_variableNames})
    #message(STATUS "${_variableName}=${${_variableName}}")
endforeach()

include_directories(${CLANG_INCLUDE_DIRS})

include(CheckCXXCompilerFlag)
CHECK_CXX_COMPILER_FLAG("-std=c++11" COMPILER_SUPPORTS_CXX11)
CHECK_CXX_COMPILER_FLAG("-std=c++0x" COMPILER_SUPPORTS_CXX0X)
if(COMPILER_SUPPORTS_CXX11)
    set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -std=c++11")
elseif(COMPILER_SUPPORTS_CXX0X)
    set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -std=c++0x")
else()
        message(STATUS "The compiler ${CMAKE_CXX_COMPILER} has no C++11 support. Please use a different C++ compiler.")
endif()

if(${WIN32})
	set(LIB_SUFFIX ".lib" )
else()
	set(LIB_SUFFIX ".a" )
endif()

set(LINK_LIBS 
		"${CLANG_LIB_DIRS}/libclang${LIB_SUFFIX}"
	)

set_target_properties( ${PROJECT_NAME}
    PROPERTIES
    LIBRARY_OUTPUT_DIRECTORY "${CMAKE_SOURCE_DIR}/"
)
add_definitions( ${CLANG_DEFINITIONS} )
target_link_libraries(${PROJECT_NAME} "-L${CLANG_LIB_DIRS}" "-lclang" "-Wl,-rpath=${CLANG_LIB_DIRS}" )
