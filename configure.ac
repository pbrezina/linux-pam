dnl Process this file with autoconf to produce a configure script.
AC_INIT([Linux-PAM], [1.3.1], , [Linux-PAM])
AC_CONFIG_SRCDIR([conf/pam_conv1/pam_conv_y.y])
AC_CONFIG_AUX_DIR([build-aux])
AM_INIT_AUTOMAKE([-Wall -Wno-portability])
AC_PREREQ([2.61])
AC_CONFIG_HEADERS([config.h])
AC_CONFIG_MACRO_DIR([m4])
AC_CANONICAL_HOST

AC_SUBST(PACKAGE)
AC_SUBST(VERSION)

dnl
dnl By default, everything under PAM is installed below /usr.
dnl
AC_PREFIX_DEFAULT(/usr)

dnl and some hacks to use /etc and /lib
test "${prefix}" = "NONE" && prefix="/usr"
if test ${prefix} = '/usr'
then
dnl If we use /usr as prefix, use /etc for config files
        if test ${sysconfdir} = '${prefix}/etc'
        then
                sysconfdir="/etc"
        fi
	if test ${libdir} = '${exec_prefix}/lib'
	then
		case "$host_cpu" in
		    x86_64|ppc64|s390x|sparc64)
			libdir="/lib64" ;;
		    *)
			libdir="/lib" ;;
		esac
	fi
	if test ${sbindir} = '${exec_prefix}/sbin'
        then
                sbindir="/sbin"
        fi
dnl If we use /usr as prefix, use /usr/share/man for manual pages
        if test ${mandir} = '${prefix}/man'
        then
                mandir='${prefix}/share/man'
        fi
dnl Add security to include directory
	if test ${includedir} = '${prefix}/include'
	then
		includedir="${prefix}/include/security"
	fi

dnl Add /var directory
        if test ${localstatedir} = '${prefix}/var'
        then
                localstatedir="/var"
        fi

fi

dnl This should be called before any macros that run the C compiler.
AC_USE_SYSTEM_EXTENSIONS

LT_INIT([disable-static])
AC_ENABLE_STATIC([no])
AC_ENABLE_SHARED([yes])

dnl Checks for programs.
AC_PROG_CC
AC_PROG_YACC
AM_PROG_LEX
AC_PROG_INSTALL
AC_PROG_LN_S
AC_PROG_MAKE_SET
AM_PROG_CC_C_O
PAM_LD_AS_NEEDED
PAM_LD_NO_UNDEFINED
PAM_LD_O1

dnl Largefile support
AC_SYS_LARGEFILE

dnl icc claims to be GCC compatible, but use other flags for warnings
if eval 'test "x$GCC" = "xyes" -a "$CC" != "icc"'; then
  for flag in \
      -W \
      -Wall \
      -Wbad-function-cast \
      -Wcast-align \
      -Wcast-qual \
      -Wmissing-declarations \
      -Wmissing-prototypes \
      -Wpointer-arith \
      -Wreturn-type \
      -Wstrict-prototypes \
      -Wwrite-strings \
      -Winline \
      -Wshadow
  do
    JAPHAR_GREP_CFLAGS($flag, [ CFLAGS="$CFLAGS $flag" ])
  done
fi
dnl icc has special warning flags
if eval 'test "x$CC" = "xicc"'; then
  for flag in \
      -Wall \
      -Wmissing-prototypes \
      -Wpointer-arith \
      -Wreturn-type \
      -Wstrict-prototypes \
      -Wwrite-strings \
      -Wshadow \
      -Wp64 \
      -Wdeprecated \
      -Wuninitialized \
      -Wmain
  do
    JAPHAR_GREP_CFLAGS($flag, [ CFLAGS="$CFLAGS $flag" ])
  done
fi

if test "x${CC_FOR_BUILD+set}" != "xset" ; then
  if test "x$cross_compiling" = "xyes" ; then
    AC_CHECK_PROGS(CC_FOR_BUILD, gcc cc)
  else
    CC_FOR_BUILD=${CC}
  fi
fi
AC_MSG_CHECKING([for CC_FOR_BUILD])
AC_MSG_RESULT([$CC_FOR_BUILD])
AC_SUBST(CC_FOR_BUILD)

if test "x${BUILD_CPPFLAGS+set}" != "xset" ; then
  if test "x$cross_compiling" = "xyes" ; then
    BUILD_CPPFLAGS=
  else
    BUILD_CPPFLAGS=${CPPFLAGS}
  fi
fi
AC_SUBST(BUILD_CPPFLAGS)

if test "x${BUILD_CFLAGS+set}" != "xset" ; then
  if test "x$cross_compiling" = "xyes" ; then
    BUILD_CFLAGS=
  else
    BUILD_CFLAGS=${CFLAGS}
  fi
fi
AC_SUBST(BUILD_CFLAGS)

if test "x${BUILD_LDFLAGS+set}" != "xset" ; then
  if test "x$cross_compiling" = "xyes" ; then
    BUILD_LDFLAGS=
  else
    BUILD_LDFLAGS=${LDFLAGS}
  fi
fi
AC_SUBST(BUILD_LDFLAGS)

AC_C___ATTRIBUTE__

dnl
dnl Check if --version-script is supported by ld
dnl
AC_CACHE_CHECK(for .symver assembler directive, libc_cv_asm_symver_directive,
[cat > conftest.s <<EOF
${libc_cv_dot_text}
_sym:
.symver _sym,sym@VERS
EOF
if ${CC-cc} -c $ASFLAGS conftest.s 1>&AS_MESSAGE_LOG_FD 2>&AS_MESSAGE_LOG_FD; then
  libc_cv_asm_symver_directive=yes
else
  libc_cv_asm_symver_directive=no
fi
rm -f conftest*])
AC_CACHE_CHECK(for ld --version-script, libc_cv_ld_version_script_option, [dnl
if test $libc_cv_asm_symver_directive = yes; then
  cat > conftest.s <<EOF
${libc_cv_dot_text}
_sym:
.symver _sym,sym@VERS
EOF
  cat > conftest.map <<EOF
VERS_1 {
        global: sym;
};

VERS_2 {
        global: sym;
} VERS_1;
EOF
  if ${CC-cc} -c $ASFLAGS conftest.s 1>&AS_MESSAGE_LOG_FD 2>&AS_MESSAGE_LOG_FD;
then
    if AC_TRY_COMMAND([${CC-cc} $CFLAGS $LDFLAGS -shared
                                -o conftest.so conftest.o
                                -nostartfiles -nostdlib
                                -Wl,--version-script,conftest.map
                       1>&AS_MESSAGE_LOG_FD]);
    then
      libc_cv_ld_version_script_option=yes
    else
      libc_cv_ld_version_script_option=no
    fi
  else
    libc_cv_ld_version_script_option=no
  fi
else
  libc_cv_ld_version_script_option=no
fi
rm -f conftest*])
AM_CONDITIONAL([HAVE_VERSIONING],
	[test "$libc_cv_ld_version_script_option" = "yes"])

dnl
dnl check for -fPIE/-pie support
dnl
dnl icc handles -fpie as -fp without error, so blacklist icc
dnl
AC_ARG_ENABLE(pie,AS_HELP_STRING([--disable-pie],
                        [disable position-independent executeables (PIE)]),
        USE_PIE=$enableval, USE_PIE=yes)

AC_CACHE_CHECK(for -fpie, libc_cv_fpie, [dnl
  cat > conftest.c <<EOF
int foo;
main () { return 0;}
EOF
  if test "$USE_PIE" = "yes" -a "$CC" != "icc" &&
        AC_TRY_COMMAND([${CC-cc} $CFLAGS $CPPFLAGS $LDFLAGS -pie -fpie
                              -o conftest conftest.c 1>&AS_MESSAGE_LOG_FD])
  then
    libc_cv_fpie=yes
    PIE_CFLAGS="-fpie"
    PIE_LDFLAGS="-pie"
  else
    libc_cv_fpie=no
    PIE_CFLAGS=""
    PIE_LDFLAGS=""
  fi
  rm -f conftest*])
AC_SUBST(libc_cv_fpie)
AC_SUBST(PIE_CFLAGS)
AC_SUBST(PIE_LDFLAGS)


dnl
dnl options and defaults
dnl

AC_ARG_ENABLE([prelude],
	AS_HELP_STRING([--disable-prelude],[do not use prelude]),
	WITH_PRELUDE=$enableval, WITH_PRELUDE=yes)
if test "$WITH_PRELUDE" = "yes" ; then
  AM_PATH_LIBPRELUDE([0.9.0])
  if test "$LIBPRELUDE_CONFIG" != "no" ; then
    LIBPRELUDE_CFLAGS="$LIBPRELUDE_CFLAGS -DPRELUDE=1"
  fi
fi

dnl lots of debugging information goes to /var/run/pam-debug.log
AC_ARG_ENABLE([debug],
    AS_HELP_STRING([--enable-debug],[specify you are building with debugging on]))

if test x"$enable_debug" = x"yes" ; then
   AC_DEFINE([PAM_DEBUG],,
		[lots of stuff gets written to /var/run/pam-debug.log])
fi

AC_ARG_ENABLE(securedir,
	AS_HELP_STRING([--enable-securedir=DIR],[path to location of PAMs @<:@default=$libdir/security@:>@]),
	SECUREDIR=$enableval, SECUREDIR=$libdir/security)
AC_SUBST(SECUREDIR)

AC_ARG_ENABLE([isadir],
	AS_HELP_STRING([--enable-isadir=DIR],[path to arch-specific module files @<:@default=../../(basename of $libdir)/security@:>@]),
ISA=$enableval,
ISA=../../`basename $libdir`/security)
unset mylibdirbase
AC_DEFINE_UNQUOTED(_PAM_ISA,"$ISA",[Define to the path, relative to SECUREDIR, where PAMs specific to this architecture can be found.])
AC_MSG_RESULT([Defining \$ISA to "$ISA"])

AC_ARG_ENABLE(sconfigdir,
	AS_HELP_STRING([--enable-sconfigdir=DIR],[path to module conf files @<:@default=$sysconfdir/security@:>@]),
	SCONFIGDIR=$enableval, SCONFIGDIR=$sysconfdir/security)
AC_SUBST(SCONFIGDIR)

AC_ARG_ENABLE(pamlocking,
	AS_HELP_STRING([--enable-pamlocking],[configure libpam to observe a global authentication lock]))

if test x"$enable_pamlocking" = "xyes"; then
   AC_DEFINE([PAM_LOCKING],,
	[libpam should observe a global authentication lock])
fi

AC_ARG_ENABLE(read-both-confs,
	AS_HELP_STRING([--enable-read-both-confs],[read both /etc/pam.d and /etc/pam.conf files]))

if test x"$enable_read_both_confs" = "xyes"; then
   AC_DEFINE([PAM_READ_BOTH_CONFS],,
		[read both /etc/pam.d and /etc/pam.conf files])
fi

AC_ARG_ENABLE([lckpwdf],
	AS_HELP_STRING([--disable-lckpwdf],[do not use the lckpwdf function]),
	WITH_LCKPWDF=$enableval, WITH_LCKPWDF=yes)
if test "$WITH_LCKPWDF" = "yes" ; then
    AC_DEFINE([USE_LCKPWDF], 1,
		[Define to 1 if the lckpwdf function should be used])
fi

AC_CHECK_HEADERS(paths.h)
AC_ARG_WITH(mailspool,
[  --with-mailspool        path to mail spool directory
                          [default _PATH_MAILDIR if defined in paths.h, otherwise /var/spool/mail]],
with_mailspool=${withval})
if test x$with_mailspool != x ; then
	pam_mail_spool="\"$with_mailspool\""
else
	AC_RUN_IFELSE([AC_LANG_SOURCE([[
#include <paths.h>
int main() {
#ifdef _PATH_MAILDIR
exit(0);
#else
exit(1);
#endif
}]])],[pam_mail_spool="_PATH_MAILDIR"],[pam_mail_spool="\"/var/spool/mail\""],[pam_mail_spool="\"/var/spool/mail\""])
fi
AC_DEFINE_UNQUOTED(PAM_PATH_MAILDIR, $pam_mail_spool,
	[Path where mails are stored])

AC_ARG_WITH(xauth,
[  --with-xauth            additional path to check for xauth when it is called from pam_xauth
                          [added to the default of /usr/X11R6/bin/xauth, /usr/bin/xauth, /usr/bin/X11/xauth]],
pam_xauth_path=${withval})
if test x$with_xauth = x ; then
	AC_PATH_PROG(pam_xauth_path, xauth)
dnl There is no sense in adding the first default path
	if test x$pam_xauth_path = x/usr/X11R6/bin/xauth ; then
		unset pam_xauth_path
	fi
fi

if test x$pam_xauth_path != x ; then
	AC_DEFINE_UNQUOTED(PAM_PATH_XAUTH, "$pam_xauth_path",
	[Additional path of xauth executable])
fi

dnl Checks for the existence of libdl - on BSD and Tru64 its part of libc
AC_CHECK_LIB([dl], [dlopen], LIBDL="-ldl", LIBDL="")
AC_SUBST(LIBDL)

# Check for cracklib
AC_ARG_ENABLE([cracklib],
        AS_HELP_STRING([--disable-cracklib],[do not use cracklib]),
        WITH_CRACKLIB=$enableval, WITH_CRACKLIB=yes)
if test x"$WITH_CRACKLIB" != xno ; then
        AC_CHECK_HEADERS([crack.h],
              AC_CHECK_LIB([crack], [FascistCheck], LIBCRACK="-lcrack", LIBCRACK=""))
else
	LIBCRACK=""
fi
if test -n "$LIBCRACK"; then
	AC_DEFINE([HAVE_LIBCRACK], [1], [Define to 1 if you have cracklib.])
fi
AC_SUBST(LIBCRACK)
AM_CONDITIONAL([HAVE_LIBCRACK], [test -n "$LIBCRACK"])

dnl Look for Linux Auditing library - see documentation
AC_ARG_ENABLE([audit],
        AS_HELP_STRING([--disable-audit],[do not enable audit support]),
        WITH_LIBAUDIT=$enableval, WITH_LIBAUDIT=yes)
if test x"$WITH_LIBAUDIT" != xno ; then
        AC_CHECK_HEADER([libaudit.h],
              [AC_CHECK_LIB(audit, audit_log_acct_message, LIBAUDIT=-laudit, LIBAUDIT="")
	       AC_CHECK_TYPE([struct audit_tty_status],
		             [HAVE_AUDIT_TTY_STATUS=yes],
			     [HAVE_AUDIT_TTY_STATUS=""],
			     [#include <libaudit.h>])]
        )
        if test ! -z "$LIBAUDIT" -a "$ac_cv_header_libaudit_h" != "no" ; then
            AC_DEFINE([HAVE_LIBAUDIT], 1, [Define to 1 if audit support should be compiled in.])
        fi
        if test ! -z "$HAVE_AUDIT_TTY_STATUS" ; then
            AC_DEFINE([HAVE_AUDIT_TTY_STATUS], 1, [Define to 1 if struct audit_tty_status exists.])

            AC_CHECK_MEMBERS([struct audit_tty_status.log_passwd], [],
                            AC_MSG_WARN([audit_tty_status.log_passwd is not available.  The log_passwd option is disabled.]),
                            [[#include <libaudit.h>]])
        fi
else
	LIBAUDIT=""
fi
AC_SUBST(LIBAUDIT)
AM_CONDITIONAL([HAVE_AUDIT_TTY_STATUS],
	       [test "x$HAVE_AUDIT_TTY_STATUS" = xyes])

AC_CHECK_HEADERS(xcrypt.h crypt.h)
AS_IF([test "x$ac_cv_header_xcrypt_h" = "xyes"],
  [crypt_libs="xcrypt crypt"],
  [crypt_libs="crypt"])

BACKUP_LIBS=$LIBS
AC_SEARCH_LIBS([crypt],[$crypt_libs], LIBCRYPT="${ac_lib:+-l$ac_lib}", LIBCRYPT="")
AC_CHECK_FUNCS(crypt_r crypt_gensalt_r)
LIBS=$BACKUP_LIBS
AC_SUBST(LIBCRYPT)
if test "$LIBCRYPT" = "-lxcrypt" -a "$ac_cv_header_xcrypt_h" = "yes" ; then
	AC_DEFINE([HAVE_LIBXCRYPT], 1, [Define to 1 if xcrypt support should be compiled in.])
fi

AC_ARG_WITH([randomdev], AS_HELP_STRING([--with-randomdev=(<path>|yes|no)],[use specified random device instead of /dev/urandom or 'no' to disable]), opt_randomdev=$withval)
if test "$opt_randomdev" = yes -o -z "$opt_randomdev"; then
       opt_randomdev="/dev/urandom"
elif test "$opt_randomdev" = no; then
       opt_randomdev=
fi
if test -n "$opt_randomdev"; then
       AC_DEFINE_UNQUOTED(PAM_PATH_RANDOMDEV, "$opt_randomdev", [Random device path.])
fi

dnl check for libdb or libndbm as fallback. Some libndbm compat
dnl libraries are unuseable, so try libdb first.
AC_ARG_ENABLE([db],
        AS_HELP_STRING([--enable-db=(db|ndbm|yes|no)],[Default behavior 'yes', which is to check for libdb first, followed by ndbm. Use 'no' to disable db support.]),
        WITH_DB=$enableval, WITH_DB=yes)
AC_ARG_WITH([db-uniquename],
	AS_HELP_STRING([--with-db-uniquename=extension],[Unique name for db libraries and functions.]))
if test x"$WITH_DB" != xno ; then
        if test x"$WITH_DB" = xyes -o x"$WITH_DB" = xdb ; then
              old_libs=$LIBS
              LIBS="$LIBS -ldb$with_db_uniquename"
              AC_CHECK_FUNCS([db_create$with_db_uniquename db_create dbm_store$with_db_uniquename dbm_store],
                [LIBDB="-ldb$with_db_uniquename"; break])
              LIBS=$old_libs
        fi
        if test -z "$LIBDB" ; then
            AC_CHECK_LIB([ndbm],[dbm_store], LIBDB="-lndbm", LIBDB="")
            if test ! -z "$LIBDB" ; then
                AC_CHECK_HEADERS(ndbm.h)
            fi
        else
            AC_CHECK_HEADERS(db.h)
        fi
fi
AC_SUBST(LIBDB)
AM_CONDITIONAL([HAVE_LIBDB], [test ! -z "$LIBDB"])

AC_ARG_ENABLE([nis],
        AS_HELP_STRING([--disable-nis], [Disable building NIS/YP support in pam_unix]))

AS_IF([test "x$enable_nis" != "xno"], [
  old_CFLAGS=$CFLAGS
  old_CPPFLAGS=$CPPFLAGS
  old_LIBS=$LIBS

  dnl if there's libtirpc available, prefer that over the system
  dnl implementation.
  PKG_CHECK_MODULES([TIRPC], [libtirpc], [
    CFLAGS="$CFLAGS $TIRPC_CFLAGS"
    CPPFLAGS="$CPPFLAGS $TIRPC_CFLAGS"
    LIBS="$LIBS $TIRPC_LIBS"
  ], [:;])

  PKG_CHECK_MODULES([NSL], [libnsl], [],
    [AC_CHECK_LIB([nsl],[yp_match],[NSL_LIBS="-lnsl"],[NSL_LIBS=""])])
  CFLAGS="$CFLAGS $NSL_CFLAGS"
  CPPFLAGS="$CPPFLAGS $NSL_CFLAGS"
  LIBS="$LIBS $NSL_LIBS"

  AC_CHECK_FUNCS([yp_get_default_domain yperr_string yp_master yp_bind yp_match yp_unbind])
  AC_CHECK_FUNCS([getrpcport rpcb_getaddr])
  AC_CHECK_HEADERS([rpc/rpc.h rpcsvc/ypclnt.h rpcsvc/yp_prot.h])
  AC_CHECK_DECLS([getrpcport], , , [
    #if HAVE_RPC_RPC_H
    # include <rpc/rpc.h>
    #endif
  ])

  CFLAGS="$old_CFLAGS"
  CPPFLAGS="$old_CPPFLAGS"
  LIBS="$old_LIBS"
])

AC_SUBST([NIS_CFLAGS])
AC_SUBST([NIS_LIBS])

AC_ARG_ENABLE([selinux],
        AS_HELP_STRING([--disable-selinux],[do not use SELinux]),
        WITH_SELINUX=$enableval, WITH_SELINUX=yes)
if test "$WITH_SELINUX" = "yes" ; then
  AC_CHECK_LIB([selinux],[getfilecon], LIBSELINUX="-lselinux", LIBSELINUX="")
else
  LIBSELINUX=""
fi
AC_SUBST(LIBSELINUX)
AM_CONDITIONAL([HAVE_LIBSELINUX], [test ! -z "$LIBSELINUX"])
if test ! -z "$LIBSELINUX" ; then
    AC_DEFINE([WITH_SELINUX], 1, [Defined if SE Linux support is compiled in])
    BACKUP_LIBS=$LIBS
    LIBS="$LIBS $LIBSELINUX"
    AC_CHECK_FUNCS(setkeycreatecon)
    AC_CHECK_FUNCS(getseuser)
    LIBS=$BACKUP_LIBS
fi

AC_ARG_ENABLE([econf],
  AS_HELP_STRING([--disable-econf], [do not use libeconf]),
  [WITH_ECONF=$enableval], WITH_ECONF=yes)
if test "$WITH_ECONF" = "yes" ; then
  PKG_CHECK_MODULES([ECONF], [libeconf], [],
  [AC_CHECK_LIB([econf],[econf_readDirs],[ECONF_LIBS="-leconf"],[ECONF_LIBS=""])])
  if test -n "$ECONF_LIBS" ; then
    ECONF_CFLAGS="-DUSE_ECONF=1 $ECONF_CFLAGS"
  fi
fi
AC_SUBST([ECONF_CFLAGS])
AC_SUBST([ECONF_LIBS])
AC_ARG_ENABLE([vendordir],
  AS_HELP_STRING([--enable-vendordir=DIR], [Directory for distribution provided configuration files]),,[])
AC_SUBST([VENDORDIR], [$enable_vendordir])
AM_CONDITIONAL([HAVE_VENDORDIR], [test "x$enable_vendordir" != x])

dnl Checks for header files.
AC_HEADER_DIRENT
AC_HEADER_STDC
AC_HEADER_SYS_WAIT
AC_CHECK_HEADERS(fcntl.h limits.h malloc.h sys/file.h sys/ioctl.h sys/time.h syslog.h net/if.h termio.h unistd.h sys/fsuid.h inittypes.h sys/vfs.h linux/magic.h)

dnl For module/pam_lastlog
AC_CHECK_HEADERS(lastlog.h utmp.h utmpx.h)

dnl Checks for typedefs, structures, and compiler characteristics.
AC_C_BIGENDIAN
AC_C_CONST
AC_TYPE_UID_T
AC_TYPE_OFF_T
AC_TYPE_PID_T
AC_TYPE_SIZE_T
AC_HEADER_TIME
AC_STRUCT_TM

dnl Checks for library functions.
AC_TYPE_GETGROUPS
AC_PROG_GCC_TRADITIONAL
AC_FUNC_MEMCMP
AC_FUNC_VPRINTF
AC_CHECK_FUNCS(fseeko getdomainname gethostname gettimeofday lckpwdf mkdir select)
AC_CHECK_FUNCS(strcspn strdup strspn strstr strtol uname)
AC_CHECK_FUNCS(getutent_r getpwnam_r getpwuid_r getgrnam_r getgrgid_r getspnam_r)
AC_CHECK_FUNCS(getgrouplist getline getdelim)
AC_CHECK_FUNCS(inet_ntop inet_pton innetgr)
AC_CHECK_FUNCS([ruserok_af ruserok], [break])
BACKUP_LIBS=$LIBS
LIBS="$LIBS -lutil"
AC_CHECK_FUNCS([logwtmp])
LIBS=$BACKUP_LIBS

AM_CONDITIONAL([COND_BUILD_PAM_RHOSTS], [test "$ac_cv_func_ruserok_af" = yes -o "$ac_cv_func_ruserok" = yes])
AM_CONDITIONAL([COND_BUILD_PAM_LASTLOG], [test "$ac_cv_func_logwtmp" = yes])

AC_CHECK_FUNCS(unshare, [UNSHARE=yes], [UNSHARE=no])
AM_CONDITIONAL([HAVE_UNSHARE], [test "$UNSHARE" = yes])

AC_ARG_ENABLE([regenerate-docu],
  AS_HELP_STRING([--disable-regenerate-docu],[Don't re-build documentation from XML sources]),
  [enable_docu=$enableval], [enable_docu=yes])
dnl
dnl Check for xsltproc
dnl
AC_PATH_PROG([XSLTPROC], [xsltproc])
if test -z "$XSLTPROC"; then
     enable_docu=no
fi
AC_PATH_PROG([XMLLINT], [xmllint],[/bin/true])
dnl check for DocBook DTD and stylesheets in the local catalog.
JH_CHECK_XML_CATALOG([-//OASIS//DTD DocBook XML V4.4//EN],
                [DocBook XML DTD V4.4], [], enable_docu=no)
JH_CHECK_XML_CATALOG([http://docbook.sourceforge.net/release/xsl/current/manpages/docbook.xsl],
                [DocBook XSL Stylesheets], [], enable_docu=no)

AC_PATH_PROG([BROWSER], [w3m])
if test ! -z "$BROWSER"; then
     BROWSER="$BROWSER -T text/html -dump"
else
     AC_PATH_PROG([BROWSER], [elinks])
     if test ! -z "$BROWSER"; then
          BROWSER="$BROWSER -no-numbering -no-references -dump"
     else
          enable_docu=no
     fi
fi

AC_PATH_PROG([FO2PDF], [fop])

AM_CONDITIONAL(ENABLE_REGENERATE_MAN, test x$enable_docu != xno)
AM_CONDITIONAL(ENABLE_GENERATE_PDF, test ! -z "$FO2PDF")


AM_GNU_GETTEXT_VERSION([0.18.3])
AM_GNU_GETTEXT([external])
AC_CHECK_FUNCS(dngettext)

AH_BOTTOM([#ifdef ENABLE_NLS
#include <libintl.h>
#define _(msgid) dgettext(PACKAGE, msgid)
#define N_(msgid) msgid
#else
#define _(msgid) (msgid)
#define N_(msgid) msgid
#endif /* ENABLE_NLS */])

dnl
dnl Check for the availability of the kernel key management facility
dnl - The pam_keyinit module only requires the syscalls, not the error codes
dnl
AC_CHECK_DECL(__NR_keyctl, [have_key_syscalls=1],[have_key_syscalls=0],[#include <sys/syscall.h>])
AC_CHECK_DECL(ENOKEY, [have_key_errors=1],[have_key_errors=0],[#include <errno.h>])

HAVE_KEY_MANAGEMENT=0
if test $have_key_syscalls$have_key_errors = 11
then
	HAVE_KEY_MANAGEMENT=1
fi

if test $HAVE_KEY_MANAGEMENT = 1; then
   AC_DEFINE([HAVE_KEY_MANAGEMENT], 1,
	     [Defined if the kernel key management facility is available])
fi
AC_SUBST([HAVE_KEY_MANAGEMENT], $HAVE_KEY_MANAGEMENT)

AM_CONDITIONAL([HAVE_KEY_MANAGEMENT], [test "$have_key_syscalls" = 1])

dnl Files to be created from when we run configure
AC_CONFIG_FILES([Makefile libpam/Makefile libpamc/Makefile libpamc/test/Makefile \
	libpam_misc/Makefile conf/Makefile conf/pam_conv1/Makefile \
	po/Makefile.in \
	modules/Makefile \
	modules/pam_access/Makefile modules/pam_cracklib/Makefile \
        modules/pam_debug/Makefile modules/pam_deny/Makefile \
	modules/pam_echo/Makefile modules/pam_env/Makefile \
	modules/pam_faildelay/Makefile \
	modules/pam_filter/Makefile modules/pam_filter/upperLOWER/Makefile \
	modules/pam_ftp/Makefile modules/pam_group/Makefile \
	modules/pam_issue/Makefile modules/pam_keyinit/Makefile \
	modules/pam_lastlog/Makefile modules/pam_limits/Makefile \
	modules/pam_listfile/Makefile modules/pam_localuser/Makefile \
	modules/pam_loginuid/Makefile modules/pam_mail/Makefile \
	modules/pam_mkhomedir/Makefile modules/pam_motd/Makefile \
	modules/pam_namespace/Makefile \
	modules/pam_nologin/Makefile modules/pam_permit/Makefile \
	modules/pam_pwhistory/Makefile modules/pam_rhosts/Makefile \
	modules/pam_rootok/Makefile modules/pam_exec/Makefile \
	modules/pam_securetty/Makefile modules/pam_selinux/Makefile \
	modules/pam_sepermit/Makefile \
	modules/pam_shells/Makefile modules/pam_stress/Makefile \
	modules/pam_succeed_if/Makefile modules/pam_tally/Makefile \
	modules/pam_tally2/Makefile modules/pam_time/Makefile \
	modules/pam_timestamp/Makefile modules/pam_tty_audit/Makefile \
	modules/pam_umask/Makefile \
	modules/pam_unix/Makefile modules/pam_userdb/Makefile \
	modules/pam_warn/Makefile modules/pam_wheel/Makefile \
	modules/pam_xauth/Makefile doc/Makefile doc/specs/Makefile \
	doc/man/Makefile doc/sag/Makefile doc/adg/Makefile \
	doc/mwg/Makefile examples/Makefile tests/Makefile \
	xtests/Makefile])
AC_OUTPUT
