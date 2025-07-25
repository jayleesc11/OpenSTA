# OpenSTA, Static Timing Analyzer
# Copyright (c) 2025, Parallax Software, Inc.
# 
# This program is free software: you can redistribute it and/or modify
# it under the terms of the GNU General Public License as published by
# the Free Software Foundation, either version 3 of the License, or
# (at your option) any later version.
# 
# This program is distributed in the hope that it will be useful,
# but WITHOUT ANY WARRANTY; without even the implied warranty of
# MERCHANTABILITY or FITNESS FOR A PARTICULAR PURPOSE. See the
# GNU General Public License for more details.
# 
# You should have received a copy of the GNU General Public License
# along with this program. If not, see <https://www.gnu.org/licenses/>.
# 
# The origin of this software must not be misrepresented; you must not
# claim that you wrote the original software.
# 
# Altered source versions must be plainly marked as such, and must not be
# misrepresented as being the original software.
# 
# This notice may not be removed or altered from any source distribution.

cmake_minimum_required (VERSION 3.10)
if(CMAKE_VERSION VERSION_GREATER_EQUAL 3.13)
# Use standard target names
cmake_policy(SET CMP0078 NEW)
endif()
if(CMAKE_VERSION VERSION_GREATER_EQUAL 3.14)
# Allows SWIG_MODULE_NAME to be set
cmake_policy(SET CMP0086 NEW)
endif()

project(STA VERSION 2.7.0
  LANGUAGES CXX
)

# Set CUDD directory directly (no need to specify with -DCUDD_DIR)
if(NOT CUDD_DIR)
  set(CUDD_DIR "${CMAKE_CURRENT_SOURCE_DIR}/../../../cudd-3.0.0")
endif()
message(STATUS "CUDD BDD package directory: ${CUDD_DIR}")

option(ENABLE_TSAN "Compile with thread santizer enabled" OFF)

# Turn on to debug compiler args.
set(CMAKE_VERBOSE_MAKEFILE OFF)

set(STA_HOME ${PROJECT_SOURCE_DIR})
message(STATUS "STA version: ${PROJECT_VERSION}")

list(APPEND CMAKE_MODULE_PATH "${CMAKE_CURRENT_SOURCE_DIR}/cmake")
include(GetGitRevisionDescription)
get_git_head_revision(GIT_REFSPEC STA_GIT_SHA1)
message(STATUS "STA git sha: ${STA_GIT_SHA1}")

# Default to bulding optimnized/release executable.
if(NOT CMAKE_BUILD_TYPE)
  set(CMAKE_BUILD_TYPE RELEASE)
endif()

# Static linking configuration
set(CMAKE_FIND_LIBRARY_SUFFIXES ".a")
set(BUILD_SHARED_LIBS OFF)
set(CMAKE_EXE_LINKER_FLAGS "${CMAKE_EXE_LINKER_FLAGS} -static -static-libgcc -static-libstdc++")

message(STATUS "System name: ${CMAKE_SYSTEM_NAME}")
message(STATUS "Compiler: ${CMAKE_CXX_COMPILER_ID} ${CMAKE_CXX_COMPILER_VERSION}")
message(STATUS "Build type: ${CMAKE_BUILD_TYPE}")
message(STATUS "Build CXX_FLAGS: ${CMAKE_CXX_FLAGS_${CMAKE_BUILD_TYPE}}")
message(STATUS "Install prefix: ${CMAKE_INSTALL_PREFIX}")

################################################################
#
# Source files.
#
################################################################

set(STA_SOURCE
  app/StaMain.cc
  
  dcalc/ArcDelayCalc.cc
  dcalc/ArcDcalcWaveforms.cc
  dcalc/ArnoldiDelayCalc.cc
  dcalc/ArnoldiReduce.cc
  dcalc/CcsCeffDelayCalc.cc
  dcalc/DcalcAnalysisPt.cc
  dcalc/DelayCalc.cc
  dcalc/DelayCalcBase.cc
  dcalc/DmpCeff.cc
  dcalc/DmpDelayCalc.cc
  dcalc/FindRoot.cc
  dcalc/GraphDelayCalc.cc
  dcalc/LumpedCapDelayCalc.cc
  dcalc/NetCaps.cc
  dcalc/ParallelDelayCalc.cc
  dcalc/PrimaDelayCalc.cc
  dcalc/UnitDelayCalc.cc
  
  graph/DelayFloat.cc
  graph/DelayNormal1.cc
  graph/DelayNormal2.cc
  graph/Graph.cc
  graph/GraphCmp.cc
  
  liberty/EquivCells.cc
  liberty/FuncExpr.cc
  liberty/InternalPower.cc
  liberty/LeakagePower.cc
  liberty/Liberty.cc
  liberty/LibertyBuilder.cc
  liberty/LibExprReader.cc
  liberty/LibertyParser.cc
  liberty/LibertyReader.cc
  liberty/LibertyWriter.cc
  liberty/LinearModel.cc
  liberty/Sequential.cc
  liberty/TableModel.cc
  liberty/TimingArc.cc
  liberty/TimingModel.cc
  liberty/TimingRole.cc
  liberty/Units.cc
  liberty/Wireload.cc
  
  network/ConcreteLibrary.cc
  network/ConcreteNetwork.cc
  network/HpinDrvrLoad.cc
  network/Network.cc
  network/NetworkCmp.cc
  network/ParseBus.cc
  network/PortDirection.cc
  network/SdcNetwork.cc
  network/VerilogNamespace.cc
  
  parasitics/ConcreteParasitics.cc
  parasitics/EstimateParasitics.cc
  parasitics/Parasitics.cc
  parasitics/ReduceParasitics.cc
  parasitics/ReportParasiticAnnotation.cc
  parasitics/SpefNamespace.cc
  parasitics/SpefReader.cc
  parasitics/SpefReaderPvt.hh

  power/Power.cc
  power/VcdReader.cc
  power/SaifReader.cc
  power/VcdParse.cc
  
  sdc/Clock.cc
  sdc/ClockGatingCheck.cc
  sdc/ClockGroups.cc
  sdc/ClockInsertion.cc
  sdc/ClockLatency.cc
  sdc/CycleAccting.cc
  sdc/DataCheck.cc
  sdc/DeratingFactors.cc
  sdc/DisabledPorts.cc
  sdc/ExceptionPath.cc
  sdc/InputDrive.cc
  sdc/PinPair.cc
  sdc/PortDelay.cc
  sdc/PortExtCap.cc
  sdc/Sdc.cc
  sdc/SdcGraph.cc
  sdc/SdcCmdComment.cc
  sdc/Variables.cc
  sdc/WriteSdc.cc
  
  sdf/ReportAnnotation.cc
  sdf/SdfReader.cc
  sdf/SdfWriter.cc
  
  search/Bdd.cc
  search/Bfs.cc
  search/CheckMaxSkews.cc
  search/CheckMinPeriods.cc
  search/CheckMinPulseWidths.cc
  search/CheckCapacitanceLimits.cc
  search/CheckFanoutLimits.cc
  search/CheckSlewLimits.cc
  search/CheckTiming.cc
  search/ClkInfo.cc
  search/ClkLatency.cc
  search/ClkNetwork.cc
  search/ClkSkew.cc
  search/Corner.cc
  search/Crpr.cc
  search/FindRegister.cc
  search/GatedClk.cc
  search/Genclks.cc
  search/Latches.cc
  search/Levelize.cc
  search/MakeTimingModel.cc
  search/Path.cc
  search/PathAnalysisPt.cc
  search/Path.cc
  search/PathEnd.cc
  search/PathEnum.cc
  search/PathExpanded.cc
  search/PathGroup.cc
  search/Property.cc
  search/ReportPath.cc
  search/Search.cc
  search/SearchPred.cc
  search/Sim.cc
  search/Sta.cc
  search/StaState.cc
  search/Tag.cc
  search/TagGroup.cc
  search/VertexVisitor.cc
  search/VisitPathEnds.cc
  search/VisitPathGroupVertices.cc
  search/WorstSlack.cc

  spice/WritePathSpice.cc
  spice/WriteSpice.cc
  spice/Xyce.cc

  tcl/TclTypeHelpers.cc

  util/Debug.cc
  util/DispatchQueue.cc
  util/Error.cc
  util/Fuzzy.cc
  util/Hash.cc
  util/Machine.cc
  util/MinMax.cc
  util/PatternMatch.cc
  util/Report.cc
  util/ReportStd.cc
  util/ReportTcl.cc
  util/RiseFallMinMax.cc
  util/RiseFallValues.cc
  util/Stats.cc
  util/StringSeq.cc
  util/StringSet.cc
  util/StringUtil.cc
  util/TokenParser.cc
  util/Transition.cc
  
  verilog/VerilogReader.cc
  verilog/VerilogWriter.cc
  )

# Source files.
set(STA_TCL_FILES
  tcl/Init.tcl
  tcl/Util.tcl
  tcl/CmdArgs.tcl
  tcl/CmdUtil.tcl
  tcl/Property.tcl
  tcl/Sta.tcl
  tcl/Splash.tcl
  tcl/Variables.tcl
  dcalc/DelayCalc.tcl
  graph/Graph.tcl
  liberty/Liberty.tcl
  network/Link.tcl
  network/Network.tcl
  network/NetworkEdit.tcl
  parasitics/Parasitics.tcl
  power/Power.tcl
  sdc/Sdc.tcl
  sdf/Sdf.tcl
  search/Search.tcl
  spice/WriteSpice.tcl
  verilog/Verilog.tcl
  )

################################################################
#
# Flex/bison scanner/parsers
#
################################################################

# Earlier versions of flex use 'register' declarations that are illegal in c++17.
#find_package(FLEX 2.6.4)
find_package(FLEX)
find_package(BISON REQUIRED 3.2)

# Verilog scan/parse.
flex_target(VerilogLex ${STA_HOME}/verilog/VerilogLex.ll
  ${CMAKE_CURRENT_BINARY_DIR}/VerilogLex.cc)
bison_target(VerilogParse ${STA_HOME}/verilog/VerilogParse.yy
  ${CMAKE_CURRENT_BINARY_DIR}/VerilogParse.cc)
add_flex_bison_dependency(VerilogLex VerilogParse)

# Liberty scan/parse.
flex_target(LibertyLex ${STA_HOME}/liberty/LibertyLex.ll
  ${CMAKE_CURRENT_BINARY_DIR}/LibertyLex.cc)
bison_target(LibertyParse ${STA_HOME}/liberty/LibertyParse.yy
  ${CMAKE_CURRENT_BINARY_DIR}/LibertyParse.cc)
add_flex_bison_dependency(LibertyLex LibertyParse)

# Liberty Expr scan/parse.
flex_target(LibExprLex ${STA_HOME}/liberty/LibExprLex.ll
  ${CMAKE_CURRENT_BINARY_DIR}/LibExprLex.cc)
bison_target(LibExprParse ${STA_HOME}/liberty/LibExprParse.yy
  ${CMAKE_CURRENT_BINARY_DIR}/LibExprParse.cc)
add_flex_bison_dependency(LibExprLex LibExprParse)

# Sdf scan/parse.
flex_target(SdfLex ${STA_HOME}/sdf/SdfLex.ll
  ${CMAKE_CURRENT_BINARY_DIR}/SdfLex.cc)
bison_target(SdfParse ${STA_HOME}/sdf/SdfParse.yy
  ${CMAKE_CURRENT_BINARY_DIR}/SdfParse.cc)
add_flex_bison_dependency(SdfLex SdfParse)

# Spef scan/parse.
flex_target(SpefLex ${STA_HOME}/parasitics/SpefLex.ll
  ${CMAKE_CURRENT_BINARY_DIR}/SpefLex.cc)
bison_target(SpefParse ${STA_HOME}/parasitics/SpefParse.yy
  ${CMAKE_CURRENT_BINARY_DIR}/SpefParse.cc)
add_flex_bison_dependency(SpefLex SpefParse)

# Saif scan/parse.
flex_target(SaifLex ${STA_HOME}/power/SaifLex.ll
  ${CMAKE_CURRENT_BINARY_DIR}/SaifLex.cc)
bison_target(SaifParse ${STA_HOME}/power/SaifParse.yy
  ${CMAKE_CURRENT_BINARY_DIR}/SaifParse.cc)
add_flex_bison_dependency(SaifLex SaifParse)

################################################################

set(STA_TCL_INIT ${CMAKE_CURRENT_BINARY_DIR}/StaTclInitVar.cc)

# TCL files included as part of the executable are shoved into StaTclInitVar.cc.
# These files are encoded and shipped as part of the executable 
# so that they do not have to be installed on the client host.
add_custom_command(OUTPUT ${STA_TCL_INIT}
  COMMAND etc/TclEncode.tcl ${STA_TCL_INIT} tcl_inits ${STA_TCL_FILES}
  WORKING_DIRECTORY ${STA_HOME}
  DEPENDS ${STA_TCL_FILES} etc/TclEncode.tcl
  )

################################################################
#
# Library dependencies
#
################################################################

# OSX tcl is deprecated and prints a warning, so look for a user installed
# version before using the system version.
# I tried to override the library search order instead but failed.
# CMAKE_FIND_FRAMEWORK LAST bypasses the version in the framework directory
# but not the one in /usr/lib.
# This calls cmake/FindTCL.cmake
# Do not use REQUIRED because it also requires TK, which is not used by OpenSTA.
# For static linking, explicitly find static TCL library
find_library(TCL_LIBRARY 
  NAMES tcl8.6 tcl86 tcl
  PATHS /usr/lib/x86_64-linux-gnu /usr/lib /lib
  NO_DEFAULT_PATH
)
find_path(TCL_INCLUDE_PATH tcl.h 
  PATHS /usr/include/tcl8.6 /usr/include
  NO_DEFAULT_PATH
)

# Set TCL_LIBRARY_PATH to embed TCL library path at compile time
set(TCL_LIBRARY_PATH "/usr/share/tcltk/tcl8.6")
add_definitions(-DTCL_LIBRARY_PATH="${TCL_LIBRARY_PATH}")

# Disable TCL_READLINE to avoid readline dependency issues in static linking
set(USE_TCL_READLINE OFF)

# Referenced by util/StaConfig.hh.cmake
set(TCL_READLINE 0)

# Zlib
include(FindZLIB)

find_package(Threads)

find_package(Eigen3 REQUIRED)

include(cmake/FindCUDD.cmake)

if("${SSTA}" STREQUAL "")
  set(SSTA 0)
endif()
message(STATUS "SSTA: ${SSTA}")

# configure a header file to pass some of the CMake settings
configure_file(${STA_HOME}/util/StaConfig.hh.cmake
  ${CMAKE_CURRENT_BINARY_DIR}/include/sta/StaConfig.hh
  )

###########################################################
# Swig
###########################################################

find_package(SWIG 3.0 REQUIRED)
include(UseSWIG)

set(STA_SWIG_FILE app/StaApp.i)

set_property(SOURCE ${STA_SWIG_FILE}
  PROPERTY CPLUSPLUS ON
)
# Ubuntu 18.04 cmake version 3.10.2 does not support the
# COMPILE_OPTIONS and INCLUDE_DIRECTORIES properties, so cram
# them into SWIG_FLAGS for the time being.
set_property(SOURCE ${STA_SWIG_FILE}
  PROPERTY SWIG_FLAGS
  -module sta
  -namespace -prefix sta
  -I${STA_HOME}
)

set(SWIG_FILES
  ${STA_HOME}/dcalc/DelayCalc.i
  ${STA_HOME}/graph/Graph.i
  ${STA_HOME}/liberty/Liberty.i
  ${STA_HOME}/network/Network.i
  ${STA_HOME}/network/NetworkEdit.i
  ${STA_HOME}/parasitics/Parasitics.i
  ${STA_HOME}/power/Power.i
  ${STA_HOME}/sdc/Sdc.i
  ${STA_HOME}/sdf/Sdf.i
  ${STA_HOME}/search/Property.i
  ${STA_HOME}/search/Search.i
  ${STA_HOME}/spice/WriteSpice.i
  ${STA_HOME}/tcl/Exception.i
  ${STA_HOME}/tcl/StaTclTypes.i
  ${STA_HOME}/util/Util.i
  ${STA_HOME}/verilog/Verilog.i
  )

set_property(SOURCE ${STA_SWIG_FILE}
  PROPERTY DEPENDS ${SWIG_FILES}
)

swig_add_library(sta_swig
  LANGUAGE tcl
  TYPE STATIC
  SOURCES ${STA_SWIG_FILE}
)

get_target_property(STA_SWIG_CXX_FILE sta_swig SOURCES)

set_source_files_properties(${STA_SWIG_CXX_FILE}
  PROPERTIES
  # No simple way to modify the swig template that emits code full of warnings
  # so suppress them.
  COMPILE_OPTIONS "-Wunused-variable;-Wno-cast-qual;-Wno-missing-braces;-Wno-deprecated-declarations"
  )

target_link_libraries(sta_swig
  PUBLIC
  OpenSTA
)

# result build/CMakeFiles/sta_swig.dir/StaAppTCL_wrap.cxx
target_include_directories(sta_swig
  PUBLIC
  include/sta

  PRIVATE
  ${STA_HOME}
  ${TCL_INCLUDE_PATH}
  ${CMAKE_CURRENT_BINARY_DIR}/include/sta
)

###########################################################
# Library
###########################################################

add_library(OpenSTA)

target_sources(OpenSTA
  PRIVATE
  ${STA_SOURCE}

  ${STA_TCL_INIT}

  ${FLEX_LibExprLex_OUTPUTS}
  ${BISON_LibExprParse_OUTPUTS}
  ${FLEX_LibertyLex_OUTPUTS}
  ${BISON_LibertyParse_OUTPUTS}

  ${FLEX_SpefLex_OUTPUTS}
  ${BISON_SpefParse_OUTPUTS}

  ${FLEX_SdfLex_OUTPUTS}
  ${BISON_SdfParse_OUTPUTS}

  ${FLEX_VerilogLex_OUTPUTS}
  ${BISON_VerilogParse_OUTPUTS}

  ${FLEX_SaifLex_OUTPUTS}
  ${BISON_SaifParse_OUTPUTS}
)

target_link_libraries(OpenSTA
  Eigen3::Eigen
  ${TCL_LIBRARY}
  ${CMAKE_THREAD_LIBS_INIT}
  ${CUDD_LIB}
  )

if (ZLIB_LIBRARIES)
  target_link_libraries(OpenSTA ${ZLIB_LIBRARIES})
endif()

target_include_directories(OpenSTA
  PUBLIC
  include
  ${TCL_INCLUDE_PATH}
  ${FLEX_INCLUDE_DIRS}
  ${CMAKE_CURRENT_BINARY_DIR}/include/sta

  PRIVATE
  include/sta
  ${STA_HOME}
  ${CUDD_INCLUDE}
  # For flex to find location.hh
  ${CMAKE_CURRENT_BINARY_DIR}
  )

# common to gcc/clang
set(CXX_FLAGS -Wall -Wextra -pedantic -Wcast-qual -Wredundant-decls -Wformat-security)

if(ENABLE_TSAN)
  message(STATUS "Thread sanitizer: ${ENABLE_TSAN}")
  set(CXX_FLAGS "${CXX_FLAGS};-fsanitize=thread")
  set(CMAKE_EXE_LINKER_FLAGS "-fsanitize=thread")
endif()

target_compile_options(OpenSTA
  PRIVATE
  $<$<CXX_COMPILER_ID:GNU>:${CXX_FLAGS}>
  $<$<CXX_COMPILER_ID:Clang>:${CXX_FLAGS} -Wno-gnu-zero-variadic-macro-arguments>
  $<$<CXX_COMPILER_ID:AppleClang>:${CXX_FLAGS} -Wno-gnu-zero-variadic-macro-arguments>
  )

# Disable compiler specific extensions like gnu++11.
set_target_properties(OpenSTA PROPERTIES CXX_EXTENSIONS OFF)
target_compile_features(OpenSTA PUBLIC cxx_std_17)

message(STATUS "STA library: ${CMAKE_BINARY_DIR}/libOpenSTA.a")

###########################################################
# Executable
###########################################################

# Note executable and lib name cannot be the same because
# on osx something is case insensitive. Using STA for the
# lib name results in "No rule to make target ../depend.
add_executable(sta app/Main.cc)

target_link_libraries(sta
  sta_swig
  OpenSTA
  )

# Add static linking for essential system libraries only
target_link_libraries(sta
  -static
  -Wl,--whole-archive
  -lpthread
  -Wl,--no-whole-archive
  -ldl
  -lz
  -lm
  -lrt
)

# Set TCL_LIBRARY path at compile time for static linking
target_compile_definitions(sta PRIVATE
  TCL_LIBRARY_PATH="/usr/share/tcltk/tcl8.6"
)

message(STATUS "STA executable: ${CMAKE_BINARY_DIR}/sta")

################################################################
# Install
# cmake .. -DCMAKE_INSTALL_PREFIX=<prefix_path>

# executable
install(TARGETS sta DESTINATION bin)

# library
install(TARGETS OpenSTA DESTINATION lib)

# include
install(DIRECTORY include/sta  DESTINATION include)
install(DIRECTORY ${CMAKE_CURRENT_BINARY_DIR}/include/sta  DESTINATION include)

################################################################

add_custom_target(sta_tags
  COMMAND etags -o TAGS
  ${STA_SOURCE}
  */*.hh
  include/sta/*.hh
  ${CMAKE_CURRENT_BINARY_DIR}/include/sta/*.hh
  ${SWIG_FILES}
  ${SWIG_FILES}

  # Append tcl tags.
  # Note that the regexp used for tcl files is broken in etags so provide one that works.
  COMMAND etags -a -o TAGS --language=none
  --regex='/^[ \\t]*proc[ \\t]+[a-zA-Z0-9_]+/'
  ${STA_TCL_FILES}
  WORKING_DIRECTORY ${STA_HOME}
)

add_custom_command(
  TARGET OpenSTA
  POST_BUILD
  COMMAND ${CMAKE_SOURCE_DIR}/etc/FindMessages.tcl > ${CMAKE_SOURCE_DIR}/doc/messages.txt || true
  WORKING_DIRECTORY ${CMAKE_SOURCE_DIR}
)
