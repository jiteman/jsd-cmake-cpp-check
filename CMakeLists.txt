cmake_minimum_required( VERSION 3.7.1 )

project( cmake_cpp_check C CXX )


include( CheckFunctionExists )
include( CheckLibraryExists )
include( CheckIncludeFile )
include( CheckIncludeFiles )
include( CheckSymbolExists )
include( CheckCCompilerFlag )
include( CheckCXXCompilerFlag )
include( CheckCSourceCompiles )
include( CheckCXXSourceCompiles )
include( CheckTypeSize )
include( CheckStructHasMember )
include( GNUInstallDirs )


## include dependencies
set( ${PROJECT_NAME}_DEPENDENCY_DIRS_VAR
	sources )
list( REMOVE_DUPLICATES ${PROJECT_NAME}_DEPENDENCY_DIRS_VAR )
include_directories( ${${PROJECT_NAME}_DEPENDENCY_DIRS_VAR} )

include_directories( ${CMAKE_BINARY_DIR} ) # for generated configuration file

message( SEND_STATUS "[JSD - CCC] CMAKE_C_FLAGS: ${CMAKE_C_FLAGS}" )

# configuration file
IF(NOT WIN32)
    # We need pthreads outside of Windows, for semaphores. It's also used to
    # set the priority and name of threads, when possible.
    CHECK_INCLUDE_FILE(pthread.h HAVE_PTHREAD_H)
    IF(NOT HAVE_PTHREAD_H)
        MESSAGE(FATAL_ERROR "PThreads is required for non-Windows builds!")
    ENDIF()

    CHECK_C_COMPILER_FLAG(-pthread HAVE_PTHREAD)
    IF(HAVE_PTHREAD)
        SET(CMAKE_REQUIRED_FLAGS "${CMAKE_REQUIRED_FLAGS} -pthread")
        SET(C_FLAGS ${C_FLAGS} -pthread)
        SET(LINKER_FLAGS ${LINKER_FLAGS} -pthread)
    ENDIF()

    CHECK_LIBRARY_EXISTS(pthread pthread_create "" HAVE_LIBPTHREAD)
    IF(HAVE_LIBPTHREAD)
        SET(EXTRA_LIBS pthread ${EXTRA_LIBS})
    ENDIF()

    CHECK_SYMBOL_EXISTS(pthread_setschedparam pthread.h HAVE_PTHREAD_SETSCHEDPARAM)

    # Some systems need pthread_np.h to get pthread_setname_np
    CHECK_INCLUDE_FILES("pthread.h;pthread_np.h" HAVE_PTHREAD_NP_H)
    IF(HAVE_PTHREAD_NP_H)
        CHECK_SYMBOL_EXISTS(pthread_setname_np "pthread.h;pthread_np.h" HAVE_PTHREAD_SETNAME_NP)
        IF(NOT HAVE_PTHREAD_SETNAME_NP)
            CHECK_SYMBOL_EXISTS(pthread_set_name_np "pthread.h;pthread_np.h" HAVE_PTHREAD_SET_NAME_NP)
        ELSE()
            CHECK_C_SOURCE_COMPILES("
#include <pthread.h>
#include <pthread_np.h>
int main()
{
    pthread_setname_np(\"testname\");
    return 0;
}"
                PTHREAD_SETNAME_NP_ONE_PARAM
            )
            CHECK_C_SOURCE_COMPILES("
#include <pthread.h>
#include <pthread_np.h>
int main()
{
    pthread_setname_np(pthread_self(), \"%s\", \"testname\");
    return 0;
}"
                PTHREAD_SETNAME_NP_THREE_PARAMS
            )
        ENDIF()
    ELSE()
        CHECK_SYMBOL_EXISTS(pthread_setname_np pthread.h HAVE_PTHREAD_SETNAME_NP)
        IF(NOT HAVE_PTHREAD_SETNAME_NP)
            CHECK_SYMBOL_EXISTS(pthread_set_name_np pthread.h HAVE_PTHREAD_SET_NAME_NP)
        ELSE()
            CHECK_C_SOURCE_COMPILES("
#include <pthread.h>
int main()
{
    pthread_setname_np(\"testname\");
    return 0;
}"
                PTHREAD_SETNAME_NP_ONE_PARAM
            )
            CHECK_C_SOURCE_COMPILES("
#include <pthread.h>
int main()
{
    pthread_setname_np(pthread_self(), \"%s\", \"testname\");
    return 0;
}"
                PTHREAD_SETNAME_NP_THREE_PARAMS
            )
        ENDIF()
    ENDIF()
ENDIF()

if( NOT WIN32 )
    # We need pthreads outside of Windows, for semaphores. It's also used to
    # set the priority and name of threads, when possible.
    check_include_files( pthread.h JSD_CCC_HAVE_PTHREAD_H )
    if( NOT JSD_CCC_HAVE_PTHREAD_H )
        message( FATAL_ERROR "PThreads is required for non-Windows builds!" )
	endif()

    CHECK_C_COMPILER_FLAG( -pthread JSD_CCC_HAVE_PTHREAD )
    if( JSD_CCC_HAVE_PTHREAD )
        set( CMAKE_REQUIRED_FLAGS "${CMAKE_REQUIRED_FLAGS} -pthread" )
        set( C_FLAGS ${C_FLAGS} -pthread )
        set( LINKER_FLAGS ${LINKER_FLAGS} -pthread )
    endif()

    CHECK_LIBRARY_EXISTS( pthread pthread_create "" JSD_CCC_HAVE_LIBPTHREAD )
    if( JSD_CCC_HAVE_LIBPTHREAD )
        set( EXTRA_LIBS pthread ${EXTRA_LIBS} )
	endif()

	message( SEND_STATUS "[JSD - CCC] CMAKE_C_FLAGS: ${CMAKE_C_FLAGS}" )

    CHECK_SYMBOL_EXISTS( pthread_setschedparam pthread.h JSD_CCC_HAVE_PTHREAD_SETSCHEDPARAM )

    # Some systems need pthread_np.h to get pthread_setname_np
    CHECK_INCLUDE_FILES( "pthread.h; pthread_np.h" JSD_CCC_HAVE_PTHREAD_NP_H )
    if( JSD_CCC_HAVE_PTHREAD_NP_H )
        CHECK_SYMBOL_EXISTS( pthread_setname_np "pthread.h; pthread_np.h" JSD_CCC_HAVE_PTHREAD_SETNAME_NP )
        if( NOT JSD_CCC_HAVE_PTHREAD_SETNAME_NP )
            CHECK_SYMBOL_EXISTS( pthread_set_name_np "pthread.h; pthread_np.h" JSD_CCC_HAVE_PTHREAD_SET_NAME_NP )
        else()
            CHECK_C_SOURCE_COMPILES( "
#include <pthread.h>
#include <pthread_np.h>
int main()
{
    pthread_setname_np(\"testname\");
    return 0;
}"
                JSD_CCC_HAVE_PTHREAD_SETNAME_NP_WITH_ONE_PARAM
            )
            CHECK_C_SOURCE_COMPILES("
#include <pthread.h>
#include <pthread_np.h>
int main()
{
    pthread_setname_np(pthread_self(), \"%s\", \"testname\");
    return 0;
}"
                JSD_CCC_HAVE_PTHREAD_SETNAME_NP_WITH_THREE_PARAMS
            )
        endif()
    else()
        CHECK_SYMBOL_EXISTS( pthread_setname_np "pthread.h" JSD_CCC_HAVE_PTHREAD_SETNAME_NP )
        if( NOT JSD_CCC_HAVE_PTHREAD_SETNAME_NP )
			CHECK_SYMBOL_EXISTS( pthread_set_name_np "pthread.h" JSD_CCC_HAVE_PTHREAD_SET_NAME_NP )
        else()
			message( FATAL_ERROR "[JSD - CCC] test" )
            CHECK_C_SOURCE_COMPILES("
#include <pthread.h>
int main()
{
    pthread_setname_np(\"testname\");
    return 0;
}"
                JSD_CCC_HAVE_PTHREAD_SETNAME_NP_WITH_ONE_PARAM
            )
            CHECK_C_SOURCE_COMPILES("
#include <pthread.h>
int main()
{
    pthread_setname_np(pthread_self(), \"%s\", \"testname\");
    return 0;
}"
                JSD_CCC_HAVE_PTHREAD_SETNAME_NP_WITH_THREE_PARAMS
            )
        endif()
    endif()
endif()


configure_file(
	"${CMAKE_SOURCE_DIR}/cmake_cpp_check_config.h.in"
	"${CMAKE_BINARY_DIR}/cmake_cpp_check_config.h" )


## this project headers and sources enumeration section
file( GLOB_RECURSE header_and_source_files ${PROJECT_SOURCE_DIR}/sources/*.h* ${PROJECT_SOURCE_DIR}/sources/*.c* )
add_library( ${PROJECT_NAME} SHARED ${header_and_source_files} "${CMAKE_BINARY_DIR}/cmake_cpp_check_config.h" )


## definition section
set( ${PROJECT_NAME}_BUILD_DEFINITIONS_VAR
	"" )
set( ${PROJECT_NAME}_LINK_DEFINITIONS_VAR
	"" )
add_definitions( "${${PROJECT_NAME}_BUILD_DEFINITIONS_VAR}" )


## required to be able to be linked into shared libraries.
set_target_properties( ${PROJECT_NAME} PROPERTIES POSITION_INDEPENDENT_CODE ON )
set_target_properties( ${PROJECT_NAME} PROPERTIES VERSION 0.1.0 )
set_target_properties( ${PROJECT_NAME} PROPERTIES SOVERSION 0.1 )


## project target section
if ( UNIX )
	set( ${PROJECT_NAME}_DEPENDENCY_LIBS_VAR
		pthread )
elseif( WINDOWS )
	set( ${PROJECT_NAME}_DEPENDENCY_LIBS_VAR
		"" )
endif()

list( REMOVE_DUPLICATES ${PROJECT_NAME}_DEPENDENCY_LIBS_VAR )
target_link_libraries( ${PROJECT_NAME} ${${PROJECT_NAME}_DEPENDENCY_LIBS_VAR} )
