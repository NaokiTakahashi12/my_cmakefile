cmake_minimum_required(VERSION 3.10)

if(NOT PROJECT_NAME)
	get_filename_component(DEFAULT_PROJECT_NAME ${CMAKE_CURRENT_SOURCE_DIR} NAME_WE)
	set(PROJECT_NAME ${DEFAULT_PROJECT_NAME} CACHE STRING "Project name" FORCE)
endif(NOT PROJECT_NAME)

if(NOT CMAKE_BUILD_TYPE)
	set(CMAKE_BUILD_TYPE RelWithDebInfo CACHE STRING "Build type" FORCE)
endif(NOT CMAKE_BUILD_TYPE)

project(${PROJECT_NAME} CXX C)

set(CMAKE_CXX_STANDARD 17)

if(NOT CMAKE_VERBOSE_MAKEFILE)
	set(CMAKE_VERBOSE_MAKEFILE false CACHE BOOL "Verbose option" FORCE)
endif(NOT CMAKE_VERBOSE_MAKEFILE)

set(CMAKE_CXX_FLAGS_DEBUG "${CMAKE_CXX_FLAGS_DEBUG} -Winline -Wextra -Wall -g -O0")

if(NOT USE_GCOV)
	set(USE_GCOV CACHE BOOL "GCOV" FORCE)
endif(NOT USE_GCOV)

if(NOT USE_BOOST)
	set(USE_BOOST false CACHE BOOL "Find and link Boost" FORCE)
endif(NOT USE_BOOST)

if(NOT USE_CV)
	set(USE_CV false CACHE BOOL "Find and link OpenCV" FORCE)
endif(NOT USE_CV)

if(NOT USE_EIGEN3)
	set(USE_EIGEN3 false CACHE BOOL "Find and link Eigen3" FORCE)
endif(NOT USE_EIGEN3)

if(NOT USE_ICE)
	set(USE_ICE false CACHE BOOL "Find and link Zeroc Ice" FORCE)
endif(NOT USE_ICE)

if(NOT USE_ODE)
	set(USE_ODE false CACHE BOOL "Find and link Open Dynamics Engine" FORCE)
endif(NOT USE_ODE)

if(NOT USE_SSH)
	set(USE_SSH false CACHE BOOL "Find and link OpenSSH" FORCE)
endif(NOT USE_SSH)

if(NOT USE_GL)
	set(USE_GL false CACHE BOOL "Find and link OpenGLW" FORCE)
endif(NOT USE_GL)

if(USE_GCOV)
	set(CMAKE_CXX_FLAGS_DEBUG "${CMAKE_CXX_FLAGS_DEBUG} -fprofile-arcs -ftest-coverage")
endif(USE_GCOV)

if(NOT EXECUTABLE_OUTPUT_PATH)
	set(EXECUTABLE_OUTPUT_PATH bin CACHE PATH "Out file insert directory" FORCE)
endif()

if(NOT INCLUDE_DIR)
	set(INCLUDE_DIR include CACHE PATH "Include directory" FORCE)
endif()

if(NOT SOURCE_DIR)
	set(SOURCE_DIR src CACHE PATH "Source directory" FORCE)
endif()

if(NOT EXISTS ${EXECUTABLE_OUTPUT_PATH})
	message(FATAL_ERROR "Not found executable path")
endif()

if(NOT EXISTS ${INCLUDE_DIR})
	message(FATAL_ERROR "Not found include path")
endif()

if(NOT EXISTS ${SOURCE_DIR})
	message(FATAL_ERROR "Not found source path")
endif()

if(USE_GL)
	find_package(OpenGL REQUIRED)

	if(OpenGL_FOUND)
		message(" ---- Found OpenGL ${OpenGL_INCLUDE_DIRS}")
		include_directories(${OpenGL_INCLUDE_DIRS})

	else()
		message(" ---- Not Found OpenGL")

	endif()
endif(USE_GL)

if(USE_SSH)
	if(APPLE)
		set(LIBSSH_TOP_DIR /usr/local)

	elseif(UNIX)
		set(LIBSSH_TOP_DIR /usr)

	endif()
	set(libssh_INCLUDE_DIRS ${LIBSSH_TOP_DIR}/include)
	set(libssh_LIBRARIES_DIR ${LIBSSH_TOP_DIR}/lib)

	if(EXISTS ${libssh_INCLUDE_DIRS} AND EXISTS ${libssh_LIBRARIES_DIR})
		message(" ---- Found libssh ${libssh_INCLUDE_DIRS}")
		include_directories(${libssh_INCLUDE_DIRS})
		file(GLOB libssh_LIBRARIES "${libssh_LIBRARIES_DIR}/libssh*.dylib")

	else()
		message(" ---- Not Found libssh Top Directories")

	endif()
endif(USE_SSH)


if(USE_ODE)
	if(APPLE)
		set(ODE_TOP_DIRECTORIES /usr/local/Cellar/ode-drawstuff/0.13/)

	elseif(UNIX)
		set(ODE_TOP_DIRECTORIES /usr/local)

	endif()
	set(ODE_INCLUDE_DIR ${ODE_TOP_DIRECTORIES}/include)
	set(ODE_LIBRARIES_DIR ${ODE_TOP_DIRECTORIES}/lib)

	if(EXISTS ${ODE_INCLUDE_DIR} AND EXISTS ${ODE_LIBRARIES_DIR})
		message(" ---- Found Ode ${ODE_INCLUDE_DIR}")
		include_directories(${ODE_INCLUDE_DIR})
		file(GLOB ODE_LIBRARIES "${ODE_LIBRARIES_DIR}/*.a")

	else()
		message(" ---- Not Found Eigen3 Top Directories")

	endif()
endif(USE_ODE)

if(USE_CV)
	find_package(OpenCV REQUIRED)

	if(OpenCV_FOUND)
		message(" ---- Found OpenCV ${OpenCV_INCLUDE_DIRS}")
		include_directories(${OpenCV_INCLUDE_DIRS})

	else(OpenCV_FOUND)
		message(" ---- Not Found OpenCV")

	endif(OpenCV_FOUND)
endif(USE_CV)

if(USE_BOOST)
	find_package(Boost	REQUIRED COMPONENTS thread system)

	if(Boost_FOUND)
		message(" ---- Found Boost ${Boost_INCLUDE_DIRS}")
		include_directories(${Boost_INCLUDE_DIRS})

	else(Boost_FOUND)
		message(" ---- Not Found Boost")

	endif(Boost_FOUND)
endif(USE_BOOST)

if(USE_EIGEN3)
	if(APPLE)
		find_package(Eigen3	REQUIRED COMPONENTS Dense Core Geometry)

		if(Eigen3_FOUND)
			message(" ---- Found Eigen3 ${EIGEN3_INCLUDE_DIRS}")
			include_directories(${EIGEN3_INCLUDE_DIRS})

		else(Eigen3_FOUND)
			message(" ---- Not Found Eigen3")

		endif(Eigen3_FOUND)
	elseif(UNIX)
		if(EXISTS $ENV{EIGEN3_INCLUDE_DIR})
			message(" ---- Used Unix Eigen3 Dir $ENV{EIGEN3_INCLUDE_DIR}")
			include_directories($ENV{EIGEN3_INCLUDE_DIR})

		else()
			message(" ---- Not Defind of EIGEN3_INCLUDE_DIRS")

		endif()
	endif()
endif(USE_EIGEN3)

if(USE_ICE)
	find_package(Ice REQUIRED COMPONENTS Ice++11 IceBox++11 IceSSL++11 IceGrid++11 IceDiscovery++11)

	if(Ice_FOUND)
		message(" ---- Found Ice ${Ice_INCLUDE_DIRS}")
		include_directories(${Ice_INCLUDE_DIRS})
		add_definitions("-D ICE_CPP11_MAPPING")
		
		if(NOT ICE_DIR)
			set(ICE_DIR ice CACHE PATH "find .ice file" FORCE)
		endif()

		if(EXISTS ${ICE_DIR})
			file(GLOB ICE_FILES "${ICE_DIR}/*.ice")
			set(ICE_OUTPUT_DIRECTORY ice_create_files CACHE PATH "Ice create file out-put directory" FORCE)
			
			if(NOT EXISTS ${Ice_SLICE2CPP_EXECUTABLE})
				message(FATAL_ERROR " ---- Not found ice slice")
			endif()
			if(NOT DEFINED ICE_FILES)
				message(FATAL_ERROR " ---- Not found ice files")
			endif()
			if(NOT EXISTS ${ICE_OUTPUT_DIRECTORY})
				message(FATAL_ERROR " ---- Not found ice output directory")
			endif()

			foreach(ICE_FILE ${ICE_FILES})
				get_filename_component(ICE_CREATE_FILENAME "${ICE_FILE}" NAME_WE)
				set(ICE_OUTPUT_FILES ${ICE_OUTPUT_FILES} ${ICE_OUTPUT_DIRECTORY}/${ICE_CREATE_FILENAME}.cpp ${ICE_OUTPUT_DIRECTORY}/${ICE_CREATE_FILENAME}.h)
			endforeach()

			add_custom_command(OUTPUT ${ICE_OUTPUT_FILES} DEPENDS ${ICE_FILES} COMMAND "${Ice_SLICE2CPP_EXECUTABLE}" ARGS ${ICE_FILES} --output-dir ${ICE_OUTPUT_DIRECTORY})

			foreach(ICE_OUTPUT_FILE ${ICE_OUTPUT_FILES})
				set(SOURCE_FILES ${SOURCE_FILES} ${ICE_OUTPUT_FILE})
			endforeach()

			set(INCLUDE_DIRS ${INCLUDE_DIRS} ${ICE_OUTPUT_DIRECTORY})

		else()
			message(FATAL_ERROR " ---- Not found ice directory")

		endif()
	else(Ice_FOUND)
		message(" ---- Not Found Ice")

	endif(Ice_FOUND)
endif(USE_ICE)

file(GLOB CXX_SOURCE_FILES "${SOURCE_DIR}/*.cpp")
file(GLOB C_SOURCE_FILES "${SOURCE_DIR}/*.c")
set(SOURCE_FILES ${SOURCE_FILES} ${CXX_SOURCE_FILES} ${C_SOURCE_FILES})

set(TARGET_NAME ${PROJECT_NAME}.out)

set(INCLUDE_DIRS ${INCLUDE_DIRS} ${SOURCE_DIR} ${INCLUDE_DIR})
include_directories(${INCLUDE_DIRS})

add_executable(${TARGET_NAME} ${SOURCE_FILES})

target_link_libraries(${TARGET_NAME} stdc++ pthread system m)
target_link_libraries(${TARGET_NAME} python2.7)

if(USE_GL)
	if(APPLE)
		target_link_libraries(${TARGET_NAME} ${OpenGL_LIBRARIES} "-framework OpenGL -lglfw")
	elseif(UNIX)
		target_link_libraries(${TARGET_NAME} ${OpenGL_LIBRARIES} "-lglfw")
	endif()
endif(USE_GL)

if(USE_SSH)
	target_link_libraries(${TARGET_NAME} ${libssh_LIBRARIES})
endif(USE_SSH)

if(USE_ODE)
	if(APPLE)
		target_link_libraries(${TARGET_NAME} ${ODE_LIBRARIES} "-framework AGL -framework OpenGL -framework GLUT")
	elseif(UNIX)
		target_link_libraries(${TARGET_NAME} ${ODE_LIBRARIES} "ode -ldrawstuff -lGL -lGLU -lglut -lX11")
	endif()
endif(USE_ODE)

if(USE_CV)
	target_link_libraries(${TARGET_NAME} ${OpenCV_LIBRARIES})
endif(USE_CV)

if(USE_BOOST)
	target_link_libraries(${TARGET_NAME} ${Boost_LIBRARIES} Boost::thread)
endif(USE_BOOST)

if(USE_ICE)
	target_link_libraries(${TARGET_NAME} ${Ice_LIBRARIES})
endif(USE_ICE)

