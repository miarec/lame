include(CheckTypeSize)
check_type_size("short" SIZEOF_SHORT LANGUAGE C)
check_type_size("int" SIZEOF_INT LANGUAGE C)
check_type_size("long" SIZEOF_LONG LANGUAGE C)
check_type_size("long long" SIZEOF_LONG_LONG LANGUAGE C)
check_type_size("float" SIZEOF_FLOAT LANGUAGE C)
check_type_size("double" SIZEOF_DOUBLE LANGUAGE C)
check_type_size("long double" SIZEOF_LONG_DOUBLE LANGUAGE C)

check_type_size("unsigned int" SIZEOF_UNSIGNED_INT LANGUAGE C)
check_type_size("unsigned long" SIZEOF_UNSIGNED_LONG LANGUAGE C)
check_type_size("unsigned long long" SIZEOF_UNSIGNED_LONG_LONG LANGUAGE C)
check_type_size("unsigned short" SIZEOF_UNSIGNED_SHORT LANGUAGE C)

check_type_size("int8_t" SIZEOF_INT8_T LANGUAGE C)
check_type_size("int16_t" SIZEOF_INT16_T LANGUAGE C)
check_type_size("int32_t" SIZEOF_INT32_T LANGUAGE C)
check_type_size("int64_t" SIZEOF_INT64_T LANGUAGE C)

check_type_size("uint8_t" SIZEOF_UINT8_T LANGUAGE C)
check_type_size("uint16_t" SIZEOF_UINT16_T LANGUAGE C)
check_type_size("uint32_t" SIZEOF_UINT32_T LANGUAGE C)
check_type_size("uint64_t" SIZEOF_UINT64_T LANGUAGE C)


set( HAVE_INT8_T SIZEOF_INT8_T )
set( HAVE_INT16_T SIZEOF_INT16_T )
set( HAVE_INT32_T SIZEOF_INT32_T )
set( HAVE_INT64_T SIZEOF_INT64_T )

set( HAVE_UINT8_T SIZEOF_UINT8_T )
set( HAVE_UINT16_T SIZEOF_UINT16_T )
set( HAVE_UINT32_T SIZEOF_UINT32_T )
set( HAVE_UINT64_T SIZEOF_UINT64_T )

if (${SIZEOF_LONG_DOUBLE} GREATER ${SIZEOF_DOUBLE})
  set(HAVE_LONG_DOUBLE 1)
  set(HAVE_LONG_DOUBLE_WIDER 1)
endif ()


if( SIZEOF_INT EQUAL 4 )
   set( A_INT32_T "int" )
   set( A_UINT32_T "unsigned int" )
elseif( SIZEOF_LONG EQUAL 4 )
   set( A_INT32_T "long" )
   set( A_UINT32_T "unsigned long" )
endif()

if( SIZEOF_INT EQUAL 8 )
   set( A_INT64_T "int" )
   set( A_UINT64_T "unsigned int" )
elseif( SIZEOF_LONG EQUAL 8 )
   set( A_INT64_T "long" )
   set( A_UINT64_T "unsigned long" )
elseif( SIZEOF_LONG_LONG EQUAL 8 )
   set( A_INT64_T "long long" )
   set( A_UINT64_T "unsigned long long" )
endif()


# Headers
include(CheckIncludeFiles)
check_include_files(alloca.h HAVE_ALLOCA_H)
check_include_file(dlfcn.h HAVE_DLFCN_H)
check_include_file(errno.h HAVE_ERRNO_H)
check_include_file(fcntl.h HAVE_FCNTL_H)
check_include_file(inttypes.h HAVE_INTTYPES_H)
check_include_file(limits.h HAVE_LIMITS_H)
check_include_file(linux/soundcard.h HAVE_LINUX_SOUNDCARD_H)
check_include_file(memory.h HAVE_MEMORY_H)
check_include_file(ncurses/termcap.h HAVE_NCURSES_TERMCAP_H)
check_include_file(stdint.h HAVE_STDINT_H)
check_include_file(stdlib.h HAVE_STDLIB_H)
check_include_file(strings.h HAVE_STRINGS_H)
check_include_file(string.h HAVE_STRING_H)
check_include_file(sys/soundcard.h HAVE_SYS_SOUNDCARD_H)
check_include_file(sys/stat.h HAVE_SYS_STAT_H)
check_include_file(sys/time.h HAVE_SYS_TIME_H)
check_include_file(termcap.h HAVE_TERMCAP_H)
check_include_file(unistd.h HAVE_UNISTD_H)
check_include_files("stdlib.h;stdarg.h;string.h;float.h" STDC_HEADERS)
check_include_files("sys/time.h;time.h" TIME_WITH_SYS_TIME)


# Functions
include(CheckFunctionExists)
check_function_exists(gettimeofday HAVE_GETTIMEOFDAY)
check_function_exists(iconv HAVE_ICONV)
check_function_exists(socket HAVE_SOCKET)
check_function_exists(strtol HAVE_STRTOL)

include(CheckSymbolExists)
check_symbol_exists(alloca "aloca.h" HAVE_ALLOCA)



include(CheckCSourceCompiles)
# Checks for actually working SSE intrinsics
  check_c_source_compiles("
#include <xmmintrin.h>
int
main(void)
{
  _mm_sfence();
  return 0;
}
" HAVE_XMMINTRIN_H)

if (${HAVE_XMMINTRIN_H})
    message(STATUS "Building with SSE intrinsics instructions")
endif()

# processor specific options
if (CMAKE_SYSTEM_PROCESSOR MATCHES "i386|i686|x86_64|AMD64|X86")
  if (${HAVE_XMMINTRIN_H})
    set( WITH_XMM 1 CACHE INTERNAL "with XMM")
    set( WITH_VECTOR 1 CACHE INTERNAL "with the internal vector library")
  endif()

  set( TAKEHIRO_IEEE754_HACK 1 "IEEE754 compabible machine" )
  set( USE_FAST_LOG 1 "faster log implementation with less but enough precission" )

elseif(CMAKE_SYSTEM_PROCESSOR MATCHES "powerpc")

  set( TAKEHIRO_IEEE754_HACK 1 "IEEE754 compabible machine" )
  set( USE_FAST_LOG 1 "faster log implementation with less but enough precission" )

endif()

set( TAKEHIRO_IEEE754_HACK 1 )
set( USE_FAST_LOG 1 )
if( ${CMakeDetermineASM_NASMCompiler} )
   set( HAVE_NASM 1 )
endif()


configure_file(
    ${CMAKE_CURRENT_SOURCE_DIR}/config.h.in 
    ${CMAKE_CURRENT_BINARY_DIR}/private/config.h 
    @ONLY
)
