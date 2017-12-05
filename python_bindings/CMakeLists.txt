
project(PyHalide)
cmake_minimum_required(VERSION 3.0) # for up-to-date FindPythonLibs.cmake
set( CMAKE_MODULE_PATH ${CMAKE_MODULE_PATH} ${CMAKE_CURRENT_SOURCE_DIR} )

# Find Python and boost python
if(USE_PYTHON EQUAL 2)
	find_package(PythonInterp 2.7 REQUIRED)
	find_package(PythonLibs 2.7 REQUIRED)
	find_package(NumPy REQUIRED)
	if (NOT USE_EXTERNAL_BOOST)
		find_package(Boost COMPONENTS python REQUIRED)
	endif()
    find_package(Halide REQUIRED)
else()
	find_package(PythonInterp 3.4 REQUIRED)
	find_package(PythonLibs 3.4 REQUIRED)
	find_package(NumPy REQUIRED)
	if (NOT USE_EXTERNAL_BOOST)
		string( REGEX REPLACE "([0-9]+).([0-9]+).[0-9.]+" "\\1\\2" PYTHON_CONCAT_VERSION_STRING ${PYTHON_VERSION_STRING} )
		find_package(Boost COMPONENTS python-py${PYTHON_CONCAT_VERSION_STRING})
		if(NOT Boost_FOUND)
			find_package(Boost COMPONENTS python3 REQUIRED)
		endif()
	endif()
    find_package(Halide REQUIRED)
endif()

add_definitions("-std=c++11")

include_directories(
  ${HALIDE_INCLUDE_DIR}
  ${Boost_INCLUDE_DIRS}
  ${PYTHON_INCLUDE_DIRS}
)

link_directories(
  ../build/lib
  ${HALIDE_ROOT_DIR}/lib
  ${Boost_LIBRARY_DIRS}
)


if(FALSE AND UNIX )
	# Disable the pointer-to-function and pointer-to-object warnings
	add_definitions( -Wno-unused-local-typedefs )
	set_property(SOURCE util.cpp PROPERTY COMPILE_FLAGS -w )
endif()

file(GLOB SrcCpp
python/*.cpp
../src/OutputImageParam.cpp
../src/ImageParam.cpp)

add_library(halide SHARED ${SrcCpp})
target_link_libraries(halide
  ${HALIDE_LIBRARIES}
  ${Boost_LIBRARIES}
  ${PYTHON_LIBRARIES}
  boost_numpy
)

set_target_properties( halide PROPERTIES PREFIX "")
if(APPLE)
set_target_properties( halide PROPERTIES SUFFIX ".so" )
endif()
