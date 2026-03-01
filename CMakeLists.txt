cmake_minimum_required(VERSION 3.16)
project(softvms C)

if(CMAKE_SOURCE_DIR STREQUAL CMAKE_BINARY_DIR)
  message(
    FATAL_ERROR
      "In-source builds are not allowed.\n"
      "Run cmake from a separate build directory:\n"
      "  cmake -B build && cmake --build build")
endif()

set(CMAKE_C_STANDARD 99)
set(CMAKE_EXPORT_COMPILE_COMMANDS ON)

include(CheckIncludeFile)
include(CheckCSourceCompiles)

find_package(X11 REQUIRED)
find_package(SDL QUIET)

check_include_file(unistd.h HAVE_UNISTD_H)
check_include_file(fcntl.h HAVE_FCNTL_H)
check_include_file(sys/time.h HAVE_SYS_TIME_H)
check_include_file(limits.h HAVE_LIMITS_H)
check_include_file(errno.h HAVE_ERRNO_H)
check_include_file(utime.h HAVE_UTIME_H)

# FindX11 already probes for Xutil.h and exposes X11_Xutil_FOUND; use that
# directly rather than a separate check_include_file probe (which requires
# cache-busting when the include path changes).
if(X11_Xutil_FOUND)
  set(HAVE_X11_XUTIL_H 1)
endif()

# Detect gettimeofday signature (BSD vs POSIX)
check_c_source_compiles(
  "
#include <stdlib.h>
#include <sys/time.h>
int main(void) {
  struct timeval tp;
  struct timezone tz;
  gettimeofday(&tp, &tz);
  return 0;
}"
  BSD_STYLE_GETTIMEOFDAY)

if(BSD_STYLE_GETTIMEOFDAY)
  check_c_source_compiles(
    "
#include <stdlib.h>
#include <sys/time.h>
extern int gettimeofday(struct timeval *, struct timezone *);
int main(void) {
  struct timeval tp; struct timezone tz;
  gettimeofday(&tp, &tz);
  return 0;
}"
    _TZ_STRUCT)
  if(NOT _TZ_STRUCT)
    set(TIMEZONE_IS_VOID 1)
  endif()
endif()

# Main target
set(SRCS src/main.c src/mainwin.c src/cpu.c src/lcdimg.c src/getopt.c)

add_executable(vms ${SRCS})

target_include_directories(
  vms PRIVATE "${CMAKE_SOURCE_DIR}/src" # prototypes.h, version.h, vmu*.xpm
              "${X11_INCLUDE_DIR}")

target_link_libraries(vms PRIVATE "${X11_LIBRARIES}")

# Propagate feature-detection results as compile definitions
foreach(
  _def
  HAVE_UNISTD_H
  HAVE_FCNTL_H
  HAVE_SYS_TIME_H
  HAVE_LIMITS_H
  HAVE_ERRNO_H
  HAVE_UTIME_H
  HAVE_X11_XUTIL_H
  BSD_STYLE_GETTIMEOFDAY
  TIMEZONE_IS_VOID)
  if(${_def})
    target_compile_definitions(vms PRIVATE ${_def})
  endif()
endforeach()

# SDL audio (optional)
if(SDL_FOUND)
  target_compile_definitions(vms PRIVATE USE_SDL_AUDIO HAVE_SDL_SDL_H
                                         HAVE_SDL_SDL_AUDIO_H)
  target_include_directories(vms PRIVATE "${SDL_INCLUDE_DIR}")
  target_link_libraries(vms PRIVATE "${SDL_LIBRARY}")
endif()
