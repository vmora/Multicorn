cmake_minimum_required(VERSION 3.4)
project(multicorn)

#set(CMAKE_FIND_LIBRARY_PREFIXES "")
#set(CMAKE_FIND_LIBRARY_SUFFIXES ".lib")



exec_program(pg_config ARGS --libdir
    OUTPUT_VARIABLE POSTGRES_LIBRARY_DIR)
find_library(POSTGRESQL_LIBRARIES NAMES postgres pq
	PATHS ${POSTGRES_LIBRARY_DIR}
	)
exec_program(pg_config ARGS --includedir-server
    OUTPUT_VARIABLE POSTGRESQL_SERVER_INCLUDE_DIR)
exec_program(pg_config ARGS --includedir
    OUTPUT_VARIABLE POSTGRESQL_INCLUDE_DIR)
if (MSVC)
	include_directories(
		${POSTGRESQL_SERVER_INCLUDE_DIR}/port/win32
		${POSTGRESQL_SERVER_INCLUDE_DIR}/port/win32_msvc
		)
endif()
message("pg confiog ${PG_CONFIG}")
message("pg_include ${POSTGRESQL_SERVER_INCLUDE_DIR}")
message("postgres lib ${POSTGRESQL_LIBRARIES}")
find_package(PythonInterp REQUIRED)
find_package(PythonLibs REQUIRED)


include_directories(
    ${POSTGRESQL_INCLUDE_DIR} 
    ${POSTGRESQL_SERVER_INCLUDE_DIR} 
    ${PYTHON_INCLUDE_DIRS})
if (MSVC)
  # make sure a pdb file is generated whatever the build type is
  set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} /Zi")
  set(CMAKE_SHARED_LINKER_FLAGS "${CMAKE_SHARED_LINKER_FLAGS} /DEBUG /OPT:REF /OPT:ICF")
endif()

add_library(multicorn SHARED src/errors.c src/python.c src/query.c src/multicorn.c src/utils.c)
target_link_libraries(multicorn ${POSTGRESQL_LIBRARIES} ${PYTHON_LIBRARIES})



file(READ multicorn.control MULTICORN_CONTROL)
string(REGEX MATCH "default_version *= *'([0-9]+.[0-9]+.[0-9]+)'" EXTVERSION ${MULTICORN_CONTROL})
string(REGEX REPLACE ".*default_version *= *'(.*)'.*" "\\1" MULTICORN_VERSION ${EXTVERSION})

configure_file(sql/multicorn.sql multicorn--${MULTICORN_VERSION}.sql COPYONLY)
configure_file(multicorn.control multicorn.control COPYONLY)


# "c:\Program Files\CMake\bin\cmake.exe" .. -DPG_LIBRARY_DIRS="C:\Program Files\PostgreSQL\9.6\bin";"C:\Program Files\PostgreSQL\9.6\lib" -DPG_INCLUDE_DIRS="C:\Program Files\PostgreSQL\9.6\include\server";"C:\Program Files\PostgreSQL\9.6\include";"C:\Program Files\PostgreSQL\9.6\include\server\port\win32";"C:\Program Files\PostgreSQL\9.6\include\server\port\win32_msvc" -DPYTHON_INCLUDE_DIRS=C:\edb\languagepack-9.6\x64\Python-3.3\Include -DPYTHON_LIBRARY_DIRS=C:\edb\languagepack-9.6\x64\Python-3.3\libs -G "Visual Studio 14 2015 Win64"
# "c:\Program Files\CMake\bin\cmake.exe" --build . --config Release
