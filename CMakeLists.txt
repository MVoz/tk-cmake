#                     C M A K E L I S T S . T X T
# SCL
#
# Copyright (c) 2010 United States Government as represented by
# the U.S. Army Research Laboratory.
#
# Redistribution and use in source and binary forms, with or without
# modification, are permitted provided that the following conditions
# are met:
#
# 1. Redistributions of source code must retain the above copyright
# notice, this list of conditions and the following disclaimer.
#
# 2. Redistributions in binary form must reproduce the above
# copyright notice, this list of conditions and the following
# disclaimer in the documentation and/or other materials provided
# with the distribution.
#
# 3. The name of the author may not be used to endorse or promote
# products derived from this software without specific prior written
# permission.
#
# THIS SOFTWARE IS PROVIDED BY THE AUTHOR ``AS IS'' AND ANY EXPRESS
# OR IMPLIED WARRANTIES, INCLUDING, BUT NOT LIMITED TO, THE IMPLIED
# WARRANTIES OF MERCHANTABILITY AND FITNESS FOR A PARTICULAR PURPOSE
# ARE DISCLAIMED. IN NO EVENT SHALL THE AUTHOR BE LIABLE FOR ANY
# DIRECT, INDIRECT, INCIDENTAL, SPECIAL, EXEMPLARY, OR CONSEQUENTIAL
# DAMAGES (INCLUDING, BUT NOT LIMITED TO, PROCUREMENT OF SUBSTITUTE
# GOODS OR SERVICES; LOSS OF USE, DATA, OR PROFITS; OR BUSINESS
# INTERRUPTION) HOWEVER CAUSED AND ON ANY THEORY OF LIABILITY,
# WHETHER IN CONTRACT, STRICT LIABILITY, OR TORT (INCLUDING
# NEGLIGENCE OR OTHERWISE) ARISING IN ANY WAY OUT OF THE USE OF THIS
# SOFTWARE, EVEN IF ADVISED OF THE POSSIBILITY OF SUCH DAMAGE.

# *******************************************************************
# ***                     Tk CMakeLists.txt                       ***
# *******************************************************************

# Minimum required version of CMake
cmake_minimum_required(VERSION 3.1.3)

# set CMake project name
project(TK)

# Without Tcl libraries AND sources, we can't build Tk.  Use TCL_SRC_PREFIX
# and TCL_BIN_PREFIX to identify the location of the Tcl we are to use.
IF(TCL_SRC_PREFIX AND TCL_BIN_PREFIX)
  SET(TCL_INCLUDE_DIRS ${TCL_SRC_PREFIX}/generic ${TCL_SRC_PREFIX}/libtommath)
  IF(WIN32)
    SET(TCL_INCLUDE_DIRS ${TCL_INCLUDE_DIRS} ${TCL_SRC_PREFIX}/win)
  ELSE(WIN32)
    SET(TCL_INCLUDE_DIRS ${TCL_INCLUDE_DIRS} ${TCL_SRC_PREFIX}/unix)
  ENDIF(WIN32)
  # Locate the TCL libraries.  We allow these to be overridden so that a parent
  # build can use a CMake target instead of an actual library path.
  IF(NOT TCL_LIBRARY)
    find_library(TCL_LIBRARY tcl PATHS ${TCL_BIN_PREFIX}/lib)
  ENDIF(NOT TCL_LIBRARY)
  IF(NOT TCL_STUB_LIBRARY)
    find_library(TCL_STUB_LIBRARY tclstub PATHS ${TCL_BIN_PREFIX}/lib)
  ENDIF(NOT TCL_STUB_LIBRARY)
  IF(NOT TCL_LIBRARY OR NOT TCL_STUB_LIBRARY)
    MESSAGE(FATAL_ERROR "Could not find Tcl libraries in ${TCL_BIN_PREFIX} - please specify the parent directory for the libraries in the TCL_BIN_PREFIX variable (e.g. \${TCL_BIN_PREFIX}/lib)")
  ENDIF(NOT TCL_LIBRARY OR NOT TCL_STUB_LIBRARY)
ELSE(TCL_SRC_PREFIX and TCL_BIN_PREFIX)
  MESSAGE(FATAL_ERROR "Please specify the source and library locations of Tcl using the TCL_SRC_PREFIX and TCL_BIN_PREFIX variables.")
ENDIF(TCL_SRC_PREFIX AND TCL_BIN_PREFIX)
MARK_AS_ADVANCED(TCL_LIBRARY)
MARK_AS_ADVANCED(TCL_STUB_LIBRARY)

# build shared libs by default
set(BUILD_SHARED_LIBS ON)

# build static libs by default
set(BUILD_STATIC_LIBS ON)

# version numbers
SET(TK_VERSION_MAJOR 8)
SET(TK_VERSION_MINOR 5)
SET(TK_VERSION_PATCH 19)
SET(TK_PATCH_LEVEL ${TK_VERSION_MAJOR}.${TK_VERSION_MINOR}.${TK_VERSION_PATCH})

SET(TK_VERSION "${TK_VERSION_MAJOR}.${TK_VERSION_MINOR}")
SET(TK_WIN_VERSION "${TK_VERSION_MAJOR}.${TK_VERSION_MINOR}.${TK_VERSION_PATCH}")
SET(MACHINE "${CMAKE_SYSTEM_PROCESSOR}")

# For Windows, we need the Resource Compiler language
IF(WIN32)
  ENABLE_LANGUAGE(RC)
ENDIF(WIN32)

#---------------------------------------------------------------------
# The following logic is what allows binaries to run successfully in
# the build directory AND install directory.

if(EXISTS "${CMAKE_CURRENT_SOURCE_DIR}/CMake/RPath_Setup.cmake")
  include("${CMAKE_CURRENT_SOURCE_DIR}/CMake/RPath_Setup.cmake")
  if(COMMAND cmake_set_rpath)
    cmake_set_rpath()
  endif(COMMAND cmake_set_rpath)
else(EXISTS "${CMAKE_CURRENT_SOURCE_DIR}/CMake/RPath_Setup.cmake")
  if(NOT HAVE_PARENT_BUILD)
    message(FATAL_ERROR "Not building as a subbuild, but CMake/RPath_Setup.cmake not found.  To build as a stand-alone build, add RPath_Setup.cmake to the CMake directory.")
  endif(NOT HAVE_PARENT_BUILD)
endif(EXISTS "${CMAKE_CURRENT_SOURCE_DIR}/CMake/RPath_Setup.cmake")

#----------------------------------------------------------------------------
# The location in which to install Tcl.  Only do this if CMAKE_INSTALL_PREFIX 
# hasn't been set already, to try and allow parent builds (if any) some control.
IF(CMAKE_INSTALL_PREFIX_INITIALIZED_TO_DEFAULT)
  IF(NOT WIN32)
    IF ("${CMAKE_BUILD_TYPE}" MATCHES "Debug")
      SET(CMAKE_INSTALL_PREFIX "/usr/local/tcldev-${TCL_VERSION}")
    ELSE ("${CMAKE_BUILD_TYPE}" MATCHES "Debug")
      SET(CMAKE_INSTALL_PREFIX "/usr")
    ENDIF ("${CMAKE_BUILD_TYPE}" MATCHES "Debug")
  ENDIF(NOT WIN32)
  SET(CMAKE_INSTALL_PREFIX ${CMAKE_INSTALL_PREFIX} CACHE STRING "TCL install prefix" FORCE)
  SET(CMAKE_INSTALL_PREFIX_INITIALIZED_TO_DEFAULT 0)
ENDIF(CMAKE_INSTALL_PREFIX_INITIALIZED_TO_DEFAULT)

#-----------------------------------------------------------------------------
# Output directories.

IF(NOT CMAKE_LIBRARY_OUTPUT_DIRECTORY)
  SET(CMAKE_LIBRARY_OUTPUT_DIRECTORY ${CMAKE_BINARY_DIR}/lib CACHE INTERNAL "Single output directory for building all libraries.")
ENDIF(NOT CMAKE_LIBRARY_OUTPUT_DIRECTORY)
IF(NOT CMAKE_ARCHIVE_OUTPUT_DIRECTORY)
  SET(CMAKE_ARCHIVE_OUTPUT_DIRECTORY ${CMAKE_BINARY_DIR}/lib CACHE INTERNAL "Single output directory for building all archives.")
ENDIF(NOT CMAKE_ARCHIVE_OUTPUT_DIRECTORY)
IF(NOT CMAKE_RUNTIME_OUTPUT_DIRECTORY)
  SET(CMAKE_RUNTIME_OUTPUT_DIRECTORY ${CMAKE_BINARY_DIR}/bin CACHE INTERNAL "Single output directory for building all executables.")
ENDIF(NOT CMAKE_RUNTIME_OUTPUT_DIRECTORY)

#-----------------------------------------------------------------------------
# Set CMake module path
SET(CMAKE_MODULE_PATH "${CMAKE_MODULE_PATH};${TK_SOURCE_DIR}/CMake")

#-----------------------------------------------------------------------------
# Check if the compiler supports pipe - if so, use it
INCLUDE(CheckCCompilerFlag)
CHECK_C_COMPILER_FLAG(-pipe PIPE_COMPILER_FLAG)
IF(PIPE_COMPILER_FLAG)
  SET(CMAKE_C_FLAGS "${CMAKE_C_FLAGS} -pipe")
ENDIF(PIPE_COMPILER_FLAG)

#-----------------------------------------------------------------------------
# Tcl/Tk's normal build system uses autotools macros, referred to as the
# TEA system.  An attempt to duplicate the required functionality from
# TEA is found in tcl.cmake
INCLUDE(${TK_SOURCE_DIR}/CMake/tcl.cmake)

#-----------------------------------------------------------------------------
# Our replacement for SC_CONFIG_MANPAGES
if(NOT TK_DISABLE_MANPAGES)
  ADD_SUBDIRECTORY(doc)
endif(NOT TK_DISABLE_MANPAGES)

#----------------------------------------------------------------------------
# First, get some standard options out of the way - things that are constant
# between various platforms or pertain to specific OS definitions
add_definitions(-DPACKAGE_NAME="tk")
add_definitions(-DPACKAGE_TARNAME="tk")
add_definitions(-DPACKAGE_VERSION="${TK_VERSION_MAJOR}.${TK_VERSION_MINOR}")
add_definitions(-DPACKAGE_STRING="tk ${TK_VERSION_MAJOR}.${TK_VERSION_MINOR}")
add_definitions(-DPACKAGE_BUGREPORT="")
add_definitions(-DSTDC_HEADERS=1)

# We're building Tk
add_definitions(-DBUILD_tk)

# First, get some standard settings out of the way
# Assume we have STDC_HEADERS until we meet a situation where we don't
ADD_TCL_CFLAG(STDC_HEADERS)
# Get the SHLIB extension from CMake
add_definitions(-DTCL_SHLIB_EXT="${CMAKE_SHARED_LIBRARY_SUFFIX}")

#--------------------------------------------------------------------
# Supply substitutes for missing POSIX header files.
#--------------------------------------------------------------------
SC_MISSING_POSIX_HEADERS()

#----------------------------------------------------------------------------
# Define some platform specific flags

IF(MSVC)
#  add_definitions(-D_CRT_SECURE_NO_DEPRECATE -D_CRT_NONSTDC_NO_DEPRECATE -Ot -Oi -fp:strict -Gs -GS -GL -MD -DBUILD_tk -DBUILD_ttk -DSUPPORT_CONFIG_EMBEDDED -DHAVE_UXTHEME_H=1)
  add_definitions(-D_CRT_SECURE_NO_DEPRECATE -D_CRT_NONSTDC_NO_DEPRECATE -Gs -GS -GL -MD -DBUILD_tk -DBUILD_ttk -DSUPPORT_CONFIG_EMBEDDED -DHAVE_UXTHEME_H=1)
ENDIF(MSVC)

IF(MINGW)
  add_definitions(-D_CRT_SECURE_NO_DEPRECATE -D_CRT_NONSTDC_NO_DEPRECATE -DBUILD_tk -DBUILD_ttk -DSUPPORT_CONFIG_EMBEDDED -DHAVE_UXTHEME_H=1)
  remove_definitions(-w)
  add_definitions(-DHAVE_NO_SEH -DEXCEPTION_DISPOSITION=int)
ENDIF(MINGW)

IF(APPLE)
  add_definitions(-DMAC_OSX_TCL=1)
  INCLUDE(CMakeFindFrameworks)
  CMAKE_FIND_FRAMEWORKS(CoreFoundation)
  IF(CoreFoundation_FRAMEWORKS)
    SET(CoreFoundation_LIBRARIES "-framework CoreFoundation")
    add_definitions(-DHAVE_COREFOUNDATION=1)	 
  ENDIF(CoreFoundation_FRAMEWORKS)
  CMAKE_FIND_FRAMEWORKS(IOKit)
  IF(IOKit_FRAMEWORKS)
    SET(IOKit_LIBRARIES "-framework IOKit")
    add_definitions(-DHAVE_COREFOUNDATION=1)	 
  ENDIF(IOKit_FRAMEWORKS)
ENDIF(APPLE)
#----------------------------------------------------------------------------

#------------------------------------------------------------------------
# Threads support
#------------------------------------------------------------------------
SC_ENABLE_THREADS()

#------------------------------------------------------------------------
# Call the LINK_LIBS macro to get M_LIBRARY properly defined (TEA
# doesn't do this, needs looking into...)
#------------------------------------------------------------------------
SC_TCL_LINK_LIBS()

#----------------------------------------------------------------------------
# 64-bit support - unlike TEA, we enable by default if the platform looks
# like it's 64-bit.  Can be overridden by specifying CMAKE_WORD_SIZE=32BIT
#----------------------------------------------------------------------------
SC_TCL_64BIT_FLAGS()

#--------------------------------------------------------------------
#       Check endianness because we can optimize some operations
#--------------------------------------------------------------------
IF(NOT WIN32)
  INCLUDE(TestBigEndian)
  TEST_BIG_ENDIAN(WORDS_BIGENDIAN)
  IF(WORDS_BIGENDIAN)
    add_definitions(-DWORDS_BIGENDIAN=1)
  ENDIF(WORDS_BIGENDIAN)
ENDIF(NOT WIN32)

#------------------------------------------------------------------------------
#       Find out all about time handling differences.
#------------------------------------------------------------------------------
SC_TIME_HANDLER()

#TODO SC_BUGGY_STRTOD - Solaris 2.4 only?  Do we still care?

#--------------------------------------------------------------------
#       Check for various typedefs and provide substitutes if
#       they don't exist.
#--------------------------------------------------------------------
TCL_CHECK_TYPE_SIZE(mode_t HAVE_MODE)
TCL_CHECK_TYPE_SIZE(pid_t HAVE_PID)
TCL_CHECK_TYPE_SIZE(size_t HAVE_SIZE)
TCL_CHECK_TYPE_SIZE(uid_t HAVE_UID)
# The following tests may need to be more elaborate to function properly
TCL_CHECK_TYPE_SIZE(socklen_t HAVE_SOCKLEN sys/types.h sys/socket.h)
TCL_CHECK_TYPE_SIZE(intptr_t HAVE_INTPTR)
TCL_CHECK_TYPE_SIZE(uintptr_t HAVE_UINTPTR)

TCL_CHECK_STRUCT_HAS_MEMBER("struct passwd" pw_gecos pwd.h HAVE_PW_GECOS)

IF(NOT HAVE_OPENDIR)
  ADD_TCL_CFLAG(USE_DIRENT2_H)
ENDIF(NOT HAVE_OPENDIR)

# The union wait test was for HP-UX - do we still need this?
SET(UNION_WAIT_SRC "
#include <sys/types.h>
#include <sys/wait.h>
int
main ()
{

union wait x;
WIFEXITED(x);           /* Generates compiler error if WIFEXITED
                         * uses an int. */
return 0;
}
")
if(NOT DEFINED HAVE_UNION_WAIT)
  CHECK_C_SOURCE_RUNS("${UNION_WAIT_SRC}" HAVE_UNION_WAIT)
endif(NOT DEFINED HAVE_UNION_WAIT)
IF(NOT HAVE_UNION_WAIT)
  ADD_TCL_CFLAG(NO_UNION_WAIT)
ENDIF(NOT HAVE_UNION_WAIT)

#--------------------------------------------------------------------
#       On Mac OS X, we can build either with X11 or with Aqua
#       For the moment, we don't allow enabling X11 on Windows,
#       eventually we may want to change that.
#       We support enabling only the new Cocoa backend, not the 
#       older Carbon code
#--------------------------------------------------------------------
IF(NOT WIN32)
  IF(APPLE)
    OPTION(TK_ENABLE_AQUA "Build Tk using the Aqua windowing system" OFF)
    IF(TK_ENABLE_AQUA)
      SET(TK_SYSTEM_GRAPHICS aqua)
      add_definitions(-DMAC_OSX_TK=1)
      add_definitions(-D_DARWIN_C_SOURCE=1)
      SET(X11_INCLUDE_DIR "${CMAKE_CURRENT_SOURCE_DIR}/xlib")
      find_library(COCOA_FRAMEWORK Cocoa)	
      MARK_AS_ADVANCED(COCOA_FRAMEWORK)
      find_library(CARBON_FRAMEWORK Carbon)
      MARK_AS_ADVANCED(CARBON_FRAMEWORK)
      SET(OPENGL_USE_AQUA ON CACHE BOOL "Building Aqua Tk" FORCE)
    ELSE(TK_ENABLE_AQUA)
      find_package(X11)
      SET(TK_SYSTEM_GRAPHICS x11)
      SET(OPENGL_USE_AQUA OFF CACHE BOOL "Building X11 Tk" FORCE)
    ENDIF(TK_ENABLE_AQUA)
  ELSE(APPLE)
    find_package(X11)
    SET(TK_SYSTEM_GRAPHICS x11)
  ENDIF(APPLE)
ELSE(NOT WIN32)
  SET(TK_SYSTEM_GRAPHICS win32)
ENDIF(NOT WIN32)

#--------------------------------------------------------------------
# Check whether the header and library for the XScreenSaver
# extension are available, and set HAVE_XSS if so.
# XScreenSaver is needed for Tk_GetUserInactiveTime().
#--------------------------------------------------------------------
INCLUDE(CMakeDependentOption)
CMAKE_DEPENDENT_OPTION(TK_ENABLE_XSS "Use XScreenSaver" ON "TK_SYSTEM_GRAPHICS STREQUAL x11;${X11_Xscreensaver_LIB}" OFF)
# FIXME: assumes if it's enabled that we have it
if (TK_ENABLE_XSS)
  add_definitions(-DHAVE_XSS=1)
endif (TK_ENABLE_XSS)

#--------------------------------------------------------------------
#       Check for freetype / fontconfig / Xft support.
#--------------------------------------------------------------------
if(NOT TK_DISABLE_XFT)
  IF(NOT WIN32)
    find_package(Freetype)
    find_package(Fontconfig)
    IF(NOT FREETYPE_FOUND)
      SET(FREETYPE_LIBRARIES "")
    ENDIF(NOT FREETYPE_FOUND)
    IF(NOT FONTCONFIG_FOUND)
      SET(FONTCONFIG_LIBRARIES "")
    ENDIF(NOT FONTCONFIG_FOUND)
  ENDIF(NOT WIN32)
  CMAKE_DEPENDENT_OPTION(TK_ENABLE_XFT "Use freetype/fontconfig/xft" ON "TK_SYSTEM_GRAPHICS STREQUAL x11;FREETYPE_FOUND;FONTCONFIG_FOUND;X11_Xft_FOUND" OFF)
else(NOT TK_DISABLE_XFT)
  set(TK_ENABLE_XFT OFF)
  set(FREETYPE_LIBRARIES "")
  set(FONTCONFIG_LIBRARIES "")
endif(NOT TK_DISABLE_XFT)


#--------------------------------------------------------------------
#	Check for XkbKeycodeToKeysym.
#--------------------------------------------------------------------

if (NOT TK_ENABLE_AQUA)
  SET(CMAKE_REQUIRED_INCLUDES ${X11_INCLUDE_DIR})
  SET(CMAKE_REQUIRED_LIBRARIES ${X11_X11_LIB})
  CHECK_INCLUDE_FILE(X11/XKBlib.h HAVE_X11_XKBLIB_H)
  if (HAVE_X11_XKBLIB_H)
    CHECK_C_SOURCE_COMPILES("#include <X11/XKBlib.h>\nint main() { void *ptr = (void*)&XkbKeycodeToKeysym; return 0;}" HAVE_XKBKEYCODETOKEYSYM)
    if (HAVE_XKBKEYCODETOKEYSYM)
      add_definitions(-DHAVE_XKBKEYCODETOKEYSYM=1)
    endif (HAVE_XKBKEYCODETOKEYSYM)
  endif (HAVE_X11_XKBLIB_H)
  SET(CMAKE_REQUIRED_INCLUDES "")
  SET(CMAKE_REQUIRED_LIBRARIES "")
endif (NOT TK_ENABLE_AQUA)

#----------------------------------------------------------------------------
# signed/unsigned char - does the Tcl code still need this? See
# http://lists.gnu.org/archive/html/autoconf/2008-06/msg00054.html
#----------------------------------------------------------------------------
SET(unsigned_char_srcs "
int
main ()
{
   static int test_array [1 - 2 * !(((char) -1) < 0)];
   test_array [0] = 0;
   return 0;
}
")
if(NOT DEFINED HAVE_UNION_WAIT)
  CHECK_C_SOURCE_RUNS("${unsigned_char_srcs}" CHAR_IS_UNSIGNED)
endif(NOT DEFINED HAVE_UNION_WAIT)
IF(CHAR_IS_UNSIGNED)
  ADD_TCL_CFLAG(__CHAR_UNSIGNED__)
ENDIF(CHAR_IS_UNSIGNED)
SET(signed_char_srcs "
int
main ()
{
   signed char *p;
   p = 0;
   return 0;
}
")
CHECK_C_SOURCE_COMPILES("${signed_char_srcs}" HAVE_SIGNED_CHAR)
IF(HAVE_SIGNED_CHAR)
  ADD_TCL_CFLAG(HAVE_SIGNED_CHAR)
ENDIF(HAVE_SIGNED_CHAR)



TCL_CHECK_INCLUDE_FILE(sys/types.h HAVE_SYS_TYPES_H)
TCL_CHECK_INCLUDE_FILE(sys/stat.h HAVE_SYS_STAT_H)
TCL_CHECK_INCLUDE_FILE(sys/fstatfs.h HAVE_SYS_FSTATFS_H)
IF(NOT HAVE_SYS_FSTATFS_H)
  add_definitions(-DNO_FSTATFS=1)
ENDIF(NOT HAVE_SYS_FSTATFS_H)
TCL_CHECK_INCLUDE_FILE(memory.h HAVE_MEMORY_H)
TCL_CHECK_INCLUDE_FILE(strings.h HAVE_STRINGS_H)
TCL_CHECK_INCLUDE_FILE(inttypes.h HAVE_INTTYPES_H)
TCL_CHECK_INCLUDE_FILE(stdint.h HAVE_STDINT_H)
TCL_CHECK_INCLUDE_FILE(unistd.h HAVE_UNISTD_H)
TCL_CHECK_INCLUDE_FILE(AvailabilityMacros.h HAVE_AVAILABILITYMACROS_H)
IF(HAVE_AVAILABILITYMACROS_H)
  add_definitions(-DHAVE_AVAILABILITYMACROS_H=1)
ENDIF(HAVE_AVAILABILITYMACROS_H)

CHECK_COMPILER_SUPPORTS_HIDDEN_D()
IF(COMPILER_SUPPORTS_HIDDEN AND NOT MSVC)
  add_definitions(-DMODULE_SCOPE=extern\ __attribute__\(\(__visibility__\("hidden"\)\)\))
ENDIF(COMPILER_SUPPORTS_HIDDEN AND NOT MSVC)

IF(${CMAKE_SYSTEM_NAME} MATCHES "^Darwin$")
  TCL_CHECK_FUNCTION_EXISTS(getattrlist HAVE_GETATTRLIST)
  TCL_CHECK_INCLUDE_FILE(copyfile.h HAVE_COPYFILE_H)
  TCL_CHECK_FUNCTION_EXISTS(copyfile HAVE_COPYFILE)
  TCL_CHECK_INCLUDE_FILE(libkern/OSAtomic.h HAVE_LIBKERN_OSATOMIC_H)
  TCL_CHECK_FUNCTION_EXISTS(OSSpinLockLock HAVE_OSSPINLOCKLOCK)
  ADD_TCL_CFLAG(USE_VFORK)
  add_definitions(-DTCL_DEFAULT_ENCODING="utf-8")
  ADD_TCL_CFLAG(TCL_LOAD_FROM_MEMORY)
  ADD_TCL_CFLAG(TCL_WIDE_CLICKS)
  TCL_CHECK_INCLUDE_FILE_USABILITY(AvailabilityMacros.h HAVE_AVAILABILITYMACROS_H)
  IF(HAVE_AVAILABILITYMACROS_H)
    SET(WEAK_IMPORT_SRCS "
#ifdef __ENVIRONMENT_MAC_OS_X_VERSION_MIN_REQUIRED__
#if __ENVIRONMENT_MAC_OS_X_VERSION_MIN_REQUIRED__ < 1020
#error __ENVIRONMENT_MAC_OS_X_VERSION_MIN_REQUIRED__ < 1020
#endif
#elif MAC_OS_X_VERSION_MIN_REQUIRED < 1020
#error MAC_OS_X_VERSION_MIN_REQUIRED < 1020
#endif
int rand(void) __attribute__((weak_import));
int main() {
rand();
return 0;
}
		")
    CHECK_C_SOURCE_COMPILES("${WEAK_IMPORT_SRCS}" WEAK_IMPORT_WORKING)
    IF(WEAK_IMPORT_WORKING)
      ADD_TCL_CFLAG(HAVE_WEAK_IMPORT)
    ENDIF(WEAK_IMPORT_WORKING)
    SET(SUSV3_SRCS "
#ifdef __ENVIRONMENT_MAC_OS_X_VERSION_MIN_REQUIRED__
#if __ENVIRONMENT_MAC_OS_X_VERSION_MIN_REQUIRED__ < 1050
#error __ENVIRONMENT_MAC_OS_X_VERSION_MIN_REQUIRED__ < 1050
#endif
#elif MAC_OS_X_VERSION_MIN_REQUIRED < 1050
#error MAC_OS_X_VERSION_MIN_REQUIRED < 1050
#endif
#define _DARWIN_C_SOURCE 1
#include <sys/cdefs.h>

int main ()	{return 0;}
		")
    CHECK_C_SOURCE_COMPILES("${SUSV3_SRCS}" SUSV3_WORKING)
    IF(SUSV3_WORKING)
      ADD_TCL_CFLAG(_DARWIN_C_SOURCE)
    ENDIF(SUSV3_WORKING)

  ENDIF(HAVE_AVAILABILITYMACROS_H)
ENDIF(${CMAKE_SYSTEM_NAME} MATCHES "^Darwin$")

add_definitions(-DTK_LAYOUT_WITH_BASE_CHUNKS=0)
add_definitions(-DTK_DRAW_IN_CONTEXT=0)
add_definitions(-DTILE_07_COMPAT=0)
add_definitions(-DDEBUG_FONTSEL=0)

# from unix/Makefile.in:
# Tk does not used deprecated Tcl constructs so it should
# compile fine with -DTCL_NO_DEPRECATED. To remove its own
# set of deprecated code uncomment the second line.
add_definitions(-DTCL_NO_DEPRECATED)


#----------------------------------------------------------------------------
# Add options for optimized and debug compile options - replaces 
# SC_ENABLE_SYMBOLS
#----------------------------------------------------------------------------
IF(TCL_OPTIMIZED)
  ADD_TCL_CFLAG(TCL_CFG_OPTIMIZED)
  SET(CMAKE_C_FLAGS "${CMAKE_C_FLAGS} -DNDEBUG=1 -O2")
ENDIF(TCL_OPTIMIZED)


#----------------------------------------------------------------------------
# GET TO WORK COMPILING
#----------------------------------------------------------------------------

ADD_SUBDIRECTORY(library)

SET(TK_GENERIC_SRCS
  generic/tk3d.c
  generic/tkArgv.c
  generic/tkAtom.c
  generic/tkBind.c
  generic/tkBitmap.c
  generic/tkButton.c
  generic/tkCanvArc.c
  generic/tkCanvBmap.c
  generic/tkCanvImg.c
  generic/tkCanvLine.c
  generic/tkCanvPoly.c
  generic/tkCanvPs.c
  generic/tkCanvText.c
  generic/tkCanvUtil.c
  generic/tkCanvWind.c
  generic/tkCanvas.c
  generic/tkClipboard.c
  generic/tkCmds.c
  generic/tkColor.c
  generic/tkConfig.c
  generic/tkConsole.c
  generic/tkCursor.c
  generic/tkEntry.c
  generic/tkError.c
  generic/tkEvent.c
  generic/tkFocus.c
  generic/tkFont.c
  generic/tkFrame.c
  generic/tkGC.c
  generic/tkGeometry.c
  generic/tkGet.c
  generic/tkGrab.c
  generic/tkGrid.c
  generic/tkImage.c
  generic/tkImgBmap.c
  generic/tkImgGIF.c
  generic/tkImgPPM.c
  generic/tkImgPhoto.c
  generic/tkListbox.c
  generic/tkMain.c
  generic/tkMenu.c
  generic/tkMenuDraw.c
  generic/tkMenubutton.c
  generic/tkMessage.c
  generic/tkObj.c
  generic/tkOldConfig.c
  generic/tkOption.c
  generic/tkPack.c
  generic/tkPanedWindow.c
  generic/tkPlace.c
  generic/tkRectOval.c
  generic/tkScale.c
  generic/tkScrollbar.c
  generic/tkSelect.c
  generic/tkStubInit.c
  generic/tkStyle.c
  generic/tkText.c
  generic/tkTextBTree.c
  generic/tkTextDisp.c
  generic/tkTextImage.c
  generic/tkTextIndex.c
  generic/tkTextMark.c
  generic/tkTextTag.c
  generic/tkTextWind.c
  generic/tkTrig.c
  generic/tkUndo.c
  generic/tkUtil.c
  generic/tkVisual.c
  generic/tkWindow.c
  )

SET(TK_HDRS
  generic/tk.h
  generic/tkDecls.h
  generic/tkPlatDecls.h
  )

SET(TTK_SRCS
  generic/ttk/ttkBlink.c
  generic/ttk/ttkButton.c
  generic/ttk/ttkCache.c
  generic/ttk/ttkClamTheme.c
  generic/ttk/ttkClassicTheme.c
  generic/ttk/ttkDefaultTheme.c
  generic/ttk/ttkElements.c
  generic/ttk/ttkEntry.c
  generic/ttk/ttkFrame.c
  generic/ttk/ttkImage.c
  generic/ttk/ttkInit.c
  generic/ttk/ttkLabel.c
  generic/ttk/ttkLayout.c
  generic/ttk/ttkManager.c
  generic/ttk/ttkNotebook.c
  generic/ttk/ttkPanedwindow.c
  generic/ttk/ttkProgress.c
  generic/ttk/ttkScale.c
  generic/ttk/ttkScrollbar.c
  generic/ttk/ttkScroll.c
  generic/ttk/ttkSeparator.c
  generic/ttk/ttkSquare.c
  generic/ttk/ttkState.c
  generic/ttk/ttkStubInit.c
  generic/ttk/ttkStubLib.c
  generic/ttk/ttkTagSet.c
  generic/ttk/ttkTheme.c
  generic/ttk/ttkTrace.c
  generic/ttk/ttkTrack.c
  generic/ttk/ttkTreeview.c
  generic/ttk/ttkWidget.c
  )

SET(TK_STUB_SRCS
  generic/tkStubInit.c
  generic/tkStubLib.c
  )

SET(TTK_STUB_SRCS
  generic/ttk/ttkStubInit.c
  generic/ttk/ttkStubLib.c
  )

SET(TK_X11_SRCS
  unix/tkUnix.c
  unix/tkUnix3d.c
  unix/tkUnixButton.c
  unix/tkUnixColor.c
  unix/tkUnixConfig.c
  unix/tkUnixCursor.c
  unix/tkUnixDraw.c
  unix/tkUnixEmbed.c
  unix/tkUnixEvent.c
  unix/tkUnixFocus.c
  unix/tkUnixInit.c
  unix/tkUnixKey.c
  unix/tkUnixMenu.c
  unix/tkUnixMenubu.c
  unix/tkUnixScale.c
  unix/tkUnixScrlbr.c
  unix/tkUnixSelect.c
  unix/tkUnixSend.c
  unix/tkUnixWm.c
  unix/tkUnixXId.c
  )

IF(TK_ENABLE_XFT)
  SET(TK_X11_SRCS ${TK_X11_SRCS} unix/tkUnixRFont.c)
ELSE(TK_ENABLE_XFT)
  SET(TK_X11_SRCS ${TK_X11_SRCS} unix/tkUnixFont.c)
ENDIF(TK_ENABLE_XFT)

SET(TK_AQUA_SRCS
  generic/tkFileFilter.c
  generic/tkMacWinMenu.c
  generic/tkPointer.c
  macosx/tkMacOSXBitmap.c
  macosx/tkMacOSXButton.c
  macosx/tkMacOSXCarbonEvents.c
  macosx/tkMacOSXClipboard.c
  macosx/tkMacOSXColor.c
  macosx/tkMacOSXConfig.c
  macosx/tkMacOSXCursor.c
  macosx/tkMacOSXDebug.c
  macosx/tkMacOSXDialog.c
  macosx/tkMacOSXDraw.c
  macosx/tkMacOSXEmbed.c
  macosx/tkMacOSXEntry.c
  macosx/tkMacOSXEvent.c
  macosx/tkMacOSXFont.c
  macosx/tkMacOSXHLEvents.c
  macosx/tkMacOSXInit.c
  macosx/tkMacOSXKeyEvent.c
  macosx/tkMacOSXKeyboard.c
  macosx/tkMacOSXMenu.c
  macosx/tkMacOSXMenubutton.c
  macosx/tkMacOSXMenus.c
  macosx/tkMacOSXMouseEvent.c
  macosx/tkMacOSXNotify.c
  macosx/tkMacOSXRegion.c
  macosx/tkMacOSXScale.c
  macosx/tkMacOSXScrlbr.c
  macosx/tkMacOSXSend.c
  macosx/tkMacOSXSubwindows.c
  macosx/tkMacOSXTest.c
  macosx/tkMacOSXWindowEvent.c
  macosx/tkMacOSXWm.c
  macosx/tkMacOSXXStubs.c
  macosx/ttkMacOSXTheme.c
  unix/tkUnix3d.c
  unix/tkUnixScale.c
  xlib/xcolors.c
  xlib/xdraw.c
  xlib/xgc.c
  xlib/ximage.c
  xlib/xutil.c
  )

configure_file(win/wish.exe.manifest.in ${CMAKE_CURRENT_BINARY_DIR}/wish.exe.manifest @ONLY)
SET(TK_WIN_SRCS
  xlib/xcolors.c
  xlib/xdraw.c
  xlib/xgc.c
  xlib/ximage.c
  xlib/xutil.c
  win/stubs.c
  win/rc/tk.rc
  generic/tkFileFilter.c
  generic/tkMacWinMenu.c
  generic/tkPointer.c
  generic/tkImgUtil.c
  unix/tkUnixMenubu.c
  unix/tkUnixScale.c
  win/tkWin32Dll.c
  win/tkWin3d.c
  win/tkWinButton.c
  win/tkWinClipboard.c
  win/tkWinColor.c
  win/tkWinConfig.c
  win/tkWinCursor.c
  win/tkWinDialog.c
  win/tkWinDraw.c
  win/tkWinEmbed.c
  win/tkWinFont.c
  win/tkWinImage.c
  win/tkWinInit.c
  win/tkWinKey.c
  win/tkWinMenu.c
  win/tkWinPixmap.c
  win/tkWinPointer.c
  win/tkWinRegion.c
  win/tkWinScrlbr.c
  win/tkWinSend.c
  win/tkWinSendCom.c
  win/tkWinTest.c
  win/tkWinWindow.c
  win/tkWinWm.c
  win/tkWinX.c
  win/ttkWinMonitor.c
  win/ttkWinTheme.c
  win/ttkWinXPTheme.c
  )

SET(TK_SRCS ${TK_GENERIC_SRCS} ${TTK_SRCS})

IF(WIN32)
  SET(TK_SRCS ${TK_SRCS} ${TK_WIN_SRCS})
ELSE(WIN32)
  IF(TK_ENABLE_AQUA)
    SET(TK_SRCS ${TK_SRCS} ${TK_AQUA_SRCS})
  ELSE(TK_ENABLE_AQUA)
    SET(TK_SRCS ${TK_SRCS} ${TK_X11_SRCS})
  ENDIF(TK_ENABLE_AQUA)
ENDIF(WIN32)

SET(TK_INCLUDE_PATH ${TK_SOURCE_DIR}/generic ${TK_SOURCE_DIR}/bitmaps ${TK_BINARY_DIR}/include)
IF(WIN32)
  SET(TK_INCLUDE_PATH ${TK_INCLUDE_PATH} ${TK_SOURCE_DIR}/win ${TK_SOURCE_DIR}/xlib ${TK_SOURCE_DIR}/xlib/X11)
ELSE(WIN32)
  SET(TK_INCLUDE_PATH ${TK_INCLUDE_PATH} ${TK_SOURCE_DIR}/unix)
  IF(APPLE)
    SET(TK_INCLUDE_PATH ${TK_INCLUDE_PATH} ${TK_SOURCE_DIR}/macosx)
    IF(TK_ENABLE_AQUA)
      SET(TK_INCLUDE_PATH ${TK_INCLUDE_PATH} ${X11_INCLUDE_DIR})
    ENDIF(TK_ENABLE_AQUA)
  ENDIF(APPLE)
ENDIF(WIN32)

IF(NOT WIN32)
  SET(WISH_SRCS unix/tkAppInit.c)
ELSE(NOT WIN32)
  SET(WISH_SRCS win/winMain.c)# win/rc/wish.rc)
ENDIF(NOT WIN32)

IF(X11_INCLUDE_DIR)
  SET(TK_X11_INCLUDE_DIRS ${TK_X11_INCLUDE_DIRS} ${X11_INCLUDE_DIR})
ENDIF(X11_INCLUDE_DIR)
IF(X11_Xft_INCLUDE_PATH)
  SET(TK_X11_INCLUDE_DIRS ${TK_X11_INCLUDE_DIRS} ${X11_Xft_INCLUDE_PATH})
ENDIF(X11_Xft_INCLUDE_PATH)
IF(X11_Xrender_INCLUDE_PATH)
  SET(TK_X11_INCLUDE_DIRS ${TK_X11_INCLUDE_DIRS} ${X11_Xrender_INCLUDE_PATH})
ENDIF(X11_Xrender_INCLUDE_PATH)
IF(X11_Xscreensaver_INCLUDE_PATH)
  SET(TK_X11_INCLUDE_DIRS ${TK_X11_INCLUDE_DIRS} ${X11_Xscreensaver_INCLUDE_PATH})
ENDIF(X11_Xscreensaver_INCLUDE_PATH)

include_directories(
  ${TCL_INCLUDE_DIRS}
  ${TK_INCLUDE_PATH}
  ${TK_X11_INCLUDE_DIRS}
  )
if(TK_ENABLE_XFT)
  include_directories(${FREETYPE_INCLUDE_DIRS})
endif(TK_ENABLE_XFT)

SET(X11_TK_LIBS "")
IF(X11_X11_LIB)
  SET(X11_TK_LIBS ${X11_TK_LIBS} ${X11_X11_LIB})
ENDIF(X11_X11_LIB)
IF(X11_Xext_LIB)
  SET(X11_TK_LIBS ${X11_TK_LIBS} ${X11_Xext_LIB})
ENDIF(X11_Xext_LIB)
if(X11_Xft_LIB AND TK_ENABLE_XFT)
  SET(X11_TK_LIBS ${X11_TK_LIBS} ${X11_Xft_LIB})
endif(X11_Xft_LIB AND TK_ENABLE_XFT)
IF(X11_Xrender_LIB)
  SET(X11_TK_LIBS ${X11_TK_LIBS} ${X11_Xrender_LIB})
ENDIF(X11_Xrender_LIB)
IF(TK_ENABLE_XSS)
  IF(X11_Xscreensaver_LIB)
    SET(X11_TK_LIBS ${X11_TK_LIBS} ${X11_Xscreensaver_LIB})
  ENDIF(X11_Xscreensaver_LIB)
ENDIF(TK_ENABLE_XSS)

IF(NOT LIB_DIR)
  SET(LIB_DIR lib)
ENDIF(NOT LIB_DIR)

IF(NOT CMAKE_LIBRARY_OUTPUT_DIRECTORY)
  SET(CMAKE_LIBRARY_OUTPUT_DIRECTORY ${CMAKE_BINARY_DIR}/${LIB_DIR})
ENDIF(NOT CMAKE_LIBRARY_OUTPUT_DIRECTORY)

add_library(tkstub STATIC ${TK_STUB_SRCS} ${TTK_STUB_SRCS})
CHECK_C_COMPILER_FLAG(-fPIC PIC_COMPILER_FLAG)
IF(PIC_COMPILER_FLAG)
  SET_TARGET_PROPERTIES(tkstub PROPERTIES COMPILE_FLAGS "-fPIC")
ENDIF(PIC_COMPILER_FLAG)
install(TARGETS tkstub 
	RUNTIME DESTINATION ${BIN_DIR}
	LIBRARY DESTINATION ${LIB_DIR}
	ARCHIVE DESTINATION ${LIB_DIR})
SET_PROPERTY(TARGET tkstub APPEND PROPERTY COMPILE_DEFINITIONS USE_TCL_STUBS)

IF(MINGW)
  set(TK_MINGW_LIBS comctl32 m dinput8 dxguid dxerr8 user32 gdi32 winmm imm32 ole32 oleaut32 shell32 version uuid)
ENDIF(MINGW)
add_library(tk ${TK_SRCS})
IF(WIN32)
  target_link_libraries(tk tkstub ${TCL_STUB_LIBRARY} ${TCL_THREADS_LIB} ${X11_TK_LIBS} ${TK_MINGW_LIBS})
ELSE(WIN32)
  target_link_libraries(tk tkstub ${TCL_STUB_LIBRARY} ${TCL_THREADS_LIB}
	  ${X11_TK_LIBS} ${CoreFoundation_LIBRARIES} ${CARBON_FRAMEWORK}
	  ${COCOA_FRAMEWORK} ${IOKit_LIBRARIES} ${FREETYPE_LIBRARIES}
	  ${PNG_LIBRARY} ${FONTCONFIG_LIBRARIES} ${M_LIBRARY})
ENDIF(WIN32)
install(TARGETS tk
	RUNTIME DESTINATION ${BIN_DIR}
	LIBRARY DESTINATION ${LIB_DIR}
	ARCHIVE DESTINATION ${LIB_DIR})
SET_TARGET_PROPERTIES(tk PROPERTIES VERSION ${TK_VERSION} SOVERSION ${TK_VERSION_MAJOR}.${TK_VERSION_MINOR})
SET_PROPERTY(TARGET tk APPEND PROPERTY COMPILE_DEFINITIONS USE_TCL_STUBS)
IF(TK_ENABLE_AQUA)
  SET_PROPERTY(TARGET tk APPEND PROPERTY COMPILE_FLAGS "-std=gnu99 -x objective-c")
ENDIF(TK_ENABLE_AQUA)

TCL_PKGINDEX(tk Tk ${TK_PATCH_LEVEL})

install(FILES ${TK_HDRS} DESTINATION include)

IF(WIN32)
  add_executable(wish WIN32 ${WISH_SRCS})
  target_link_libraries(wish tcl tk imm32.lib comctl32.lib)
ELSE(WIN32)
  add_executable(wish ${WISH_SRCS})
  target_link_libraries(wish ${TCL_LIBRARY} tk ${CoreFoundation_LIBRARIES}
	  ${CARBON_FRAMEWORK} ${COCOA_FRAMEWORK} ${IOKit_LIBRARIES}
	  ${TCL_THREADS_LIB} ${M_LIBRARY} ${FREETYPE_LIBRARIES} ${PNG_LIBRARY} ${FONTCONFIG_LIBRARIES})
  IF(TK_ENABLE_AQUA)
    SET_PROPERTY(TARGET wish APPEND PROPERTY COMPILE_FLAGS "-std=gnu99 -x objective-c")
  ENDIF(TK_ENABLE_AQUA)
ENDIF(WIN32)
SET_TARGET_PROPERTIES(wish PROPERTIES VERSION ${TK_VERSION_MAJOR}.${TK_VERSION_MINOR})
install(TARGETS wish DESTINATION bin)

IF(NOT WIN32)
  FILE(COPY unix/tkAppInit.c DESTINATION ${CMAKE_LIBRARY_OUTPUT_DIRECTORY}/tk${TK_PATCH_LEVEL})
  foreach(CFG_TYPE ${CMAKE_CONFIGURATION_TYPES})
    string(TOUPPER "${CFG_TYPE}" CFG_TYPE_UPPER)
    FILE(COPY unix/tkAppInit.c DESTINATION ${CMAKE_LIBRARY_OUTPUT_DIRECTORY_${CFG_TYPE_UPPER}}/tk${TK_PATCH_LEVEL})
  endforeach(CFG_TYPE ${CMAKE_CONFIGURATION_TYPES})
  INSTALL(FILES unix/tkAppInit.c DESTINATION lib/tk${TK_PATCH_LEVEL})
ENDIF(NOT WIN32)
