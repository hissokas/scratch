# Check http://elementaryos.org/docs/developer-guide/cmake for documentation

cmake_minimum_required (VERSION 2.8)
cmake_policy (VERSION 2.8)
project (io.elementary.code)
enable_testing ()
include (GNUInstallDirs)
set (DATADIR "${CMAKE_INSTALL_FULL_DATAROOTDIR}/${CMAKE_PROJECT_NAME}")
set (SCRIPTDIR "${CMAKE_INSTALL_FULL_DATAROOTDIR}/${CMAKE_PROJECT_NAME}/scripts")
set (PKGDATADIR "${DATADIR}/${CMAKE_PROJECT_NAME}")
set (GETTEXT_PACKAGE ${CMAKE_PROJECT_NAME})
set (RELEASE_NAME "The elementary Text Editor.")
set (EXEC_NAME ${CMAKE_PROJECT_NAME})
set (VERSION "2.4.1")
set (VERSION_INFO "Release")
set (LIBNAME "scratchcore")
set (LIBVERSION "0")
set (SOVERSION 0)
set (PLUGINDIR "${CMAKE_INSTALL_FULL_LIBDIR}/${CMAKE_PROJECT_NAME}/plugins")
set (PREFIX ${CMAKE_INSTALL_PREFIX})
set (DOLLAR "$")
list (APPEND CMAKE_MODULE_PATH ${CMAKE_SOURCE_DIR}/cmake)

# Some configuration
configure_file (${CMAKE_SOURCE_DIR}/src/config.vala.cmake ${CMAKE_SOURCE_DIR}/src/config.vala)
configure_file (${CMAKE_SOURCE_DIR}/src/${LIBNAME}.pc.cmake ${CMAKE_BINARY_DIR}/src/${LIBNAME}.pc)

# Some definitions
add_definitions(-DGETTEXT_PACKAGE=\"${GETTEXT_PACKAGE}\")

# Vala
find_package (Vala REQUIRED)
include (ValaVersion)
ensure_vala_version ("0.22" MINIMUM)
include (ValaPrecompile)

add_definitions(-DVALA_VERSION="${VALA_SHORTVER}")
add_definitions(-w) # Disable gcc warnings

option (USE_ZEITGEIST "Use Zeitgeist integration" ON)

if (USE_ZEITGEIST)
  message ("-- Zeitgeist integration enabled")
  set (ZEITGEIST_DEPS zeitgeist-2.0)
  set (ZEITGEIST_OPTIONS --define=HAVE_ZEITGEIST)
else ()
  message ("-- Zeitgeist integration disabled")
endif ()

# Dependencies
set (SCRATCH_DEPS
  libpeas-1.0
  libpeas-gtk-1.0
  gtksourceview-3.0
  gee-0.8
  gobject-2.0
  glib-2.0
  gio-2.0
  gtk+-3.0
  granite>=0.3.0
  ${ZEITGEIST_DEPS})

find_package (PkgConfig)
pkg_check_modules (GTK+ REQUIRED
  gtk+-3.0>=3.10)

pkg_check_modules (LIBSOURCE REQUIRED
  libpeas-gtk-1.0
  gtksourceview-3.0>=3.10
  gthread-2.0)

pkg_check_modules (DEPS REQUIRED ${SCRATCH_DEPS})

pkg_check_modules (GTKSOURCEVIEW_3_18 QUIET gtksourceview-3.0>=3.18)
if (GTKSOURCEVIEW_3_18_FOUND)
  set(GTKSOURCEVIEW_FLAGS "--define=GTKSOURCEVIEW_3_18")
  message ("-- Found gtksourceview >=3.18 GtkSourceMap available")
endif ()

pkg_check_modules (GRANITE_0_4_2 QUIET granite>=0.4.2)
if (GRANITE_0_4_2_FOUND)
  set(GRANITE_FLAGS "--define=GRANITE_MESSAGEDIALOG")
  message ("-- GraniteMessageDialog available")
endif ()

set(NORMAL_CFLAGS ${DEPS_CFLAGS} ${LIBSOURCE_CFLAGS} ${GCONF_CFLAGS})
set(NORMAL_LINK_DIRS ${DEPS_LIBRARY_DIRS} ${LIBSOURCE_LIBRARY_DIRS} ${GCONF_LIBRARY_DIRS})
set(NORMAL_LIBRARIES ${DEPS_LIBRARIES} ${LIBSOURCE_LIBRARIES} ${GCONF_LIBRARIES})

add_definitions (${DEPS_CFLAGS} ${LIBSOURCE_CFLAGS} ${GCONF_CFLAGS})
link_libraries (${DEPS_LIBRARIES} ${LIBSOURCE_LIBRARIES} ${GCONF_LIBRARIES})
link_directories (${DEPS_LIBRARY_DIRS} ${LIBSOURCE_LIBRARY_DIRS} ${GCONF_LIBRARY_DIRS})


# Some include paths
include_directories(${CMAKE_BINARY_DIR}/${LIBNAME})
include_directories(${CMAKE_BINARY_DIR}/src)

# Subdirectories
add_subdirectory (src)
add_subdirectory (plugins)
add_subdirectory (schemas)
add_subdirectory (po)
add_subdirectory (data)
