dnl Process this file with autoconf to produce a configure script.
AC_INIT([libpwquality], [1.4.5])
AC_CONFIG_HEADERS([config.h])
AM_INIT_AUTOMAKE([dist-bzip2 no-dist-gzip -Wall])
AC_PREREQ(2.61)
AC_CONFIG_MACRO_DIR([m4])
AC_CANONICAL_HOST

dnl We don't build with srcdir != builddir at present; the Python
dnl bindings fail
echo \#buildapi-variable-no-builddir > /dev/null

dnl LT Version numbers, remember to change them *before* a release.
dnl   (Interfaces removed:    CURRENT++, AGE=0, REVISION=0)
dnl   (Interfaces added:      CURRENT++, AGE++, REVISION=0)
dnl   (No interfaces changed:                   REVISION++)
PWQUALITY_LT_CURRENT=1
PWQUALITY_LT_AGE=0
PWQUALITY_LT_REVISION=2

AC_SUBST(PACKAGE)
AC_SUBST(VERSION)
AC_SUBST(PWQUALITY_LT_CURRENT)
AC_SUBST(PWQUALITY_LT_AGE)
AC_SUBST(PWQUALITY_LT_REVISION)

dnl Checks for programs.
AC_USE_SYSTEM_EXTENSIONS
AM_PROG_AR
AC_PROG_CC
AC_PROG_INSTALL
AC_PROG_LN_S
AC_PROG_LIBTOOL

dnl and some hacks to use /etc
test "${prefix}" = "NONE" && prefix="/usr"
if test ${prefix} = '/usr'
then
dnl If we use /usr as prefix, use /etc for config files
        if test ${sysconfdir} = '${prefix}/etc'
        then
                sysconfdir="/etc"
        fi
dnl If we use /usr as prefix, use /usr/share/man for manual pages
        if test ${mandir} = '${prefix}/man'
        then
                mandir='${prefix}/share/man'
        fi
fi

dnl Localization support
AM_GNU_GETTEXT_VERSION([0.18.2])
AM_GNU_GETTEXT([external])
AH_BOTTOM([#ifdef ENABLE_NLS
#include <libintl.h>
#define _(msgid) dgettext(PACKAGE, msgid)
#define N_(msgid) msgid
#else   
#define _(msgid) (msgid)
#define N_(msgid) msgid
#endif /* ENABLE_NLS */])

dnl Library version scripts

dnl For some systems we know that we have ld_version scripts.
dnl Use it then as default.
have_ld_version_script=no
case "${host}" in
    *-*-linux*)
        have_ld_version_script=yes
        ;;
    *-*-gnu*)
        have_ld_version_script=yes
        ;;
esac

AC_ARG_ENABLE([ld-version-script],
              AC_HELP_STRING([--enable-ld-version-script],
                             [enable/disable use of linker version script.
                              (default is system dependent)]),
              [have_ld_version_script=$enableval],
              [ : ] )
AM_CONDITIONAL(HAVE_LD_VERSION_SCRIPT, test "$have_ld_version_script" = "yes")

dnl PAM support
PAM_LIBS=
AC_ARG_ENABLE([pam],
  AS_HELP_STRING([--enable-pam],[build PAM module @<:@default=check@:>@]),
  [],
  [enable_pam=check])
if test "$enable_pam" != "no"; then
  if test "$enable_pam" = "check"; then
      AC_CHECK_LIB([pam], [pam_start], [enable_pam=yes], [enable_pam=no])
  else
      fail=0
      AC_CHECK_LIB([pam], [pam_start], [], [fail=1])
      test $fail = 1 &&
        AC_MSG_ERROR([You must install the PAM development package in order to compile libpwquality])
  fi
  AC_CHECK_LIB(
    [pam],
    [pam_modutil_check_user_in_passwd],
    [AC_DEFINE([HAVE_PAM_CHECK_USER_IN_PASSWD], [], [have pam_modutil_check_user_in_passwd])],
    []
  )  
fi

if test "$enable_pam" = "yes"; then
  PAM_LIBS="${PAM_LIBS} -lpam"
fi
AC_SUBST(PAM_LIBS)
AM_CONDITIONAL(HAVE_PAM, test "$enable_pam" = "yes")

dnl options and defaults
AC_ARG_WITH(securedir,
        AS_HELP_STRING([--with-securedir=DIR],[path to location of PAMs @<:@default=$libdir/security@:>@]),
        SECUREDIR=$withval, SECUREDIR=$libdir/security)
AC_SUBST(SECUREDIR)

AC_ARG_WITH(sconfigdir,
	AS_HELP_STRING([--with-sconfigdir=DIR],[path to PAM conf files @<:@default=$sysconfdir/security@:>@]),
	SCONFIGDIR=$withval, SCONFIGDIR=$sysconfdir/security)
AC_SUBST(SCONFIGDIR)

AC_ARG_ENABLE([python-bindings],
              AC_HELP_STRING([--enable-python-bindings],
                             [enable Python bindings (default: yes)]),
              [enable_python_bindings=$enableval],
              enable_python_bindings=yes)
AM_CONDITIONAL(BUILD_PYTHON_BINDINGS, test x"$enable_python_bindings" = xyes)

AC_ARG_WITH(python-binary,
            [  --with-python-binary=python3    python binary name with which modules should be built],
            PYTHONBINARY=$withval, [PYTHONBINARY=python])
AC_SUBST(PYTHONBINARY)

AC_ARG_WITH(python-rev,
            [  --with-python-rev=2.7    version of python for which modules should be built],
            PYTHONREV=$withval,[
            AC_MSG_CHECKING([python version])
            [PYTHONREV=`$PYTHONBINARY -c "import sys;sys.stdout.write('%s.%s' % (sys.version_info[0], sys.version_info[1]))"`]
            AC_MSG_RESULT($PYTHONREV)
            ])
AC_SUBST(PYTHONREV)

AC_ARG_WITH(pythonsitedir,
            [  --with-pythonsitedir=DIR    directory of the python site packages @<:@default=$libdir/python$PYTHONREV/site-packages@:>@],
            PYTHONSITEDIR=$withval,[
            AC_MSG_CHECKING([python site directory])
            [PYTHONSITEDIR=$libdir/python$PYTHONREV/site-packages]
            AC_MSG_RESULT($PYTHONSITEDIR)
            ])
AC_SUBST(PYTHONSITEDIR)

AC_ARG_WITH([randomdev], AS_HELP_STRING([--with-randomdev=(<path>|yes)],[use specified random device instead of /dev/urandom]), opt_randomdev=$withval)
if test "$opt_randomdev" = yes -o -z "$opt_randomdev"; then
       opt_randomdev="/dev/urandom"
fi
AC_DEFINE_UNQUOTED(CONF_PATH_RANDOMDEV, "$opt_randomdev", [Random device path.])

dnl Check for cracklib
AC_ARG_ENABLE([cracklib-check],
        AS_HELP_STRING([--disable-cracklib-check], [disable cracklib dictionary check]),
        [enable_cracklib_check=$enableval])

AS_IF([test "x$enable_cracklib_check" != "xno"], [
AC_CHECK_HEADERS([crack.h],[
    dnl If cracklib is shared -lz is not needed
    AC_CHECK_LIB([crack], [FascistCheck], LIBCRACK="-lcrack" LIBS="$LIBS", LIBCRACK="")
if test "x$ac_cv_lib_crack_FascistCheck" = "xno" ; then
    unset ac_cv_lib_crack_FascistCheck
    AC_CHECK_LIB([crack], [FascistCheck], LIBCRACK="-lz -lcrack" LIBS="$LIBS -lz", LIBCRACK="", [-lz])
fi
    ])
if test "x$LIBCRACK" = "x" ; then
    AC_MSG_ERROR([No or unusable cracklib library])
fi
AC_SUBST([LIBCRACK])
])

dnl Checks for typedefs, structures, and compiler characteristics.
AC_C_BIGENDIAN

dnl Files to be created from when we run configure
AC_CONFIG_FILES([Makefile libpwquality.spec src/Makefile src/pwquality.pc po/Makefile.in python/Makefile
                 doc/Makefile doc/man/Makefile python/setup.py])
AC_OUTPUT
