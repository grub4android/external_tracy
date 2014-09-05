# common
cmake_minimum_required(VERSION 2.8)
project(tracy)
set(CMAKE_BUILD_TYPE Release)

# get architecture
execute_process( COMMAND ${CMAKE_C_COMPILER} -dumpmachine COMMAND cut -d- -f1 COMMAND tr -d '\n' OUTPUT_VARIABLE ARCHITECTURE )
if("${ARCHITECTURE}" STREQUAL "arm")
  set(ARCH_HEADER "eabi")
  set(ARCH_PATH "arm")
endif()

# generate syscall header
execute_process(
	COMMAND ${CMAKE_SOURCE_DIR}/gen_syscall_header ${ARCH_HEADER} ${CMAKE_C_COMPILER} ${CMAKE_BINARY_DIR}/include
	OUTPUT_VARIABLE STDOUT
)

# common cflags
set(CMAKE_C_FLAGS "-ansi -pedantic -Wall -Wextra -Werror -Wno-unused-result -pipe")
set(CMAKE_C_FLAGS_RELEASE "-O2 -fomit-frame-pointer")
set(CMAKE_C_FLAGS_DEBUG  "-ggdb")
set(CMAKE_C_FLAGS "-fPIC -D_POSIX_SOURCE -D_GNU_SOURCE -D_POSIX_C_SOURCE=199309L")
if($ENV{STATIC_COMPILE})
	set(CMAKE_C_FLAGS "-static-libgcc -Wl,-static")
endif()

# tracy defines
set(TRACY_SOURCES
	tracy.c
	tracy-mem.c
	tracy-safe-fork.c
	tracy-modification.c
	tracy-event.c
	ll.c
	trampy.c
	arch/${ARCH_PATH}/arch.c
)
set(TRACY_FLAGS "-D_POSIX_SOURCE -D_GNU_SOURCE -D_POSIX_C_SOURCE=199309L")
set(TRACY_INCLUDES ${CMAKE_BINARY_DIR}/include)

if($ENV{STATIC_COMPILE})
# static
add_library(tracy STATIC ${TRACY_SOURCES})
else()
# shared
add_library(tracy SHARED ${TRACY_SOURCES})
endif()

set_target_properties(tracy PROPERTIES COMPILE_FLAGS ${TRACY_FLAGS})
set_property(TARGET tracy PROPERTY INCLUDE_DIRECTORIES ${TRACY_INCLUDES})
target_link_libraries(tracy pthread)

# tests
#add_executable(fork tests/fork.c)
#target_link_libraries(fork tracy)
#target_link_libraries(fork pthread)
