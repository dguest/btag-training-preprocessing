#
# Build configuration for the IOTests package
#

# Set the name of the package:
atlas_subdir( BTagTrainingPreprocessing )

# turn off an annoying set of warnings I get from HDF5
macro( disable_overloaded_warning name )
  string(REPLACE "-Woverloaded-virtual" "" NEW_FLAGS ${${name}})
  set(${name} ${NEW_FLAGS})
endmacro()
disable_overloaded_warning(CMAKE_CXX_FLAGS_DEBUG)
disable_overloaded_warning(CMAKE_CXX_FLAGS_RELEASE)
disable_overloaded_warning(CMAKE_CXX_FLAGS_RELWITHDEBINFO)

# Packages that this package depends on:
atlas_depends_on_subdirs(
   PRIVATE
   Control/xAODRootAccess
   Event/xAOD/xAODEventInfo
   Event/xAOD/xAODCaloCluster
   Event/xAOD/xAODTrack
   Control/AthToolSupport/AsgTools
   Reconstruction/Jet/JetCalibTools
   Reconstruction/Jet/JetSelectorTools
   InnerDetector/InDetRecTools/InDetTrackSelectionTool
   Reconstruction/Jet/JetMomentTools
   Reconstruction/Jet/JetJvtEfficiency )

# External(s) used by the package:
list(APPEND CMAKE_PREFIX_PATH $ENV{ROOTSYS})
find_package(ROOT REQUIRED COMPONENTS RIO Hist Tree Net Core)

# find zlib
find_package( ZLIB REQUIRED )
get_filename_component(ZLIB_LIB_DIR ZLIB_LIBRARIES DIRECTORY)

# Grab HDF5 from AnalysisBaseExternals.  The way we do it is ugly
# until atlas externals is fixed, see here:
#
# https://gitlab.cern.ch/atlas/atlasexternals/merge_requests/209
#
set( H5_LOC $ENV{AnalysisBaseExternals_DIR} )
set( HDF5_INCLUDE_DIRS "${H5_LOC}/include/")
set( HDF5_LIB "${H5_LOC}/lib/")
set( SUFFFIX -shared${CMAKE_SHARED_LIBRARY_SUFFIX})
set( HDF5_LIBRARIES
  ${HDF5_LIB}/libhdf5_cpp${SUFFFIX} ${HDF5_LIB}/libhdf5${SUFFFIX} )

# not sure how to add a subdirectory within atlas properly, so we have
# this hack
set(HDFTUPLE_INCLUDE_DIR ${CMAKE_CURRENT_SOURCE_DIR}/ttree2hdf5/include)
set(HDFTUPLE_LIBRARIES ${CMAKE_LIBRARY_OUTPUT_DIRECTORY}/libHdfTuple-static.a)

# Build the test executable:
atlas_add_executable( dump-test
  utils/dump-test.cxx src/BTagJetWriter.cxx
  src/BTagTrackWriter.cxx
  src/Jet.cxx src/Track.hh
  INCLUDE_DIRS ${ROOT_INCLUDE_DIRS} src ${HDF5_INCLUDE_DIRS}
  LINK_LIBRARIES ${ROOT_LIBRARIES}
  xAODRootAccess xAODCaloEvent xAODTracking xAODJet xAODTruth AsgTools JetCalibToolsLib JetSelectorToolsLib InDetTrackSelectionToolLib JetMomentToolsLib JetJvtEfficiencyLib )

if( BUILTIN_HDF5 )
  add_dependencies(dump-test HDF5)
endif()

# Add HDF5 libraries
target_link_libraries(dump-test ${HDF5_LIBRARIES} ${HDFTUPLE_LIBRARIES})

# Add HDF5 ntuple writer
target_include_directories(dump-test
  PRIVATE ${HDFTUPLE_INCLUDE_DIR}
  ${HDF5_INCLUDE_DIRS})
