AC_INIT([table-sqlite], [1.0], [bugs@opensmtpd.org])
AC_CONFIG_AUX_DIR(etc)
AM_INIT_AUTOMAKE([-Wall foreign subdir-objects])
AC_CONFIG_LIBOBJ_DIR(openbsd-compat)
AC_PROG_CC
AC_USE_SYSTEM_EXTENSIONS

AC_REPLACE_FUNCS([ \
	asprintf \
	getprogname \
	err \
	strlcat \
	strlcpy \
	strsep \
	strtonum \
])

AC_SEARCH_LIBS([sqlite3_open_v2], [sqlite3], [], [
	AC_MSG_ERROR([requires sqlite3])
])

CFLAGS="$CFLAGS -I$srcdir/openbsd-compat"

AC_CHECK_HEADER([sys/tree.h], [], [
	CFLAGS="$CFLAGS -I$srcdir/openbsd-compat/tree"
])

AC_CONFIG_HEADERS([config.h])
AC_CONFIG_FILES([
	Makefile
])
AC_OUTPUT
