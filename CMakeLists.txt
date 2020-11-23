cmake_minimum_required(VERSION 3.14)

project(libserialport LANGUAGES C)

if(UNIX AND NOT APPLE)
    set(SOURCES serialport.c linux.c linux_termios.c timing.c)
elseif(APPLE)
    set(SOURCES serialport.c macosx.c timing.c)
elseif(WIN32)
    set(SOURCES serialport.c windows.c timing.c)
endif()

set(HEADERS config.h)

set(CMAKE_INCLUDE_CURRENT_DIR ON)
set(CMAKE_INCLUDE_CURRENT_DIR_IN_INTERFACE ON)

add_library(libserialport ${SOURCES} ${HEADERS})

set(SP_PACKAGE_VERSION_MAJOR 0)
set(SP_PACKAGE_VERSION_MINOR 1)
set(SP_PACKAGE_VERSION_MICRO 1)
set(SP_PACKAGE_VERSION_STRING "${SP_PACKAGE_VERSION_MAJOR}.${SP_PACKAGE_VERSION_MINOR}.${SP_PACKAGE_VERSION_MICRO}")
message("Serialport package version: " ${SP_PACKAGE_VERSION_STRING})

set(SP_LIB_VERSION_CURRENT 1)
set(SP_LIB_VERSION_REVISION 0)
set(SP_LIB_VERSION_AGE 1)
set(SP_LIB_VERSION_STRING "${SP_LIB_VERSION_CURRENT}:${SP_LIB_VERSION_REVISION}:${SP_LIB_VERSION_AGE}")
set(SP_LIB_VERSION_STRING_CLEAN "${SP_LIB_VERSION_CURRENT}.${SP_LIB_VERSION_REVISION}.${SP_LIB_VERSION_AGE}")
message("Serialport lib version: " ${SP_LIB_VERSION_STRING})

### configure config.h

include(CheckSymbolExists)
include(CheckIncludeFile)
include(CheckTypeSize)
include(CheckStructHasMember)

check_symbol_exists(clock_gettime "time.h" HAVE_CLOCK_GETTIME)

check_type_size("BOTHER" BOTHER_SIZE)
if(BOTHER_SIZE)
        set(HAVE_DECL_BOTHER 1)
endif()

CHECK_INCLUDE_FILE("dlfcn.h" HAVE_DLFCN_H)


CHECK_INCLUDE_FILE("sys/file.h" HAVE_SYS_FILE_H)
check_symbol_exists(flock "sys/file.h" HAVE_FLOCK)

check_symbol_exists(realpath "stdlib.h" HAVE_REALPATH)

CHECK_INCLUDE_FILE("inttypes.h" HAVE_INTTYPES_H)
CHECK_INCLUDE_FILE("memory.h" HAVE_MEMORY_H)

CHECK_INCLUDE_FILE("sys/stat.h" HAVE_SYS_STAT_H)
CHECK_INCLUDE_FILE("sys/types.h" HAVE_SYS_TYPES_H)
CHECK_INCLUDE_FILE("unistd.h" HAVE_UNISTD_H)

CHECK_INCLUDE_FILE("stdlib.h" HAVE_STDLIB_H)
CHECK_INCLUDE_FILE("strings.h" HAVE_STRINGS_H)
CHECK_INCLUDE_FILE("string.h" HAVE_STRING_H)

check_struct_has_member("struct termios" c_ispeed linux/termios.h HAVE_STRUCT_TERMIOS_C_ISPEED LANGUAGE C)
check_struct_has_member("struct termios" c_ospeed linux/termios.h HAVE_STRUCT_TERMIOS_C_OSPEED LANGUAGE C)

set(CMAKE_EXTRA_INCLUDE_FILES linux/termios.h)
check_type_size("struct termios2" TERMIOS2_SIZE)
if(TERMIOS2_SIZE)
	set(HAVE_STRUCT_TERMIOS2 1)
	check_struct_has_member("struct termios2" c_ispeed linux/termios.h HAVE_STRUCT_TERMIOS2_C_ISPEED LANGUAGE C)
	check_struct_has_member("struct termios2" c_ospeed linux/termios.h HAVE_STRUCT_TERMIOS2_C_OSPEED LANGUAGE C)
endif()

check_type_size("struct termiox" TERMIOX_SIZE)
if(TERMIOX_SIZE)
	set(HAVE_STRUCT_TERMIOX 1)
endif()

set(CMAKE_EXTRA_INCLUDE_FILES linux/serial.h)
check_type_size("struct serial_struct" SERIAL_STRUCT_SIZE)
if(SERIAL_STRUCT_SIZE)
	set(HAVE_STRUCT_SERIAL_STRUCT 1)
endif()

if(NOT WIN32)
    set(SP_API  "__attribute__((visibility(\"default\")))" )
    set(SP_PRIV "__attribute__((visibility(\"hidden\")))" )
else()
    set(SP_API  "__declspec(dllexport)")
    set(SP_PRIV "")
endif()

configure_file(cmake.config.h.in ${CMAKE_CURRENT_SOURCE_DIR}/config.h)
#this define is needed for config.h inclusion
target_compile_options(libserialport PRIVATE -DLIBSERIALPORT_ATBUILD)

#set properties

if(APPLE)
    find_library(IOKIT IOKit)
    find_library(FOUNDATION Foundation)
    target_link_libraries(libserialport ${IOKIT} ${FOUNDATION})
elseif(WIN32)
    target_link_libraries(libserialport setupapi)
endif()


set_target_properties(libserialport PROPERTIES PUBLIC_HEADER "libserialport.h")
set_target_properties(libserialport PROPERTIES VERSION ${SP_PACKAGE_VERSION_STRING})
set_target_properties(libserialport PROPERTIES POSITION_INDEPENDENT_CODE ON)

#prevent liblibserialport.so
set_target_properties(libserialport PROPERTIES OUTPUT_NAME "serialport") 

install(TARGETS libserialport)

install(TARGETS libserialport
    EXPORT libserialport-config
    )

export(TARGETS libserialport
    FILE "${CMAKE_CURRENT_BINARY_DIR}/libdi2-config.cmake"
    )

install(EXPORT libserialport-config
    DESTINATION "${CMAKE_INSTALL_LIBDIR}/cmake/${CMAKE_PROJECT_NAME}"
    )
