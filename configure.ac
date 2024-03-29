AC_PREREQ([2.64])

m4_define([wayland_major_version],  [1])
m4_define([wayland_minor_version], [17])
m4_define([wayland_micro_version], [90])
m4_define([wayland_version],
          [wayland_major_version.wayland_minor_version.wayland_micro_version])

AC_INIT([wayland],
        [wayland_version],
	[https://gitlab.freedesktop.org/wayland/wayland/issues/],
        [wayland],
        [https://wayland.freedesktop.org/])

AC_SUBST([WAYLAND_VERSION_MAJOR], [wayland_major_version])
AC_SUBST([WAYLAND_VERSION_MINOR], [wayland_minor_version])
AC_SUBST([WAYLAND_VERSION_MICRO], [wayland_micro_version])
AC_SUBST([WAYLAND_VERSION], [wayland_version])

AC_CONFIG_HEADERS([config.h])
AC_CONFIG_MACRO_DIR([m4])

AM_INIT_AUTOMAKE([1.11 foreign no-dist-gzip dist-xz subdir-objects])

AM_SILENT_RULES([yes])

# Check for programs
AC_PROG_CC
AC_PROG_CXX
AC_PROG_GREP
AM_PROG_AS
AC_PROG_NM

# check if we have C++ compiler. This is hacky workaround,
# for a reason why it is this way see
# http://lists.gnu.org/archive/html/bug-autoconf/2010-05/msg00001.html
have_cpp_compiler=yes

if ! which "$CXX" &>/dev/null; then
	have_cpp_compiler=no
fi

AM_CONDITIONAL(ENABLE_CPP_TEST, test "x$have_cpp_compiler" = "xyes")

# Initialize libtool
LT_PREREQ([2.2])
LT_INIT([disable-static])

PKG_PROG_PKG_CONFIG()

AC_ARG_ENABLE([fatal-warnings],
	      AC_HELP_STRING([--enable-fatal-warnings],
			     [Build with -Werror]),
	      [enable_fatal_warnings=$enableval],
	      [enable_fatal_warnings=no])
AS_IF([test x"$enable_fatal_warnings" != "xno"], [
      WERROR_CFLAGS="-Werror"
])

if test "x$GCC" = "xyes"; then
	GCC_CFLAGS="$WERROR_CFLAGS -Wall -Wextra -Wno-unused-parameter -g -Wstrict-prototypes -Wmissing-prototypes -fvisibility=hidden"
fi
AC_SUBST(GCC_CFLAGS)

AC_CHECK_HEADERS([sys/prctl.h])
AC_CHECK_FUNCS([accept4 mkostemp posix_fallocate prctl memfd_create strndup])

# *BSD don't have libdl, but they have its functions in libc
WESTON_SEARCH_LIBS([DL], [dl], [dlsym])

# OpenBSD doesn't have librt, but it has its functions in libc
WESTON_SEARCH_LIBS([RT], [rt], [clock_gettime])

AC_ARG_ENABLE([libraries],
	      [AC_HELP_STRING([--disable-libraries],
			      [Disable compilation of wayland libraries])],
	      [],
	      [enable_libraries=yes])

AC_ARG_WITH([host-scanner],
              [AC_HELP_STRING([--with-host-scanner],
                              [Use installed wayland-scanner from host PATH during build])],
              [],
              [with_host_scanner=no])

AC_ARG_ENABLE([documentation],
	      [AC_HELP_STRING([--disable-documentation],
		              [Disable building the documentation])],
	      [],
	      [enable_documentation=yes])

AC_ARG_ENABLE([dtd-validation],
	      [AC_HELP_STRING([--disable-dtd-validation],
			      [Disable DTD validation of the protocol])],
	      [],
	      [enable_dtd_validation=yes])

AM_CONDITIONAL(USE_HOST_SCANNER, test "x$with_host_scanner" = xyes)

AM_CONDITIONAL(ENABLE_LIBRARIES, test "x$enable_libraries" = xyes)

AC_ARG_WITH(icondir, [  --with-icondir=<dir>    Look for cursor icons here],
		     [  ICONDIR=$withval],
		     [  ICONDIR=${datadir}/icons])
AC_SUBST([ICONDIR])

if test "x$enable_libraries" = "xyes"; then
	PKG_CHECK_MODULES(FFI, [libffi])
	AC_CHECK_DECL(SFD_CLOEXEC,[],
		      [AC_MSG_ERROR("SFD_CLOEXEC is needed to compile wayland libraries")],
		      [[#include <sys/signalfd.h>]])
	AC_CHECK_DECL(TFD_CLOEXEC,[],
		      [AC_MSG_ERROR("TFD_CLOEXEC is needed to compile wayland libraries")],
		      [[#include <sys/timerfd.h>]])
	AC_CHECK_DECL(CLOCK_MONOTONIC,[],
		      [AC_MSG_ERROR("CLOCK_MONOTONIC is needed to compile wayland libraries")],
		      [[#include <time.h>]])
fi

PKG_CHECK_MODULES(EXPAT, [expat])

AM_CONDITIONAL([DTD_VALIDATION], [test "x$enable_dtd_validation" = "xyes"])
if test "x$enable_dtd_validation" = "xyes"; then
	PKG_CHECK_MODULES(LIBXML, [libxml-2.0])
	AC_DEFINE(HAVE_LIBXML, 1, [libxml-2.0 is available])
	AC_CONFIG_LINKS([src/wayland.dtd.embed:protocol/wayland.dtd])
fi

AC_PATH_PROG(XSLTPROC, xsltproc)
AM_CONDITIONAL([HAVE_XSLTPROC], [test "x$XSLTPROC" != "x"])


AM_CONDITIONAL(BUILD_DOCS, [test x$enable_documentation = xyes])
if test "x$enable_documentation" = "xyes"; then
	AC_PATH_PROG(DOXYGEN, doxygen)

	if test "x$DOXYGEN" = "x"; then
		AC_MSG_ERROR([Documentation build requested but doxygen not found. Install doxygen or disable the documentation using --disable-documentation])
	fi

	AC_MSG_CHECKING([for compatible doxygen version])
	doxygen_version=`$DOXYGEN --version`
	AS_VERSION_COMPARE([$doxygen_version], [1.6.0],
	                   [AC_MSG_RESULT([no])
	                    AC_MSG_ERROR([Doxygen $doxygen_version too old. Doxygen 1.6+ required for documentation build. Install required doxygen version or disable the documentation using --disable-documentation])],
	                   [AC_MSG_RESULT([yes])],
	                   [AC_MSG_RESULT([yes])])

	AC_PATH_PROG(XMLTO, xmlto)

	if test "x$XMLTO" = "x"; then
		AC_MSG_ERROR([Documentation build requested but xmlto not found. Install xmlto or disable the documentation using --disable-documentation])
	fi

	AC_PATH_PROG(DOT, dot)
	if test "x$DOT" = "x"; then
		AC_MSG_ERROR([Documentation build requested but graphviz's dot not found. Install graphviz or disable the documentation using --disable-documentation])
	fi
	AC_MSG_CHECKING([for compatible dot version])
	dot_version=`$DOT -V 2>&1|$GREP -o ['[0-9]*\.[0-9]*\.[0-9]*']`
	AS_VERSION_COMPARE([$dot_version], [2.26.0],
	                   [AC_MSG_RESULT([no])
	                    AC_MSG_ERROR([Graphviz dot $dot_version too old. Graphviz 2.26+ required for documentation build. Install required graphviz version or disable the documentation using --disable-documentation])],
	                   [AC_MSG_RESULT([yes])],
	                   [AC_MSG_RESULT([yes])])

	AC_MSG_CHECKING([for docbook stylesheets])
	DOCS_STYLESHEET=http://docbook.sourceforge.net/release/xsl/current/manpages/docbook.xsl
	AC_PATH_PROGS_FEATURE_CHECK([XSLTPROC_TMP], [xsltproc],
				    AS_IF([`"$ac_path_XSLTPROC_TMP" --nonet "$DOCS_STYLESHEET" > /dev/null 2>&1`],
					  [HAVE_DOCS_STYLESHEET=yes]))
	if test "x$HAVE_DOCS_STYLESHEET" != "xyes"; then
		AC_MSG_RESULT([no])
		AC_MSG_ERROR([Documentation build requested but docbook-xsl stylesheets are not found. Install the docbook-xsl package or disable the documentation using --disable-documentation])
	fi

	AC_MSG_RESULT([yes])
	AC_SUBST(DOCS_STYLESHEET)

	AC_CONFIG_FILES([
	doc/doxygen/wayland.doxygen
	])

fi
AM_CONDITIONAL([HAVE_XMLTO], [test "x$XMLTO" != "x"])

AC_CONFIG_FILES([Makefile
		 cursor/wayland-cursor.pc
		 cursor/wayland-cursor-uninstalled.pc
		 doc/Makefile
		 doc/publican/Makefile
		 doc/doxygen/Makefile
		 doc/man/Makefile
		 egl/wayland-egl.pc
		 egl/wayland-egl-backend.pc
		 src/wayland-server-uninstalled.pc
		 src/wayland-client-uninstalled.pc
		 src/wayland-scanner-uninstalled.pc
		 src/wayland-server.pc
		 src/wayland-client.pc
		 src/wayland-scanner.pc
		 src/wayland-version.h])
AC_OUTPUT
