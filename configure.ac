#                                               -*- Autoconf -*-
# Process this file with autoconf to produce a configure script.

AC_PREREQ(2.64)
AC_INIT([libplist], [1.13.0], [https://github.com/libimobiledevice/libplist/issues],, [http://libimobiledevice.org])
AM_INIT_AUTOMAKE([dist-bzip2 no-dist-gzip check-news])
m4_ifdef([AM_SILENT_RULES], [AM_SILENT_RULES])
AC_CONFIG_SRCDIR([src/])
AC_CONFIG_HEADERS([config.h])
AC_CONFIG_MACRO_DIR([m4])

dnl libtool versioning
# +1 : 0 : +1  == adds new functions to the interface
# +1 : 0 : 0   == changes or removes functions (changes include both
#                 changes to the signature and the semantic)
#  ? :+1 : ?   == just internal changes
# CURRENT : REVISION : AGE
LIBPLIST_SO_VERSION=3:0:0

AC_SUBST(LIBPLIST_SO_VERSION)

# Checks for programs.
AC_PROG_CC
AC_PROG_CXX

# AC_PROG_CXX will return "g++" even if no c++ compiler is installed
AC_LANG_PUSH([C++])
AC_MSG_CHECKING([whether $CXX is available and compiles a program])
AC_COMPILE_IFELSE([AC_LANG_PROGRAM([],[])],
                  [AC_MSG_RESULT([yes])],
                  [AC_MSG_RESULT([no])
                   AC_MSG_ERROR([C++ compiler not available or unable to compile])])
AC_LANG_POP

AM_PROG_CC_C_O
AC_PROG_LIBTOOL

# Checks for header files.
AC_HEADER_STDC
AC_CHECK_HEADERS([stdint.h stdlib.h string.h])

# Checks for typedefs, structures, and compiler characteristics.
AC_C_CONST
AC_TYPE_SIZE_T
AC_TYPE_SSIZE_T
AC_TYPE_UINT16_T
AC_TYPE_UINT32_T
AC_TYPE_UINT8_T

# Checks for library functions.
AC_CHECK_FUNCS([asprintf strcasecmp strdup strerror strndup stpcpy vasprintf gmtime_r localtime_r timegm])

# Checking endianness
AC_C_BIGENDIAN([AC_DEFINE([__BIG_ENDIAN__], [1], [big endian])],
               [AC_DEFINE([__LITTLE_ENDIAN__], [1], [little endian])])


# Check for operating system
AC_MSG_CHECKING([whether to enable WIN32 build settings])
case ${host_os} in
  *mingw32*|*cygwin*)
    win32=true
    AC_MSG_RESULT([yes])
    AC_SUBST(WINDRES)
    ;;
  *)
    win32=false
    AC_MSG_RESULT([no])
    ;;
esac
AM_CONDITIONAL(WIN32, test x$win32 = xtrue)

# if we are not on win32, check for pthread
if test x"$win32" != x"true"; then
  AX_PTHREAD([], [AC_MSG_ERROR([pthread is required to build libplist])])
  AC_CHECK_LIB(pthread, [pthread_once], [], [AC_MSG_ERROR([pthread with pthread_once required to build libplist])])
fi

# Check if struct tm has a tm_gmtoff member
AC_CACHE_CHECK(for tm_gmtoff in struct tm, ac_cv_struct_tm_gmtoff,
	AC_TRY_COMPILE([
		#include <time.h>
	], [
		struct tm tm;
		tm.tm_gmtoff = 1;
	], ac_cv_struct_tm_gmtoff=yes, ac_cv_struct_tm_gmtoff=no))

if (test "$ac_cv_struct_tm_gmtoff" = "yes"); then
	AC_DEFINE(HAVE_TM_TM_GMTOFF, 1, [Define if struct tm has a tm_gmtoff member])
fi

# Check if struct tm has a tm_zone member
AC_CACHE_CHECK(for tm_zone in struct tm, ac_cv_struct_tm_zone,
	AC_TRY_COMPILE([
		#include <time.h>
	], [
		struct tm tm;
		tm.tm_zone = 1;
	], ac_cv_struct_tm_zone=yes, ac_cv_struct_tm_zone=no))

if (test "$ac_cv_struct_tm_zone" = "yes"); then
	AC_DEFINE(HAVE_TM_TM_ZONE, 1, [Define if struct tm has a tm_zone member])
fi

# Cython Python Bindings
AC_ARG_WITH([cython],
            [AS_HELP_STRING([--without-cython],
            [build Python bindings using Cython (default is yes)])],
            [build_cython=false],
            [build_cython=true])
if test "$build_cython" = "true"; then
            AM_PATH_PYTHON(2.3)
            AC_PROG_CYTHON(0.17.0)
            CYTHON_PYTHON
else
            CYTHON=false
fi
if [test "x$CYTHON" != "xfalse"]; then
            AC_MSG_CHECKING([for libplist Cython bindings])
            CYTHON_PLIST_INCLUDE_DIR=$($PKG_CONFIG --variable=includedir libplist)/plist/cython
            if [test ! -d "$CYTHON_PLIST_INCLUDE_DIR"]; then
                CYTHON_PLIST_INCLUDE_DIR=.
                CYTHON_SUB=cython
                cython_python_bindings=yes
                AC_MSG_RESULT([Using built-in libplist Cython bindings (assuming this is a first build)])
            else
                AC_SUBST([CYTHON_PLIST_INCLUDE_DIR])
                AC_MSG_RESULT([$CYTHON_PLIST_INCLUDE_DIR])
                CYTHON_SUB=cython
                cython_python_bindings=yes
            fi
else
            CYTHON_SUB=
            cython_python_bindings=no
fi
AM_CONDITIONAL([HAVE_CYTHON],[test "x$CYTHON_SUB" = "xcython"])
AC_SUBST([CYTHON_SUB])

AS_COMPILER_FLAGS(GLOBAL_CFLAGS, "-Wall -Wextra -Wredundant-decls -Wshadow -Wpointer-arith  -Wwrite-strings -Wswitch-default -Wno-unused-parameter -fvisibility=hidden $PTHREAD_CFLAGS")
GLOBAL_LDFLAGS="$PTHREAD_LIBS"

AC_ARG_ENABLE(debug,
AS_HELP_STRING([--enable-debug],
               [enable debugging, default: no]),
[case "${enableval}" in
             yes) debug=yes ;;
             no)  debug=no ;;
             *)   AC_MSG_ERROR([bad value ${enableval} for --enable-debug]) ;;
esac],
[debug=no])

if (test "x$debug" = "xyes"); then
       AC_DEFINE(DEBUG, 1, [Define if debug code should be enabled.])
       GLOBAL_CFLAGS+=" -g"
fi

AC_SUBST(GLOBAL_CFLAGS)
AC_SUBST(GLOBAL_LDFLAGS)

case "$GLOBAL_CFLAGS" in
    *-fvisibility=hidden*)
        AC_DEFINE([HAVE_FVISIBILITY], [1], [Define if compiled with -fvisibility=hidden])
esac

m4_ifdef([AM_SILENT_RULES],[AM_SILENT_RULES([yes])])

AC_OUTPUT([
Makefile
libcnary/Makefile
src/Makefile
src/libplist.pc
src/libplist++.pc
include/Makefile
tools/Makefile
cython/Makefile
test/Makefile
doxygen.cfg
])

echo "
Configuration for $PACKAGE $VERSION:
-------------------------------------------

  Install prefix ..........: $prefix
  Debug code ..............: $debug
  Python bindings .........: $cython_python_bindings

  Now type 'make' to build $PACKAGE $VERSION,
  and then 'make install' for installation.
"
