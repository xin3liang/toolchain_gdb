dnl Autoconf configure script for GDB server.
dnl Copyright 2000, 2002, 2003, 2004, 2005 Free Software Foundation, Inc.
dnl
dnl This file is part of GDB.
dnl
dnl This program is free software; you can redistribute it and/or modify
dnl it under the terms of the GNU General Public License as published by
dnl the Free Software Foundation; either version 2 of the License, or
dnl (at your option) any later version.
dnl
dnl This program is distributed in the hope that it will be useful,
dnl but WITHOUT ANY WARRANTY; without even the implied warranty of
dnl MERCHANTABILITY or FITNESS FOR A PARTICULAR PURPOSE.  See the
dnl GNU General Public License for more details.
dnl
dnl You should have received a copy of the GNU General Public License
dnl along with this program; if not, write to the Free Software
dnl Foundation, Inc., 59 Temple Place - Suite 330, Boston, MA 02111-1307, USA.

dnl Process this file with autoconf to produce a configure script.

AC_PREREQ(2.59)dnl

AC_INIT(server.c)
AC_CONFIG_HEADER(config.h:config.in)

AC_PROG_CC

AC_CANONICAL_SYSTEM

AC_PROG_INSTALL

AC_ARG_PROGRAM

AC_HEADER_STDC

AC_CHECK_HEADERS(sgtty.h termio.h termios.h sys/reg.h string.h dnl
		 proc_service.h sys/procfs.h thread_db.h linux/elf.h dnl
		 stdlib.h unistd.h)

AC_CHECK_DECLS(strerror)

AC_CHECK_TYPES(socklen_t, [], [],
[#include <sys/types.h>
#include <sys/socket.h>
])
. ${srcdir}/configure.srv

if test "${srv_linux_usrregs}" = "yes"; then
  AC_DEFINE(HAVE_LINUX_USRREGS, 1,
	    [Define if the target supports PTRACE_PEEKUSR for register ]
	    [access.])
fi

if test "${srv_linux_regsets}" = "yes"; then
  AC_DEFINE(HAVE_LINUX_REGSETS, 1,
	    [Define if the target supports register sets.])

  AC_MSG_CHECKING(for PTRACE_GETREGS)
  AC_CACHE_VAL(gdbsrv_cv_have_ptrace_getregs,
  [AC_TRY_COMPILE([#include <sys/ptrace.h>],
		  [PTRACE_GETREGS;],
		  [gdbsrv_cv_have_ptrace_getregs=yes],
		  [gdbsrv_cv_have_ptrace_getregs=no])])
  AC_MSG_RESULT($gdbsrv_cv_have_ptrace_getregs)
  if test "${gdbsrv_cv_have_ptrace_getregs}" = "yes"; then
    AC_DEFINE(HAVE_PTRACE_GETREGS, 1,
	      [Define if the target supports PTRACE_GETREGS for register ]
	      [access.])
  fi

  AC_MSG_CHECKING(for PTRACE_GETFPXREGS)
  AC_CACHE_VAL(gdbsrv_cv_have_ptrace_getfpxregs,
  [AC_TRY_COMPILE([#include <sys/ptrace.h>],
		  [PTRACE_GETFPXREGS;],
		  [gdbsrv_cv_have_ptrace_getfpxregs=yes],
		  [gdbsrv_cv_have_ptrace_getfpxregs=no])])
  AC_MSG_RESULT($gdbsrv_cv_have_ptrace_getfpxregs)
  if test "${gdbsrv_cv_have_ptrace_getfpxregs}" = "yes"; then
    AC_DEFINE(HAVE_PTRACE_GETFPXREGS, 1,
	      [Define if the target supports PTRACE_GETFPXREGS for extended ]
	      [register access.])
  fi
fi

if test "$ac_cv_header_sys_procfs_h" = yes; then
  BFD_HAVE_SYS_PROCFS_TYPE(lwpid_t)
  BFD_HAVE_SYS_PROCFS_TYPE(psaddr_t)
  BFD_HAVE_SYS_PROCFS_TYPE(prgregset_t)
  BFD_HAVE_SYS_PROCFS_TYPE(prfpregset_t)

  dnl Check for broken prfpregset_t type

  dnl For Linux/i386, glibc 2.1.3 was released with a bogus
  dnl prfpregset_t type (it's a typedef for the pointer to a struct
  dnl instead of the struct itself).  We detect this here, and work
  dnl around it in gdb_proc_service.h.

  if test $bfd_cv_have_sys_procfs_type_prfpregset_t = yes; then
    AC_MSG_CHECKING(whether prfpregset_t type is broken)
    AC_CACHE_VAL(gdb_cv_prfpregset_t_broken,
      [AC_TRY_RUN([#include <sys/procfs.h>
       int main ()
       {
         if (sizeof (prfpregset_t) == sizeof (void *))
           return 1;
         return 0;
       }],
       gdb_cv_prfpregset_t_broken=no,
       gdb_cv_prfpregset_t_broken=yes,
       gdb_cv_prfpregset_t_broken=yes)])
    AC_MSG_RESULT($gdb_cv_prfpregset_t_broken)
    if test $gdb_cv_prfpregset_t_broken = yes; then
      AC_DEFINE(PRFPREGSET_T_BROKEN, 1,
		[Define if the prfpregset_t type is broken.])
    fi
  fi

  BFD_HAVE_SYS_PROCFS_TYPE(elf_fpregset_t)
fi

srv_thread_depfiles=
srv_libs=
USE_THREAD_DB=

if test "$srv_linux_thread_db" = "yes"; then
  SRV_CHECK_THREAD_DB
  if test "$srv_cv_thread_db" = no; then
    AC_WARN([Could not find libthread_db.])
    AC_WARN([Disabling thread support in gdbserver.])
    srv_linux_thread_db=no
  else
    srv_libs="$srv_cv_thread_db"
  fi
  old_LDFLAGS="$LDFLAGS"
  LDFLAGS="$LDFLAGS -Wl,--export-dynamic"
  AC_TRY_LINK([], [], [RDYNAMIC=-Wl,--export-dynamic], [RDYNAMIC=])
  AC_SUBST(RDYNAMIC)
  LDFLAGS="$old_LDFLAGS"
fi

if test "$srv_linux_thread_db" = "yes"; then
  srv_thread_depfiles="thread-db.o proc-service.o"
  USE_THREAD_DB="-DUSE_THREAD_DB"
  AC_CACHE_CHECK([for TD_VERSION], gdbsrv_cv_have_td_version,
  [AC_TRY_COMPILE([#include <thread_db.h>], [TD_VERSION;],
                  [gdbsrv_cv_have_td_version=yes],
                  [gdbsrv_cv_have_td_version=no])])
  if test $gdbsrv_cv_have_td_version = yes; then
    AC_DEFINE(HAVE_TD_VERSION, 1, [Define if TD_VERSION is available.])
  fi
fi

GDBSERVER_DEPFILES="$srv_regobj $srv_tgtobj $srv_thread_depfiles"
GDBSERVER_LIBS="$srv_libs"

AC_SUBST(GDBSERVER_DEPFILES)
AC_SUBST(GDBSERVER_LIBS)
AC_SUBST(USE_THREAD_DB)

AC_OUTPUT(Makefile,
[case x$CONFIG_HEADERS in
xconfig.h:config.in)
echo > stamp-h ;;
esac
])
