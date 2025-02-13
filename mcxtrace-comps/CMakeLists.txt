cmake_minimum_required( VERSION 3.17.0 )

# This will install McXtrace components
project( mcxtrace-comps C )

# Set module path
set(CMAKE_MODULE_PATH ${CMAKE_MODULE_PATH} "${CMAKE_SOURCE_DIR}/cmake/Modules/")

# Set McCode values (from mkdist or defaults)
include(MCUtil)
setupMCCODE("mcxtrace")

set(WORK "${PROJECT_BINARY_DIR}/work")

# CPack configuration
set(CPACK_PACKAGE_NAME          "${FLAVOR}-comps-${MCCODE_VERSION}")
set(CPACK_RESOURCE_FilE_LICENSE "${CMAKE_CURRENT_SOURCE_DIR}/../COPYING")
set(CPACK_PACKAGE_CONTACT       "pkwi@fysik.dtu.dk")

set(CPACK_PACKAGE_VERSION       "1.0")
set(CPACK_PACKAGE_VERSION_MAJOR "1")
set(CPACK_PACKAGE_VERSION_MINOR "0")


# Debian
set(CPACK_DEBIAN_PACKAGE_DEPENDS "${FLAVOR}-${MCCODE_VERSION}, libnexus-dev, libxrl-dev")

# NSIS
set(CPACK_NSIS_PACKAGE_NAME "${MCCODE_STRING} Components")
set(CPACK_NSIS_DISPLAY_NAME "${MCCODE_STRING} Components")

include(CPack)

option( ENABLE_CIF2HKL "Build Third Party code cif2hkl (fortran)" ON )#TODO: Only enable enable Fortran if this is on
if ( ENABLE_CIF2HKL )
  enable_language( Fortran )
  add_executable(
    cif2hkl
    ${CMAKE_CURRENT_SOURCE_DIR}/libs/cif2hkl/cif2hkl.F90
    )
  target_compile_options(cif2hkl PUBLIC "-ffree-line-length-512")
  install (
    PROGRAMS "${PROJECT_BINARY_DIR}/cif2hkl${DOT_EXE_SUFFIX}"
    DESTINATION "${DEST_BINDIR}"
    )
endif()



file_globsrc( tmp "share/*" )
install( FILES ${tmp} DESTINATION "${DEST_DATADIR_CODEFILES}" )

set( tmp_complist "misc" "monitors" "obsolete" "optics" "samples" "sources" "union" "sasmodels" "astrox")
if ( NOT EXCLUDE_CONTRIB )
  list( APPEND tmp_complist "contrib" )
endif()
foreach(name ${tmp_complist})
  file_globsrc( tmp "${name}/*" )
  install( FILES ${tmp} DESTINATION "${DEST_DATADIR_COMPS}/${name}" )
endforeach()

# contrib subfolders / examplified from McStas
#if ( NOT EXCLUDE_CONTRIB )
#  foreach(contrib_subdir doc)
#    file_globsrc( tmp "contrib/${contrib_subdir}/*" )
#    install( FILES ${tmp} DESTINATION "${DEST_DATADIR_COMPS}/contrib/${contrib_subdir}" )
#  endforeach()
#endif()

if ( NOT EXCLUDE_DATA )
  # base data dir
  file_globsrc( tmp "data/*" )
  install( FILES ${tmp} DESTINATION "${DEST_DATADIR_DATAFILES}" )
  # datadir subfolders / examplified from McStas
  # foreach(data_subdir Gas_tables ISIS_tables)
  #  file_globsrc( tmp "data/${data_subdir}/*" )
  #  install( FILES ${tmp} DESTINATION "${DEST_DATADIR_COMPS}/data/${data_subdir}" )
  # endforeach()
endif()

install( DIRECTORY "examples/" DESTINATION "${DEST_DATADIR_EXAMPLES}")

file_globsrc( tmp "editors/*" )
install( FILES ${tmp} DESTINATION "${DEST_DATADIR_EDITORS}" )

# Include mcstas-comp revision tag file
configure_file("${CMAKE_CURRENT_SOURCE_DIR}/revision.in" "${WORK}/revision" @ONLY)

# copy some descriptory files
configure_directory ("NOMENCLATURE" "work/nomenclature")
configure_directory ("README.md" "work/readme")
install( FILES "${WORK}/nomenclature/NOMENCLATURE" DESTINATION "${DEST_DATADIR_COMPS}")
install( FILES "${WORK}/readme/README.md" DESTINATION "${DEST_DATADIR_COMPS}")
install( FILES "${WORK}/revision" DESTINATION "${DEST_DATADIR_COMPS}" )
