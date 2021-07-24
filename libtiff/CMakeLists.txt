# CMake build for libtiff
#
# Copyright © 2015 Open Microscopy Environment / University of Dundee
# Written by Roger Leigh <rleigh@codelibre.net>
#
# Permission to use, copy, modify, distribute, and sell this software and
# its documentation for any purpose is hereby granted without fee, provided
# that (i) the above copyright notices and this permission notice appear in
# all copies of the software and related documentation, and (ii) the names of
# Sam Leffler and Silicon Graphics may not be used in any advertising or
# publicity relating to the software without the specific, prior written
# permission of Sam Leffler and Silicon Graphics.
#
# THE SOFTWARE IS PROVIDED "AS-IS" AND WITHOUT WARRANTY OF ANY KIND,
# EXPRESS, IMPLIED OR OTHERWISE, INCLUDING WITHOUT LIMITATION, ANY
# WARRANTY OF MERCHANTABILITY OR FITNESS FOR A PARTICULAR PURPOSE.
#
# IN NO EVENT SHALL SAM LEFFLER OR SILICON GRAPHICS BE LIABLE FOR
# ANY SPECIAL, INCIDENTAL, INDIRECT OR CONSEQUENTIAL DAMAGES OF ANY KIND,
# OR ANY DAMAGES WHATSOEVER RESULTING FROM LOSS OF USE, DATA OR PROFITS,
# WHETHER OR NOT ADVISED OF THE POSSIBILITY OF DAMAGE, AND ON ANY THEORY OF
# LIABILITY, ARISING OUT OF OR IN CONNECTION WITH THE USE OR PERFORMANCE
# OF THIS SOFTWARE.

# Generate headers
configure_file(${CMAKE_CURRENT_SOURCE_DIR}/tif_config.h.cmake.in
               ${CMAKE_CURRENT_BINARY_DIR}/tif_config.h
               @ONLY)
configure_file(${CMAKE_CURRENT_SOURCE_DIR}/tiffconf.h.cmake.in
               ${CMAKE_CURRENT_BINARY_DIR}/tiffconf.h
               @ONLY)

extra_dist(
  SConstruct
  tif_config.h-vms
  tif_config.vc.h
  tif_config.wince.h
  tiffconf.vc.h
  tiffconf.wince.h
  libtiff.def
  libtiff.map
  libtiffxx.map)

set(tiff_HEADERS
  tiff.h
  tiffio.h
  tiffvers.h)

set(tiff_noinst_HEADERS
  t4.h
  tif_dir.h
  tif_predict.h
  tiffiop.h
  uvcode.h)

set(nodist_tiff_HEADERS
  ${CMAKE_CURRENT_BINARY_DIR}/tiffconf.h)

set(tiff_SOURCES
  tif_aux.c
  tif_close.c
  tif_codec.c
  tif_color.c
  tif_compress.c
  tif_dir.c
  tif_dirinfo.c
  tif_dirread.c
  tif_dirwrite.c
  tif_dumpmode.c
  tif_error.c
  tif_extension.c
  tif_fax3.c
  tif_fax3sm.c
  tif_flush.c
  tif_getimage.c
  tif_jbig.c
  tif_jpeg.c
  tif_jpeg_12.c
  tif_luv.c
  tif_lzma.c
  tif_lzw.c
  tif_next.c
  tif_ojpeg.c
  tif_open.c
  tif_packbits.c
  tif_pixarlog.c
  tif_predict.c
  tif_print.c
  tif_read.c
  tif_strip.c
  tif_swab.c
  tif_thunder.c
  tif_tile.c
  tif_version.c
  tif_warning.c
  tif_webp.c
  tif_write.c
  tif_zip.c
  tif_zstd.c)

set(tiffxx_HEADERS
  tiffio.hxx)

set(tiffxx_SOURCES
  tif_stream.cxx)

if(USE_WIN32_FILEIO)
  extra_dist(tif_unix.c)
  list(APPEND tiff_SOURCES tif_win32.c)
else()
  extra_dist(tif_win32.c)
  list(APPEND tiff_SOURCES tif_unix.c)
endif()

add_library(tiff ${tiff_SOURCES} ${tiff_HEADERS} ${nodist_tiff_HEADERS}
            ${tiff_port_SOURCES} libtiff.def)
target_include_directories(tiff
    PUBLIC
        $<BUILD_INTERFACE:${CMAKE_CURRENT_SOURCE_DIR}>
        $<BUILD_INTERFACE:${CMAKE_CURRENT_BINARY_DIR}>
        ${TIFF_INCLUDES}
)
target_compile_definitions(tiff
  PUBLIC
    TIFF_SIZE_T=${TIFF_SIZE_T}
    TIFF_SIZE_FORMAT="%${TIFF_SIZE_FORMAT}"
    TIFF_SSIZE_T=${TIFF_SSIZE_T}
    TIFF_SSIZE_FORMAT="%${TIFF_SSIZE_FORMAT}"
)
target_link_libraries(tiff ${TIFF_LIBRARY_DEPS})
set_target_properties(tiff PROPERTIES SOVERSION ${SO_COMPATVERSION})
if(NOT CYGWIN)
    # This property causes shared libraries on Linux to have the full version
    # encoded into their final filename.  We disable this on Cygwin because
    # it causes cygz-${TIFF_FULL_VERSION}.dll to be created when cygz.dll
    # seems to be the default.
    set_target_properties(tiff PROPERTIES VERSION ${SO_VERSION})
endif()
if(HAVE_LD_VERSION_SCRIPT)
  set_target_properties(tiff PROPERTIES LINK_FLAGS
                        "-Wl,--version-script=${CMAKE_CURRENT_SOURCE_DIR}/libtiff.map")
endif()

if(HUNTER_ENABLED)
  set(EXPORT_PROJECT_NAME "${PROJECT_NAME}")
  set(TARGETS_EXPORT_NAME "${EXPORT_PROJECT_NAME}Targets")

  install(TARGETS ${EXPORT_PROJECT_NAME}
          EXPORT ${TARGETS_EXPORT_NAME}
          RUNTIME DESTINATION ${CMAKE_INSTALL_BINDIR}
          LIBRARY DESTINATION ${CMAKE_INSTALL_LIBDIR}
          ARCHIVE DESTINATION ${CMAKE_INSTALL_LIBDIR}
          INCLUDES DESTINATION ${CMAKE_INSTALL_INCLUDEDIR})

  install(FILES ${${EXPORT_PROJECT_NAME}_HEADERS} ${nodist_${EXPORT_PROJECT_NAME}_HEADERS}
          DESTINATION "${CMAKE_INSTALL_INCLUDEDIR}")

  set(config_install_dir "${CMAKE_INSTALL_LIBDIR}/cmake/${EXPORT_PROJECT_NAME}")
  set(generated_dir "${CMAKE_CURRENT_BINARY_DIR}/generated")
  set(version_config "${generated_dir}/${EXPORT_PROJECT_NAME}ConfigVersion.cmake")
  set(project_config "${generated_dir}/${EXPORT_PROJECT_NAME}Config.cmake")

  include(CMakePackageConfigHelpers)

  write_basic_package_version_file(
      "${version_config}" COMPATIBILITY SameMajorVersion
  )

  configure_package_config_file(
      "${CMAKE_CURRENT_SOURCE_DIR}/Config.cmake.in"
      "${project_config}"
      INSTALL_DESTINATION "${config_install_dir}"
  )

  install(
      FILES "${project_config}" "${version_config}"
      DESTINATION "${config_install_dir}"
  )

  install(
      EXPORT "${TARGETS_EXPORT_NAME}"
      DESTINATION "${config_install_dir}"
  )
else()
  install(TARGETS tiff
        RUNTIME DESTINATION ${CMAKE_INSTALL_FULL_BINDIR}
        LIBRARY DESTINATION ${CMAKE_INSTALL_FULL_LIBDIR}
        ARCHIVE DESTINATION ${CMAKE_INSTALL_FULL_LIBDIR})

  install(FILES ${tiff_HEADERS} ${nodist_tiff_HEADERS}
        DESTINATION "${CMAKE_INSTALL_FULL_INCLUDEDIR}")
endif()

if(CXX_SUPPORT)
  add_library(tiffxx ${tiffxx_SOURCES} ${tiffxx_HEADERS})
  target_link_libraries(tiffxx tiff)
  target_compile_definitions(tiffxx
    PUBLIC
      TIFF_SIZE_T=${TIFF_SIZE_T}
      TIFF_SIZE_FORMAT="%${TIFF_SIZE_FORMAT}"
      TIFF_SSIZE_T=${TIFF_SSIZE_T}
      TIFF_SSIZE_FORMAT="%${TIFF_SSIZE_FORMAT}"
  )
  set_target_properties(tiffxx PROPERTIES SOVERSION ${SO_COMPATVERSION})
  if(NOT CYGWIN)
    # This property causes shared libraries on Linux to have the full version
    # encoded into their final filename.  We disable this on Cygwin because
    # it causes cygz-${TIFF_FULL_VERSION}.dll to be created when cygz.dll
    # seems to be the default.
    set_target_properties(tiffxx PROPERTIES VERSION ${SO_VERSION})
  endif()
  if(HAVE_LD_VERSION_SCRIPT)
    set_target_properties(tiffxx PROPERTIES LINK_FLAGS
                          "-Wl,--version-script=${CMAKE_CURRENT_SOURCE_DIR}/libtiffxx.map")
  endif()

  if(HUNTER_ENABLED)
    set(EXPORT_PROJECT_NAME "${PROJECT_NAME}xx")
    set(TARGETS_EXPORT_NAME "${EXPORT_PROJECT_NAME}Targets")

    install(TARGETS ${EXPORT_PROJECT_NAME}
            EXPORT ${TARGETS_EXPORT_NAME}
            RUNTIME DESTINATION ${CMAKE_INSTALL_BINDIR}
            LIBRARY DESTINATION ${CMAKE_INSTALL_LIBDIR}
            ARCHIVE DESTINATION ${CMAKE_INSTALL_LIBDIR}
            INCLUDES DESTINATION ${CMAKE_INSTALL_INCLUDEDIR})

    install(FILES ${${EXPORT_PROJECT_NAME}_HEADERS}
            DESTINATION "${CMAKE_INSTALL_INCLUDEDIR}")

    set(cxx_config_install_dir "${CMAKE_INSTALL_LIBDIR}/cmake/${EXPORT_PROJECT_NAME}")
    set(cxx_generated_dir "${CMAKE_CURRENT_BINARY_DIR}/generated")
    set(cxx_version_config "${cxx_generated_dir}/${EXPORT_PROJECT_NAME}ConfigVersion.cmake")
    set(cxx_project_config "${cxx_generated_dir}/${EXPORT_PROJECT_NAME}Config.cmake")

    include(CMakePackageConfigHelpers)

    write_basic_package_version_file(
        "${cxx_version_config}" COMPATIBILITY SameMajorVersion
    )

    configure_package_config_file(
        "${CMAKE_CURRENT_SOURCE_DIR}/Config.cmake.in"
        "${cxx_project_config}"
        INSTALL_DESTINATION "${cxx_config_install_dir}"
    )

    install(
        FILES "${cxx_project_config}" "${cxx_version_config}"
        DESTINATION "${cxx_config_install_dir}"
    )

    install(
        EXPORT "${TARGETS_EXPORT_NAME}"
        DESTINATION "${cxx_config_install_dir}"
    )
  else()
    install(TARGETS tiffxx
            RUNTIME DESTINATION ${CMAKE_INSTALL_FULL_BINDIR}
            LIBRARY DESTINATION ${CMAKE_INSTALL_FULL_LIBDIR}
            ARCHIVE DESTINATION ${CMAKE_INSTALL_FULL_LIBDIR})

    install(FILES ${tiffxx_HEADERS}
            DESTINATION "${CMAKE_INSTALL_FULL_INCLUDEDIR}")
  endif()

endif()
