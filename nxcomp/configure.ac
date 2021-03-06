# ***************************************************************************
# *** configure.ac for nxcomp                                         ***
# ***************************************************************************

m4_define([nxcomp_version], m4_esyscmd([tr -d '\n' < VERSION]))

# Initialize Autoconf
AC_PREREQ(2.60)

AC_INIT([libXcomp], [nxcomp_version], [https://github.com/ArcticaProject/nx-libs/issues])
AC_CONFIG_AUX_DIR([build-aux])
AC_PROG_CXX
AC_CONFIG_SRCDIR([Makefile.am])
AC_CONFIG_HEADERS([config.h])
AC_CONFIG_MACRO_DIR([m4])

AM_INIT_AUTOMAKE([foreign no-dist-gzip dist-bzip2])

# Initialize libtool
AC_PROG_LIBTOOL

# enable this to add the variable to the Makefiles
#AC_ARG_VAR(NX_VERSION_CUSTOM, [custom version string])

if test "x$NX_VERSION_CUSTOM" != x; then
    AC_DEFINE_UNQUOTED([NX_VERSION_CUSTOM], ["${NX_VERSION_CUSTOM}"], [user provided custom version string])
fi

COMP_VERSION=nxcomp_version
AC_SUBST([COMP_VERSION])

LT_COMP_VERSION=["$(printf '%s\n' "${COMP_VERSION}" | awk -F '.' '/^[0-9]+\.[0-9]+\.[0-9]+.*$/ { print $1 ":" $2 ":" $3; }')"]
AC_SUBST([LT_COMP_VERSION])

# Silence warning: ar: 'u' modifier ignored since 'D' is the default
AC_SUBST(AR_FLAGS, [cr])

# Upstream's pkg.m4 (since 0.27) offers this now, but define our own
# compatible version in case the local version of pkgconfig isn't new enough.
# https://bugs.freedesktop.org/show_bug.cgi?id=48743
m4_ifdef([PKG_INSTALLDIR], [PKG_INSTALLDIR],
         [AC_ARG_WITH([pkgconfigdir],
                      [AS_HELP_STRING([--with-pkgconfigdir],
                      [install directory for nxcomp.pc pkg-config file])],
                       [],[with_pkgconfigdir='$(libdir)/pkgconfig'])
          AC_SUBST([pkgconfigdir], [${with_pkgconfigdir}])])

PKG_CHECK_MODULES([JPEG], [libjpeg], [], [LIBJPEG_FALLBACK_CHECK])
PKG_CHECK_MODULES([PNG], [libpng])
PKG_CHECK_MODULES([Z], [zlib], [], [ZLIB_FALLBACK_CHECK])

AC_LANG([C++])
NX_COMPILER_BRAND
NX_DEFAULT_OPTIONS

NX_BUILD_ON_CYGWIN32
NX_BUILD_ON_AMD64
NX_BUILD_ON_DARWIN
NX_BUILD_ON_SUN
NX_BUILD_ON_FreeBSD

# Build PIC libraries.

if test "$CYGWIN32" != yes -a "$DARWIN" != yes; then
  CXXFLAGS="$CXXFLAGS -fPIC"
  CFLAGS="$CFLAGS -fPIC"
fi

# On FreeBSD search libraries and includes under /usr/local.

if test "$FreeBSD" = yes; then
  LIBS="$LIBS -L/usr/local/lib"
  CPPFLAGS="$CPPFLAGS -I/usr/local/include"
fi

AX_PTHREAD([], AC_MSG_ERROR([no POSIX threads support detected]))

# If in_addr_t is not defined use unsigned int.
AC_CHECK_TYPES([in_addr_t], [], [], [[#include <netinet/in.h>]])

AC_ARG_ENABLE([cxx11],
              [AS_HELP_STRING([--enable-cxx11],
                              [enable optional features requiring C++11 support (disabled by default)])],
              [AS_IF([test x$enableval = xyes],
                    [AX_CXX_COMPILE_STDCXX_11([], [mandatory])])])

# Check if std::put_time is available.
AC_MSG_CHECKING([if std::put_time is available])
AC_COMPILE_IFELSE([AC_LANG_PROGRAM(
[[
#include <iomanip>
#include <ctime>
]],
[[
std::time_t t = std::time(NULL);
std::tm tm = *std::localtime(&t);
(void) std::put_time(&tm, "%c");
]])],
                  [AC_MSG_RESULT([yes])
                   AC_DEFINE(HAVE_STD_PUT_TIME, [1],
                             [Use std::put_time to format times, must be made available by the compiler if turned on.])],
                  [AC_MSG_RESULT([no])])

# Check if ::ctime_s is available.
AC_MSG_CHECKING([if ::ctime_s is available])
AC_COMPILE_IFELSE([AC_LANG_PROGRAM(
[[
#define __STDC_WANT_LIB_EXT1__ 1
#include <ctime>
]],
[[
time_t res = time(NULL);
char str[26] = { };
::ctime_s(str, sizeof(str), &res);
]])],
                  [AC_MSG_RESULT([yes])
                   AC_DEFINE(HAVE_CTIME_S, [1],
                             [Use ::ctime_s to format times, must be made available by the compiler if turned on.])],
                  [AC_MSG_RESULT([no])])

AC_ARG_ENABLE([debug],
              [AS_HELP_STRING([--enable-debug],
                              [enable to get info session log output (disabled by default)])],
              [AS_IF([test x$enableval = xyes],
                     [AC_DEFINE(INFO, 1, [Define this to get info session log output.])
                      info_messages=yes])])

AC_ARG_ENABLE([valgrind],
              [AS_HELP_STRING([--enable-valgrind],
                              [enable for extra valgrind hacks (disabled by default)])],
              [AS_IF([test x$enableval = xyes],
                     [AC_DEFINE(VALGRIND, 1, [Define this for extra valgrind hacks.])
                      valgrind_hacks=yes])])

AC_CONFIG_FILES([
Makefile
src/Makefile
test/Makefile
nxcomp.pc
])

AC_OUTPUT
