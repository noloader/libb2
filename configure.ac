AC_PREREQ([2.61])
AC_INIT([libb2], [0.98.1], [contact@blake2.net], [libb2], [https://blake2.net])
AC_CONFIG_SRCDIR([src/blake2b.c])
AC_CONFIG_HEADERS([src/config.h])
AC_CONFIG_AUX_DIR([build-aux])
AM_INIT_AUTOMAKE([foreign 1.9])
AC_CONFIG_MACRO_DIR([m4])

dnl https://www.gnu.org/software/libtool/manual/html_node/Updating-version-info.html
dnl interface, revision, age
B2_LIBRARY_VERSION=1:4:0
AC_SUBST(B2_LIBRARY_VERSION)

AC_LANG_C
AC_PROG_CC
AC_PROG_CC_C99
AC_PROG_SED
AC_PROG_GREP
AC_PROG_EGREP
AC_PROG_LIBTOOL
PKG_PROG_PKG_CONFIG

dnl Parse the autotools triplet. The Host is the one the package
dnl runs on. The Build is the machine that does the compiling.
dnl if $build != $host, then its a cross-compile.
AC_CANONICAL_HOST

dnl Set the language
AC_LANG([C])
# LT_LANG([C])

AC_CHECK_FUNCS(explicit_bzero)
AC_CHECK_FUNCS(explicit_memset)
AC_CHECK_FUNCS(memset_s)
AC_CHECK_HEADERS([stddef.h stdint.h stdlib.h string.h])
dnl AX_FORCEINLINE()

AC_C_BIGENDIAN(
    AC_DEFINE(NATIVE_BIG_ENDIAN, 1, [machine is big-endian]),
    AC_DEFINE(NATIVE_LITTLE_ENDIAN, 1, [machine is little-endian]),
    AC_MSG_ERROR(unknown endianness),
    AC_MSG_ERROR(universal endianness not supported)
)

dnl Checks for typedefs, structures, and compiler characteristics.
AC_C_INLINE
AC_TYPE_SIZE_T
AC_TYPE_UINT32_T
AC_TYPE_UINT64_T
AC_TYPE_UINT8_T

dnl Checks for library functions.
AC_CHECK_FUNCS([memset])

dnl AM_PROG_AR
AC_PROG_LIBTOOL
dnl LT_INIT


dnl save these once at the start of the script
saved_CPPFLAGS="$CPPFLAGS"
saved_CFLAGS="$CFLAGS"
saved_LDFLAGS="$LDFLAGS"


dnl since default=yes, use option --disable-fat
AC_ARG_ENABLE(fat,
AC_HELP_STRING([--disable-fat],
               [build a fat binary on systems that support it [default=enabled]]),
               [case "x$enable_fat" in
                  xyes|xno) ;;
                  *) AC_MSG_ERROR([bad value $enable_fat for --enable-fat, need yes or no]) ;;
                esac]
)

dnl canonicalize --enable-fat
AS_IF([test "x$enable_fat" = "xno"],
  [AC_SUBST([enable_fat], [no])],
  [AC_SUBST([enable_fat], [yes])]
)

dnl since default=no, use option --enable-native
AC_ARG_ENABLE(native,
AC_HELP_STRING([--enable-native],
               [build a binary optimized for the compile time CPU on systems that support it [default=disabled]]),
               [case "x$enable_native" in
                  xyes|xno) ;;
                  *) AC_MSG_ERROR([bad value $enable_native for --enable-native, need yes or no]) ;;
                esac]
)

dnl canonicalize --enable-native
AS_IF([test "x$enable_native" = "xyes"],
  [AC_SUBST([enable_native], [yes])],
  [AC_SUBST([enable_native], [no])]
)

dnl since default=no, use option --enable-openmp
AC_ARG_ENABLE(openmp,
AC_HELP_STRING([--enable-openmp],
               [build a binary with OpenMP threads [default=disabled]]),
               [case "x$enable_openmp" in
                  xyes|xno) ;;
                  *) AC_MSG_ERROR([bad value $enable_openmp for --enable-openmp, need yes or no]) ;;
                esac]
)

dnl only test for OpenMP if enabled
AS_IF([test "x$enable_openmp" = "xyes"],
[
  AX_OPENMP([AC_SUBST([enable_openmp], [yes])],
            [AC_MSG_ERROR([failed to enable OpenMP])
             AC_SUBST([enable_openmp], [no])])
],
[
  AC_SUBST([enable_openmp], [no])
])


dnl sanity check: enable-fat only applies to x86-based machines
AS_CASE([$host],
  [ i386-* | i486-* | i586-* | i686-* | i86pc-* | amd64-* | x86_64-* ], [:],
  dnl default case
  [
    AC_MSG_NOTICE([disabling fat build because machine is not x86])
    AC_SUBST([enable_fat], [no])
  ]
)

dnl sanity check: enable-fat and enable-native are mutually exclusive
AS_IF([test "x$enable_fat" = "xyes" && test "x$enable_native" = "xyes"],
  [
    AC_MSG_NOTICE([disabling fat build because both fat and native are specified.])
    AC_SUBST([enable_fat], [no])
  ]
)


dnl add -Wall if not specified by user and option is available
have_wall=`echo "$AM_CFLAGS $saved_CFLAGS" | $GREP -c '\-Wall'`
AS_IF([test "$have_wall" -eq 0],
  [AX_CHECK_COMPILE_FLAG([-Wall], [AM_CFLAGS="$AM_CFLAGS -Wall"])]
)

dnl add -g2 if not specified by user and option is available
have_syms=`echo "$AM_CFLAGS $saved_CFLAGS" | $EGREP -c '\-g |\-g0|\-g1|\-g2|\-g3|\-ggdb'`
AS_IF([test "$have_syms" -eq 0],
  [AX_CHECK_COMPILE_FLAG([-g2], [AM_CFLAGS="$AM_CFLAGS -g2"])]
)

dnl add -O3 if not specified by user and option is available
have_opts=`echo "$AM_CFLAGS $saved_CFLAGS" | $EGREP -c '\-O0|\-O1|\-O2|\-O3|\-O4|\-O5|\-xO0|\-xO1|\-xO2|\-xO3|\-xO4|\-xO5|\-Os|\-Og|\-Ofast'`
AS_IF([test "$have_opts" -eq 0],
[
  AX_CHECK_COMPILE_FLAG([-O3], [AM_CFLAGS="$AM_CFLAGS -O3"], [
    AX_CHECK_COMPILE_FLAG([-xO3], [AM_CFLAGS="$AM_CFLAGS -xO3"])
])])


dnl Check for -march=native, -mtune=native and -native in turn.
dnl Not all architectures support -march=native
AS_IF([test "x$enable_native" = "xyes"],
[
  AX_CHECK_COMPILE_FLAG([-march=native], [AM_CFLAGS="$AM_CFLAGS -march=native"], [
    AX_CHECK_COMPILE_FLAG([-mtune=native], [AM_CFLAGS="$AM_CFLAGS -mtune=native"], [
      AX_CHECK_COMPILE_FLAG([-native], [AM_CFLAGS="$AM_CFLAGS -native"], [
        AC_MSG_ERROR([failed to determine native option])
])])])])


dnl The fat build needs a compiler that knows instruction sets used in the makefile.
dnl Don't set the flags in AM_CFLAGS. The makefile automatically adds the arch option.
AS_IF([test "x$enable_fat" = "xyes"],
[
  AX_CHECK_COMPILE_FLAG([-msse2],   [], AC_MSG_ERROR([Compiler does not know -msse2.]))
  AX_CHECK_COMPILE_FLAG([-mssse3],  [], AC_MSG_ERROR([Compiler does not know -mssse3.]))
  AX_CHECK_COMPILE_FLAG([-msse4.1], [], AC_MSG_ERROR([Compiler does not know -msse4.1.]))
  AX_CHECK_COMPILE_FLAG([-mxop],    [], AC_MSG_ERROR([Compiler does not know -mxop.]))
  AX_CHECK_COMPILE_FLAG([-mavx],    [], AC_MSG_ERROR([Compiler does not know -mavx.]))
])

dnl Generate list of arch options for native. Early Clang does not enable them with -march-native.
AS_IF([test "x$enable_native" = "xyes"],
[
  AX_EXT
  AM_CFLAGS="${AM_CFLAGS} ${SIMD_FLAGS} ${CPUEXT_FLAGS}"
])


dnl Restore user's flags
CPPFLAGS="$saved_CPPFLAGS"
CFLAGS="$saved_CFLAGS"
LDFLAGS="$saved_LDFLAGS"


dnl Cleanup whitespace
AM_CPPFLAGS=`echo "$AM_CPPFLAGS" | $SED 's/^[ \t]*//'`
AM_CFLAGS=`echo "$AM_CFLAGS" | $SED 's/^[ \t]*//'`
AM_LDFLAGS=`echo "$AM_LDFLAGS" | $SED 's/^[ \t]*//'`


dnl Set OpenMP flags if enabled
AC_SUBST(OPENMP_CPPFLAGS)
AC_SUBST(OPENMP_CFLAGS)
AC_SUBST(OPENMP_LDFLAGS)
AC_SUBST(OPENMP_LIBS)


dnl set our flags for the Makefiles
AC_SUBST(AM_CPPFLAGS)
AC_SUBST(AM_CFLAGS)
AC_SUBST(AM_LDFLAGS)


AM_CONDITIONAL([USE_FAT], [test "x$enable_fat" = "xyes"])
dnl Only move away from ref with SSSE3; SSE2 is generally slower
AM_CONDITIONAL([USE_SSE], [test "x$ax_cv_have_ssse3_ext" = "xyes"])


PKG_INSTALLDIR
AC_CONFIG_FILES([Makefile
                 src/Makefile
                 libb2.pc
                 ])
AC_OUTPUT

echo ""
echo "****************************************"
echo "          LIBB2 OPTION SUMMARY          "
echo "****************************************"
echo "enable-fat: $enable_fat"
echo "enable-native: $enable_native"
echo "enable-openmp: $enable_openmp"

dnl these are populated by AX_EXT
AS_IF([test "x$enable_native" = "xyes"],
[
  echo ""
  echo "  SSE2: $ax_cv_have_sse2_ext"
  echo " SSSE3: $ax_cv_have_ssse3_ext"
  echo "   XOP: $ax_cv_have_xop_ext"
  echo "SSE4.1: $ax_cv_have_sse41_ext"
  echo "SSE4.2: $ax_cv_have_sse42_ext"
  echo "   AVX: $ax_cv_have_avx_ext"
  echo "  AVX2: $ax_cv_have_avx2_ext"
])
echo "****************************************"

dnl change to true to print the flags
AS_IF([false],
[
  echo ""
  echo "AM_CPPFLAGS: $AM_CPPFLAGS"
  echo "AM_CFLAGS: $AM_CFLAGS"
  echo "AM_LDFLAGS: $AM_LDFLAGS"

  echo ""
  echo "CPPFLAGS: $CPPFLAGS"
  echo "CFLAGS: $CFLAGS"
  echo "LDFLAGS: $LDFLAGS"

  echo ""
  echo "OPENMP_CFLAGS: $OPENMP_CFLAGS"
  echo "OPENMP_LDFLAGS: $OPENMP_LDFLAGS"
  echo "OPENMP_LIBS: $OPENMP_LIBS"
  echo "****************************************"
])
