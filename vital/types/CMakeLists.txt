#
# vital/types library and headers
#
# This sub-library is intended to be home to canonical typedefs and other
# commonly useful type definitions.
#

set( vital_types_public_headers
  geo_polygon.h
  geodesy.h
  polygon.h
  vector.h
  )
set( vital_type_sources
  geo_polygon.cxx
  geodesy.cxx
  polygon.cxx
  )

kwiver_install_headers(
  ${vital_types_public_headers}
  SUBDIR vital/types
)
kwiver_install_headers(  # generated export header file
  ${CMAKE_CURRENT_BINARY_DIR}/vital_types_export.h
  SUBDIR vital/types
  NOPATH
)
kwiver_add_library(
  vital_types
  ${vital_types_public_headers}
  ${vital_type_sources}
  ${CMAKE_CURRENT_BINARY_DIR}/vital_types_export.h
  )

# Depend on file configuration
add_dependencies( vital_types kwiver_configure )

target_link_libraries( vital_types
  PUBLIC    Eigen3::Eigen
            vital_config
  )
