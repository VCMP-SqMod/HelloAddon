cmake_minimum_required(VERSION 3.0.2)
project(SqMod)

# Several plugin options
option(BUILTIN_RUNTIMES "Include the MinGW runtime into the binary itself." ON)
option(FORCE_32BIT_BIN "Create a 32-bit executable binary if the compiler defaults to 64-bit." OFF)
option(ENABLE_API21 "Build for 2.1 API." OFF)

# Default to c++14 standard
if(CMAKE_VERSION VERSION_LESS "3.1")
	if(CMAKE_CXX_COMPILER_ID STREQUAL "GNU")
		set (CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -std=c++14")
	endif()
else()
	# Apparently the above does not work with cmake from on debian 8
	set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -std=c++14")
	# Try the standard method as well
	set(CMAKE_CXX_STANDARD 14)
	set(CMAKE_CXX_STANDARD_REQUIRED ON)
endif()

# Determine if build mode
if(CMAKE_BUILD_TYPE MATCHES Debug)
	add_compile_options(-g)
endif()

# Include mingw runntime into the binary
if(GCC OR MINGW)
	if(BUILTIN_RUNTIMES)
		set(CMAKE_EXE_LINKER_FLAGS "-static-libgcc -static-libstdc++ -static")
	endif()
endif()

# Enable position independent code
if(UNIX)
	set(CMAKE_POSITION_INDEPENDENT_CODE ON)
endif()

# Create the Squirrel module
add_library(SqHello MODULE Module.cpp
	Common.hpp Common.cpp
)

# Determine if build mode
if(CMAKE_BUILD_TYPE MATCHES Release)
	target_compile_definitions(SqHello PRIVATE NDEBUG=1)
else()
	target_compile_definitions(SqHello PRIVATE _DEBUG=1 SQMOD_EXCEPTLOC=1)
endif()

# Don't prefix the module binary.
set_target_properties(SqHello PROPERTIES PREFIX "")
# Custmize module binary name/
if(WIN32)
	if(CMAKE_SIZEOF_VOID_P EQUAL 8 AND NOT FORCE_32BIT_BIN)
		set_target_properties(SqHello PROPERTIES OUTPUT_NAME "mod_hello_64")
	else()
		set_target_properties(SqHello PROPERTIES OUTPUT_NAME "mod_hello_32")
	endif()
else(WIN32)
	if(CMAKE_SIZEOF_VOID_P EQUAL 8 AND NOT FORCE_32BIT_BIN)
		set_target_properties(SqHello PROPERTIES OUTPUT_NAME "mod_hello_64")
	else()
		set_target_properties(SqHello PROPERTIES OUTPUT_NAME "mod_hello_32")
	endif()
endif(WIN32)

# Include SDK library
add_subdirectory(sdk)

# Link to SDK library
target_link_libraries(SqHello SqAddSDK)

# Include current directory in the search path
target_include_directories(SqHello PRIVATE ${CMAKE_CURRENT_LIST_DIR})
