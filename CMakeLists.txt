cmake_minimum_required(VERSION 3.1)

project(rme)

if(NOT CMAKE_BUILD_TYPE)
	set(CMAKE_BUILD_TYPE RelWithDebInfo)
endif()
message("Detected Build Type: ${CMAKE_BUILD_TYPE}")

if(DEFINED ENV{VCPKG_ROOT} AND NOT DEFINED CMAKE_TOOLCHAIN_FILE)
	set(CMAKE_TOOLCHAIN_FILE "$ENV{VCPKG_ROOT}/scripts/buildsystems/vcpkg.cmake" CACHE STRING "")
endif()
message("Detected Toolchain: ${CMAKE_TOOLCHAIN_FILE}")

message("Detected compiler: ${CMAKE_CXX_COMPILER_ID}")
if (CMAKE_CXX_COMPILER_ID STREQUAL "MSVC")
	set(CMAKE_CXX_FLAGS "/EHsc /O2 /Ob2 /Oi /Ot /Os /DNDEBUG /D__RELEASE__ /DWXUSINGDLL /DWXMSVC_VERSION_AUTO /D_CRT_SECURE_NO_WARNINGS /MD /Gy /W3 /Zi /MP /GS- /Zm114")
	set(CMAKE_EXE_LINKER_FLAGS "/LIBPATH:comctl32.lib /LIBPATH:WS2_32.lib /DEBUG /SUBSYSTEM:WINDOWS /LARGEADDRESSAWARE /OPT:REF /OPT:ICF")
	# set(CMAKE_CXX_FLAGS_DEBUG "/Od /Zi /D__DEBUG__")
	# set(CMAKE_CXX_FLAGS_MINSIZEREL "/O1 /DNDEBUG")
	# set(CMAKE_CXX_FLAGS_RELEASE "/O2 /DNDEBUG")
	# set(CMAKE_CXX_FLAGS_RELWITHDEBINFO "/Od /Zi")
elseif (CMAKE_CXX_COMPILER_ID STREQUAL "GNU")
	set(CMAKE_CXX_FLAGS "-Wall -Wextra  -Wno-unused-parameter -Wno-unused-variable -Wno-deprecated-declarations -Wno-overloaded-virtual -Wno-strict-aliasing -Wno-sign-compare -Wno-unused-function -Wunused-result")
	set(CMAKE_CXX_FLAGS_DEBUG          "-O0 -g -D__DEBUG__")
	set(CMAKE_CXX_FLAGS_MINSIZEREL     "-Os -DNDEBUG")
	set(CMAKE_CXX_FLAGS_RELEASE        "-O4 -DNDEBUG")
	set(CMAKE_CXX_FLAGS_RELWITHDEBINFO "-O2 -g")
endif()

find_package(OpenGL REQUIRED)

if(APPLE)
    set(CMAKE_PREFIX_PATH /usr/local/opt/libarchive)
endif()
find_package(LibArchive REQUIRED)

if(WIN32)
    set(Boost_THREADAPI win32)
endif()
find_package(Boost 1.34.0 COMPONENTS thread system REQUIRED)

find_package(wxWidgets REQUIRED COMPONENTS gl core base html aui adv net )

find_package(GLUT REQUIRED)
find_package(ZLIB REQUIRED)

include(${wxWidgets_USE_FILE})
include(${CMAKE_CURRENT_LIST_DIR}/source/CMakeLists.txt)

if (MSVC)
	add_executable(rme ${rme_H} ${rme_SRC} ${CMAKE_CURRENT_LIST_DIR}/vcproj/Project/RME.rc)
else()
	add_executable(rme ${rme_H} ${rme_SRC})
endif()

set_target_properties(rme PROPERTIES CXX_STANDARD 17)
set_target_properties(rme PROPERTIES CXX_STANDARD_REQUIRED ON)

include_directories(${Boost_INCLUDE_DIRS} ${LibArchive_INCLUDE_DIRS} ${OPENGL_INCLUDE_DIR} ${GLUT_INCLUDE_DIRS} ${ZLIB_INCLUDE_DIR})
target_link_libraries(rme ${wxWidgets_LIBRARIES} ${Boost_LIBRARIES} ${LibArchive_LIBRARIES} ${OPENGL_LIBRARIES} ${GLUT_LIBRARIES} ${ZLIB_LIBRARIES})
