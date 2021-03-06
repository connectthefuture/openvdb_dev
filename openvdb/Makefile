# Copyright (c) 2012-2016 DreamWorks Animation LLC
#
# All rights reserved. This software is distributed under the
# Mozilla Public License 2.0 ( http://www.mozilla.org/MPL/2.0/ )
#
# Redistributions of source code must retain the above copyright
# and license notice and the following restrictions and disclaimer.
#
# *     Neither the name of DreamWorks Animation nor the names of
# its contributors may be used to endorse or promote products derived
# from this software without specific prior written permission.
#
# THIS SOFTWARE IS PROVIDED BY THE COPYRIGHT HOLDERS AND CONTRIBUTORS
# "AS IS" AND ANY EXPRESS OR IMPLIED WARRANTIES, INCLUDING, BUT NOT
# LIMITED TO, THE IMPLIED WARRANTIES OF MERCHANTABILITY AND FITNESS FOR
# A PARTICULAR PURPOSE ARE DISCLAIMED. IN NO EVENT SHALL THE COPYRIGHT
# OWNER OR CONTRIBUTORS BE LIABLE FOR ANY INDIRECT, INCIDENTAL,
# SPECIAL, EXEMPLARY, OR CONSEQUENTIAL DAMAGES (INCLUDING, BUT NOT
# LIMITED TO, PROCUREMENT OF SUBSTITUTE GOODS OR SERVICES; LOSS OF USE,
# DATA, OR PROFITS; OR BUSINESS INTERRUPTION) HOWEVER CAUSED AND ON ANY
# THEORY OF LIABILITY, WHETHER IN CONTRACT, STRICT LIABILITY, OR TORT
# (INCLUDING NEGLIGENCE OR OTHERWISE) ARISING IN ANY WAY OUT OF THE USE
# OF THIS SOFTWARE, EVEN IF ADVISED OF THE POSSIBILITY OF SUCH DAMAGE.
# IN NO EVENT SHALL THE COPYRIGHT HOLDERS' AND CONTRIBUTORS' AGGREGATE
# LIABILITY FOR ALL CLAIMS REGARDLESS OF THEIR BASIS EXCEED US$250.00.
#
# Makefile for the OpenVDB library

# See INSTALL for a list of requirements.
#
# Targets:
#   lib                 the OpenVDB library
#
#   doc                 HTML documentation (doc/html/index.html)
#   pdfdoc              PDF documentation (doc/latex/refman.pdf;
#                       requires LaTeX and ghostscript)
#   python              OpenVDB Python module
#   pytest              unit tests for the Python module
#   pydoc               HTML documentation for the Python module
#                       (doc/html/python/index.html)
#   vdb_print           command-line tool to inspect OpenVDB files
#   vdb_render          command-line tool to ray-trace OpenVDB files
#   vdb_view            command-line tool to view OpenVDB files
#   vdb_test            unit tests for the OpenVDB library
#
#   all                 [default target] all of the above
#   install             install all of the above except vdb_test
#                       into subdirectories of DESTDIR
#   depend              recompute source file header dependencies
#   clean               delete generated files from the local directory
#   test                run tests
#
# Options:
#   abi=3               build for compatibility with the OpenVDB 3.x Grid ABI
#                       (some OpenVDB 4.x features will be disabled)
#   shared=no           link executables against static OpenVDB libraries
#                       (default: link against shared libraries)
#   debug=yes           build with debugging symbols and without optimization
#   verbose=yes         run commands (e.g., doxygen) in verbose mode

#
# libConfig allows us to specify a single variable for whether we build
# debug/release and shared/static
ifdef libConfig
  ifeq (Debug,$(strip $(libConfig)))
    debug   := yes
    shared  := yes
  endif
  ifeq (DebugStatic,$(strip $(libConfig)))
    debug   := yes
    shared  := no
  endif
  ifeq (Release,$(strip $(libConfig)))
    debug   := no
    shared  := yes
  endif
  ifeq (ReleaseStatic,$(strip $(libConfig)))
    debug   := no
    shared  := no
  endif
  ifdef WINDOWS_NT
    ifeq (yes,$(strip $(debug)))
      debug := full
    endif
  endif
  $(info === Building $(libConfig): debug=$(debug) shared=$(shared) ===)
endif

#
# The following variables must be defined, either here or on the command line
# (e.g., "make install DESTDIR=/usr/local"):
#
# Note that if you plan to build the Houdini OpenVDB tools (distributed
# separately), you must build the OpenVDB library and the Houdini tools
# against compatible versions of the Boost, OpenEXR and TBB libraries.
# Fortunately, all three are included in the Houdini HDK, so the relevant
# variables below point by default to the HDK library and header directories:
# $(HDSO) and $(HT)/include, respectively.  (Source the houdini_setup script
# to set those two environment variables.)
#
# To build the OpenVDB Python module, you will need local distributions of
# Python, Boost.Python, and optionally NumPy.  As of Houdini 12.5, the HDK
# includes versions 2.5 and 2.6 of Python as well as the Boost.Python headers.
# Unfortunately, it does not include the libboost_python library, nor does it
# include NumPy, so both Boost.Python and NumPy have to be built separately.
# Point the variables $(BOOST_PYTHON_LIB_DIR), $(BOOST_PYTHON_LIB) and
# $(NUMPY_INCL_DIR) below to your local distributions of those libraries.
#

# The directory into which to install libraries, executables and header files
DESTDIR := $(prefix)
include platform.mk

ifeq (,$(prefix))
  $(warning === WARNING === No prefix set)
endif
LOCAL_INCLUDE	:= $(prefix)/include
LOCAL_LIB	:= $(prefix)/lib

BOOST_VERSION	?= 1_55
# The parent directory of the boost/ header directory
BOOST_INCL_DIR	:= $(LOCAL_INCLUDE)/boost-$(BOOST_VERSION)
# The directory containing libboost_iostreams, libboost_system, etc.
BOOST_LIB_DIR := $(LOCAL_LIB)
ifdef WINDOWS_NT
  BOOST_LIB :=
else
  BOOST_LIB := -lboost_iostreams -lboost_system
endif
BOOST_THREAD_LIB := -lboost_thread

# The parent directory of the OpenEXR/ header directory
EXR_INCL_DIR := $(LOCAL_INCLUDE)
# The directory containing libIlmImf, libIlmThread, etc.
EXR_LIB_DIR := $(LOCAL_LIB)
EXR_LIB := -lIlmImf -lIlmThread -lIex -lImath
ifdef WINDOWS_NT
  ifneq (full,$(strip $(debug)))
    ifeq (no,$(strip $(shared)))
      EXR_LIB := IlmImf.lib IlmThread.lib Iex.lib Imath.lib
    else
      EXR_LIB := IlmImf.lib IlmThread.lib Iex.lib Imath.lib
    endif
  else
    ifeq (no,$(strip $(shared)))
      EXR_LIB := IlmImf_d.lib IlmThread_d.lib Iex_d.lib Imath_d.lib
    else
      EXR_LIB := IlmImf_d.lib IlmThread_d.lib Iex_d.lib Imath_d.lib
    endif
  endif
else
  EXR_LIB := -lIlmImf -lIlmThread -lIex -lImath
endif

# The parent directory of the OpenEXR/ header directory (which contains half.h)
HALF_INCL_DIR := $(LOCAL_INCLUDE)
# The directory containing libHalf
HALF_LIB_DIR := $(LOCAL_LIB)
ifdef WINDOWS_NT
  ifneq (full,$(strip $(debug)))
    ifeq (no,$(strip $(shared)))
      HALF_LIB := libHalf.lib
    else
      HALF_LIB := Half.lib
    endif
  else
    ifeq (no,$(strip $(shared)))
      HALF_LIB := libHalf_d.lib
    else
      HALF_LIB := Half_d.lib
    endif
  endif
else
  HALF_LIB := -lHalf
endif

# The parent directory of the tbb/ header directory
TBB_INCL_DIR := $(LOCAL_INCLUDE)
# The directory containing libtbb
TBB_LIB_DIR := $(LOCAL_LIB)
ifdef WINDOWS_NT
  ifneq (full,$(strip $(debug)))
    ifeq (no,$(strip $(shared)))
      TBB_LIB := libtbb.lib
    else
      TBB_LIB := tbb.lib
    endif
  else
    ifeq (no,$(strip $(shared)))
      TBB_LIB := libtbb_debug.lib
    else
      TBB_LIB := tbb_debug.lib
    endif
  endif
else
  TBB_LIB := -ltbb
endif

# The parent directory of the blosc.h header
# (leave blank if Blosc is unavailable)
BLOSC_INCL_DIR := $(LOCAL_INCLUDE)
# The directory containing libblosc
BLOSC_LIB_DIR := $(LOCAL_LIB)
ifdef WINDOWS_NT
  ifneq (full,$(strip $(debug)))
    ifeq (no,$(strip $(shared)))
      BLOSC_LIB := libblosc.lib
    else
      BLOSC_LIB := blosc.lib
    endif
  else
    ifeq (no,$(strip $(shared)))
      BLOSC_LIB := libblosc_d.lib
    else
      BLOSC_LIB := blosc_d.lib
    endif
  endif
else
  BLOSC_LIB := -lblosc
endif

# The parent directory of the zlib header directory (which contains zlib.h)
ZLIB_INCL_DIR := $(LOCAL_INCLUDE)
ZLIB_LIB_DIR := $(LOCAL_LIB)
ifdef WINDOWS_NT
  ifneq (full,$(strip $(debug)))
    ifeq (no,$(strip $(shared)))
      ZLIB_LIB := zlib.lib
    else
      ZLIB_LIB := zdll.lib
    endif
  else
    ifeq (no,$(strip $(shared)))
      ZLIB_LIB := zlib_d.lib
    else
      ZLIB_LIB := zdll_d.lib
    endif
  endif
else
  ZLIB_LIB := -lz
endif

# A scalable, concurrent malloc replacement library
# such as jemalloc (included in the Houdini HDK) or TBB malloc
# (leave blank if unavailable)
ifdef WINDOWS_NT
  ifneq (full,$(strip $(debug)))
    CONCURRENT_MALLOC_LIB := tbbmalloc_proxy.lib
  else
    CONCURRENT_MALLOC_LIB := tbbmalloc_proxy_debug.lib
  endif
ifdef AMD64
CONCURRENT_MALLOC_LIB += /INCLUDE:__TBB_malloc_proxy
else
CONCURRENT_MALLOC_LIB += /INCLUDE:___TBB_malloc_proxy
endif
CONCURRENT_MALLOC_LIB_DIR := $(LOCAL_LIB)
else ifdef MBSD
CONCURRENT_MALLOC_LIB :=
CONCURRENT_MALLOC_LIB_DIR :=
else
CONCURRENT_MALLOC_LIB := -ljemalloc
#CONCURRENT_MALLOC_LIB := -ltbbmalloc_proxy -ltbbmalloc
# The directory containing the malloc replacement library
CONCURRENT_MALLOC_LIB_DIR := $(LOCAL_LIB)
endif

# The parent directory of the cppunit/ header directory
# (leave blank if CppUnit is unavailable)
CPPUNIT_INCL_DIR := $(LOCAL_INCLUDE)
# The directory containing libcppunit
CPPUNIT_LIB_DIR := $(LOCAL_LIB)
ifdef WINDOWS_NT
  ifneq (full,$(strip $(debug)))
    CPPUNIT_LIB := cppunit.lib
  else
    CPPUNIT_LIB := cppunitd.lib
  endif
else
  CPPUNIT_LIB := -lcppunit
endif

# The parent directory of the log4cplus/ header directory
# (leave blank if log4cplus is unavailable)
LOG4CPLUS_INCL_DIR :=
# The directory containing liblog4cplus
LOG4CPLUS_LIB_DIR :=
LOG4CPLUS_LIB :=

# The directory containing glfw.h
# (leave blank if GLFW is unavailable)
GLFW_INCL_DIR :=
# The directory containing libglfw
GLFW_LIB_DIR :=
GLFW_LIB :=
# The major version number of the GLFW library
# (header filenames changed between GLFW 2 and 3, so this must be specified explicitly)
GLFW_MAJOR_VERSION := 3

# The version of Python for which to build the OpenVDB module
# (leave blank if Python is unavailable)
PYTHON_VERSION := 2.7
PYTHON_VERSION_NO_DOT := 27
# The directory containing Python.h
PYTHON_INCL_DIR := $(HFS)/python/include/python$(PYTHON_VERSION)
# The directory containing pyconfig.h
PYCONFIG_INCL_DIR := $(PYTHON_INCL_DIR)
# The directory containing libpython
ifdef WINDOWS_NT
PYTHON_LIB_DIR := $(HFS)/python$(PYTHON_VERSION_NO_DOT)/libs
PYTHON_LIB := python$(PYTHON_VERSION_NO_DOT).lib
else ifdef LINUX
PYTHON_LIB_DIR := $(HFS)/python/lib
PYTHON_LIB := -lpython$(PYTHON_VERSION)
else ifdef MBSD
PYTHON_LIB_DIR := $(HFS)/Frameworks/Python.framework/Versions/$(PYTHON_VERSION)/lib
PYTHON_LIB := -lpython$(PYTHON_VERSION)
endif
# The directory containing libboost_python
BOOST_PYTHON_LIB_DIR :=
BOOST_PYTHON_LIB :=
# The directory containing arrayobject.h
# (leave blank if NumPy is unavailable)
NUMPY_INCL_DIR :=
# The Epydoc executable
# (leave blank if Epydoc is unavailable)
EPYDOC :=
# Set PYTHON_WRAP_ALL_GRID_TYPES to "yes" to specify that the Python module
# should expose (almost) all of the grid types defined in openvdb.h
# Otherwise, only FloatGrid, BoolGrid and Vec3SGrid will be exposed
# (see, e.g., exportIntGrid() in python/pyIntGrid.cc).
# Compiling the Python module with PYTHON_WRAP_ALL_GRID_TYPES set to "yes"
# can be very memory-intensive.
PYTHON_WRAP_ALL_GRID_TYPES := no
#
PYTHON_LIBS_RPATH := \
    -Wl,-rpath,$(PYTHON_LIB_DIR) -L$(PYTHON_LIB_DIR) $(PYTHON_LIB) \
    -Wl,-rpath,$(BOOST_PYTHON_LIB_DIR) -L$(BOOST_PYTHON_LIB_DIR) $(BOOST_PYTHON_LIB) \
#

# The Doxygen executable
# (leave blank if Doxygen is unavailable)
DOXYGEN :=


#
# Ideally, users shouldn't need to change anything below this line.
#

SHELL = /bin/bash

# Turn off implicit rules for speed.
.SUFFIXES:

ifneq (,$(INSTALL_DIR))
    $(warning Warning: $$(INSTALL_DIR) is no longer used; set $$(DESTDIR) instead.)
endif

ifeq (full,$(strip $(debug)))
  OPTIMIZE := $(DEBUG_FLAGS)
  ifdef WINDOWS_NT
    LDFLAGS += -DEBUG
  else
  endif
else
  ifeq (yes,$(strip $(debug)))
    OPTIMIZE := $(DEBUG_FLAGS)
    ifdef WINDOWS_NT
      LDFLAGS += -DEBUG
    endif
  else
    OPTIMIZE := $(OPTIMIZE_FLAGS)
  endif
endif
ifeq (,$(strip $(debug)))
  debug := no
endif
ifneq (no,$(strip $(debug)))
STATICLIB_SUFFIX := $(addprefix _d,$(STATICLIB_SUFFIX))
SHAREDLIB_SUFFIX := $(addprefix _d,$(SHAREDLIB_SUFFIX))
IMPORTLIB_SUFFIX := $(addprefix _d,$(IMPORTLIB_SUFFIX))
endif

ifeq (yes,$(strip $(verbose)))
    QUIET :=
    QUIET_TEST := -v
else
    QUIET := > /dev/null
    QUIET_TEST := $(QUIET)
endif

has_blosc := no
ifneq (,$(and $(BLOSC_LIB_DIR),$(BLOSC_INCL_DIR),$(BLOSC_LIB)))
    has_blosc := yes
endif

has_glfw := no
ifneq (,$(and $(GLFW_LIB_DIR),$(GLFW_INCL_DIR),$(GLFW_LIB)))
    has_glfw := yes
endif

has_log4cplus := no
ifneq (,$(and $(LOG4CPLUS_LIB_DIR),$(LOG4CPLUS_INCL_DIR),$(LOG4CPLUS_LIB)))
    has_log4cplus := yes
endif

has_python := no
ifneq (,$(and $(PYTHON_VERSION),$(PYTHON_LIB_DIR),$(PYTHON_INCL_DIR), \
    $(PYCONFIG_INCL_DIR),$(PYTHON_LIB),$(BOOST_PYTHON_LIB_DIR),$(BOOST_PYTHON_LIB)))
    has_python := yes
endif

INCLDIRS := -I . -I .. \
    $(CXXISYSTEM) $(BOOST_INCL_DIR) \
    $(CXXISYSTEM) $(HALF_INCL_DIR) \
    $(CXXISYSTEM) $(TBB_INCL_DIR) \
    $(CXXISYSTEM) $(ZLIB_INCL_DIR) \
#
ifeq (yes,$(has_blosc))
    INCLDIRS += $(CXXISYSTEM) $(BLOSC_INCL_DIR)
endif
ifeq (yes,$(has_log4cplus))
    INCLDIRS += $(CXXISYSTEM) $(LOG4CPLUS_INCL_DIR)
endif

CXXFLAGS += $(OPTIMIZE) $(INCLDIRS)

ifeq (yes,$(has_blosc))
    CXXFLAGS += -DOPENVDB_USE_BLOSC
endif
ifeq (yes,$(has_log4cplus))
    CXXFLAGS += -DOPENVDB_USE_LOG4CPLUS
endif
ifeq (3,$(strip $(abi)))
    CXXFLAGS += -DOPENVDB_3_ABI_COMPATIBLE
endif
ifneq (2,$(strip $(GLFW_MAJOR_VERSION)))
    CXXFLAGS += -DOPENVDB_USE_GLFW_3
endif

LIBS := \
    -L $(HALF_LIB_DIR) $(HALF_LIB) \
    -L $(TBB_LIB_DIR) $(TBB_LIB) \
    -L $(ZLIB_LIB_DIR) $(ZLIB_LIB) \
    -L $(BOOST_LIB_DIR) $(BOOST_LIB) \
#
LIBS_RPATH := \
    -Wl,-rpath,'$$ORIGIN/.' $(LDDIROPT)$(HALF_LIB_DIR) $(HALF_LIB) \
    -Wl,-rpath,'$$ORIGIN/.' $(LDDIROPT)$(TBB_LIB_DIR) $(TBB_LIB) \
    -Wl,-rpath,'$$ORIGIN/.' $(LDDIROPT)$(ZLIB_LIB_DIR) $(ZLIB_LIB) \
    -Wl,-rpath,'$$ORIGIN/.' $(LDDIROPT)$(BOOST_LIB_DIR) $(BOOST_LIB) \
#
ifeq (yes,$(has_blosc))
LIBS += \
    -L $(BLOSC_LIB_DIR) $(BLOSC_LIB)
LIBS_RPATH += \
    -Wl,-rpath,'$$ORIGIN/.' $(LDDIROPT)$(BLOSC_LIB_DIR) $(BLOSC_LIB)
endif
ifeq (yes,$(has_log4cplus))
LOG4CPLUS_LIBS := -L $(LOG4CPLUS_LIB_DIR) $(LOG4CPLUS_LIB)
LOG4CPLUS_LIBS_RPATH := \
    -Wl,-rpath,'$$ORIGIN/.' $(LDDIROPT)$(LOG4CPLUS_LIB_DIR) $(LOG4CPLUS_LIB)
else
LOG4CPLUS_LIBS :=
LOG4CPLUS_LIBS_RPATH :=
endif
ifneq (,$(strip $(CONCURRENT_MALLOC_LIB)))
ifneq (,$(strip $(CONCURRENT_MALLOC_LIB_DIR)))
ALLOC_LIBS = -L $(CONCURRENT_MALLOC_LIB_DIR) $(CONCURRENT_MALLOC_LIB)
ALLOC_LIBS_RPATH := \
    -Wl,-rpath,'$$ORIGIN/.' $(LDDIROPT)$(CONCURRENT_MALLOC_LIB_DIR) $(CONCURRENT_MALLOC_LIB)
else
ALLOC_LIBS :=
ALLOC_LIBS_RPATH :=
endif
endif

INCLUDE_NAMES := \
    Exceptions.h \
    Grid.h \
    io/Archive.h \
    io/Compression.h \
    io/File.h \
    io/GridDescriptor.h \
    io/io.h \
    io/Queue.h \
    io/Stream.h \
    io/TempFile.h \
    math/BBox.h \
    math/ConjGradient.h \
    math/Coord.h \
    math/DDA.h \
    math/FiniteDifference.h \
    math/LegacyFrustum.h \
    math/Maps.h \
    math/Mat.h \
    math/Mat3.h \
    math/Mat4.h \
    math/Math.h \
    math/Operators.h \
    math/Proximity.h \
    math/QuantizedUnitVec.h \
    math/Quat.h \
    math/Ray.h \
    math/Stats.h \
    math/Stencils.h \
    math/Transform.h\
    math/Tuple.h\
    math/Vec2.h \
    math/Vec3.h \
    math/Vec4.h \
    Metadata.h \
    MetaMap.h \
    openvdb.h \
    Platform.h \
    PlatformConfig.h \
    points/AttributeArray.h \
    points/AttributeArrayString.h \
    points/AttributeGroup.h \
    points/AttributeSet.h \
    points/IndexFilter.h \
    points/IndexIterator.h \
    points/PointAttribute.h \
    points/PointConversion.h \
    points/PointCount.h \
    points/PointDataGrid.h \
    points/PointGroup.h \
    points/StreamCompression.h \
    tools/ChangeBackground.h \
    tools/Clip.h \
    tools/Composite.h \
    tools/Dense.h \
    tools/DenseSparseTools.h \
    tools/Diagnostics.h \
    tools/Filter.h \
    tools/GridOperators.h \
    tools/GridTransformer.h \
    tools/Interpolation.h \
    tools/LevelSetAdvect.h \
    tools/LevelSetFilter.h \
    tools/LevelSetFracture.h \
    tools/LevelSetMeasure.h \
    tools/LevelSetMorph.h \
    tools/LevelSetPlatonic.h \
    tools/LevelSetRebuild.h \
    tools/LevelSetSphere.h \
    tools/LevelSetTracker.h \
    tools/LevelSetUtil.h \
    tools/MeshToVolume.h \
    tools/Morphology.h \
    tools/MultiResGrid.h \
    tools/ParticleAtlas.h \
    tools/ParticlesToLevelSet.h \
    tools/PointAdvect.h \
    tools/PointIndexGrid.h \
    tools/PointPartitioner.h \
    tools/PointScatter.h \
    tools/PointsToMask.h \
    tools/PoissonSolver.h \
    tools/Prune.h \
    tools/RayIntersector.h \
    tools/RayTracer.h \
    tools/SignedFloodFill.h \
    tools/Statistics.h \
    tools/TopologyToLevelSet.h \
    tools/ValueTransformer.h \
    tools/VectorTransformer.h \
    tools/VelocityFields.h \
    tools/VolumeAdvect.h \
    tools/VolumeToMesh.h \
    tools/VolumeToSpheres.h \
    tree/InternalNode.h \
    tree/Iterator.h \
    tree/LeafBuffer.h \
    tree/LeafManager.h \
    tree/LeafNode.h \
    tree/LeafNodeBool.h \
    tree/LeafNodeMask.h \
    tree/NodeManager.h \
    tree/NodeUnion.h \
    tree/RootNode.h \
    tree/Tree.h \
    tree/TreeIterator.h \
    tree/ValueAccessor.h \
    Types.h \
    util/CpuTimer.h \
    util/Formats.h \
    util/logging.h \
    util/MapsUtil.h \
    util/Name.h \
    util/NodeMasks.h \
    util/NullInterrupter.h \
    util/PagedArray.h \
    util/Util.h \
    version.h \
#

SRC_NAMES := \
    Grid.cc \
    io/Archive.cc \
    io/Compression.cc \
    io/File.cc \
    io/GridDescriptor.cc \
    io/Queue.cc \
    io/Stream.cc \
    io/TempFile.cc \
    math/Maps.cc \
    math/Proximity.cc \
    math/QuantizedUnitVec.cc \
    math/Transform.cc \
    Metadata.cc \
    MetaMap.cc \
    openvdb.cc \
    Platform.cc \
    points/AttributeArray.cc \
    points/AttributeArrayString.cc \
    points/AttributeGroup.cc \
    points/AttributeSet.cc \
    points/points.cc \
    points/StreamCompression.cc \
    util/Formats.cc \
    util/Util.cc \
#

UNITTEST_INCLUDE_NAMES := \
    unittest/util.h \
#

UNITTEST_SRC_NAMES := \
    unittest/main.cc \
    unittest/TestAttributeArray.cc \
    unittest/TestAttributeArrayString.cc \
    unittest/TestAttributeSet.cc \
    unittest/TestAttributeGroup.cc \
    unittest/TestBBox.cc \
    unittest/TestConjGradient.cc \
    unittest/TestCoord.cc \
    unittest/TestCpt.cc \
    unittest/TestCurl.cc \
    unittest/TestDense.cc \
    unittest/TestDenseSparseTools.cc \
    unittest/TestDiagnostics.cc \
    unittest/TestDivergence.cc \
    unittest/TestDoubleMetadata.cc \
    unittest/TestExceptions.cc \
    unittest/TestFile.cc \
    unittest/TestFloatMetadata.cc \
    unittest/TestGradient.cc \
    unittest/TestGrid.cc \
    unittest/TestGridBbox.cc \
    unittest/TestGridDescriptor.cc \
    unittest/TestGridIO.cc \
    unittest/TestGridTransformer.cc \
    unittest/TestIndexFilter.cc \
    unittest/TestIndexIterator.cc \
    unittest/TestInit.cc \
    unittest/TestInt32Metadata.cc \
    unittest/TestInt64Metadata.cc \
    unittest/TestInternalOrigin.cc \
    unittest/TestLaplacian.cc \
    unittest/TestLeaf.cc \
    unittest/TestLeafBool.cc \
    unittest/TestLeafManager.cc \
    unittest/TestLeafMask.cc \
    unittest/TestLeafIO.cc \
    unittest/TestLeafOrigin.cc \
    unittest/TestLevelSetRayIntersector.cc \
    unittest/TestLevelSetUtil.cc \
    unittest/TestLinearInterp.cc \
    unittest/TestMaps.cc \
    unittest/TestMat4Metadata.cc \
    unittest/TestMath.cc \
    unittest/TestMeanCurvature.cc \
    unittest/TestMeshToVolume.cc \
    unittest/TestMetadata.cc \
    unittest/TestMetadataIO.cc \
    unittest/TestMetaMap.cc \
    unittest/TestMultiResGrid.cc \
    unittest/TestName.cc \
    unittest/TestNodeIterator.cc \
    unittest/TestNodeManager.cc \
    unittest/TestNodeMask.cc \
    unittest/TestParticleAtlas.cc \
    unittest/TestParticlesToLevelSet.cc \
    unittest/TestPointAttribute.cc \
    unittest/TestPointConversion.cc \
    unittest/TestPointCount.cc \
    unittest/TestPointDataLeaf.cc \
    unittest/TestPointGroup.cc \
    unittest/TestPointIndexGrid.cc \
    unittest/TestPointPartitioner.cc \
    unittest/TestPointsToMask.cc \
    unittest/TestPoissonSolver.cc \
    unittest/TestPrePostAPI.cc \
    unittest/TestQuadraticInterp.cc \
    unittest/TestQuantizedUnitVec.cc \
    unittest/TestQuat.cc \
    unittest/TestRay.cc \
    unittest/TestStats.cc \
    unittest/TestStream.cc \
    unittest/TestStreamCompression.cc \
    unittest/TestStringMetadata.cc \
    unittest/TestTools.cc \
    unittest/TestTopologyToLevelSet.cc \
    unittest/TestTransform.cc \
    unittest/TestTree.cc \
    unittest/TestTreeCombine.cc \
    unittest/TestTreeGetSetValues.cc \
    unittest/TestTreeIterators.cc \
    unittest/TestTreeVisitor.cc \
    unittest/TestUtil.cc \
    unittest/TestValueAccessor.cc \
    unittest/TestVec2Metadata.cc \
    unittest/TestVec3Metadata.cc \
    unittest/TestVolumeRayIntersector.cc \
    unittest/TestVolumeToMesh.cc \
    unittest/TestVolumeToSpheres.cc \
#

DOC_FILES := doc/doc.txt doc/faq.txt doc/changes.txt doc/codingstyle.txt doc/examplecode.txt doc/api_0_98_0.txt doc/math.txt doc/python.txt
DOC_INDEX := doc/html/index.html
DOC_PDF := doc/latex/refman.pdf

LIBVIEWER_INCLUDE_NAMES := \
    viewer/Camera.h \
    viewer/ClipBox.h \
    viewer/Font.h \
    viewer/RenderModules.h \
    viewer/Viewer.h \
#
# Used for "install" target only
LIBVIEWER_PUBLIC_INCLUDE_NAMES := \
    viewer/Viewer.h \
#
LIBVIEWER_SRC_NAMES := \
    viewer/Camera.cc \
    viewer/ClipBox.cc \
    viewer/Font.cc \
    viewer/RenderModules.cc \
    viewer/Viewer.cc \
#
ifdef MBSD
    LIBVIEWER_FLAGS := -framework Cocoa -framework OpenGL -framework IOKit
else
    LIBVIEWER_FLAGS := -lGL -lGLU
endif


CMD_INCLUDE_NAMES := \
#

CMD_SRC_NAMES := \
    cmd/openvdb_print/main.cc \
#
RENDER_SRC_NAMES := \
    cmd/openvdb_render/main.cc \
#
VIEW_SRC_NAMES := \
    cmd/openvdb_view/main.cc \
#


PYTHON_INCLUDE_NAMES := \
    python/pyopenvdb.h \
    python/pyutil.h \
    python/pyAccessor.h \
    python/pyGrid.h \
#
# Used for "install" target only
PYTHON_PUBLIC_INCLUDE_NAMES := \
    python/pyopenvdb.h \
#
PYTHON_SRC_NAMES := \
    python/pyFloatGrid.cc \
    python/pyIntGrid.cc \
    python/pyMetadata.cc \
    python/pyOpenVDBModule.cc \
    python/pyTransform.cc \
    python/pyVec3Grid.cc \
#
PYCXXFLAGS := $(CXXISYSTEM) python $(CXXISYSTEM) $(PYTHON_INCL_DIR) $(CXXISYSTEM) $(PYCONFIG_INCL_DIR)
ifneq ($(strip $(NUMPY_INCL_DIR)),)
PYCXXFLAGS += $(CXXISYSTEM) $(NUMPY_INCL_DIR) -DPY_OPENVDB_USE_NUMPY
endif
ifneq (no,$(strip $(PYTHON_WRAP_ALL_GRID_TYPES)))
PYCXXFLAGS += -DPY_OPENVDB_WRAP_ALL_GRID_TYPES
endif


HEADER_SUBDIRS := $(dir $(INCLUDE_NAMES))

ifdef WINDOWS_NT
GETOPT_INCLUDE := port/getopt.c
else
GETOPT_INCLUDE :=
endif

ALL_INCLUDE_FILES := \
    $(INCLUDE_NAMES) \
    $(UNITTEST_INCLUDE_NAMES) \
    $(CMD_INCLUDE_NAMES) \
    $(LIBVIEWER_INCLUDE_NAMES) \
    $(GETOPT_INCLUDE) \
    $(PYTHON_INCLUDE_NAMES) \
#
SRC_FILES := \
    $(SRC_NAMES) \
    $(UNITTEST_SRC_NAMES) \
    $(CMD_SRC_NAMES) \
    $(RENDER_SRC_NAMES) \
    $(VIEW_SRC_NAMES) \
    $(LIBVIEWER_SRC_NAMES) \
    $(PYTHON_SRC_NAMES) \
#
ALL_SRC_FILES := $(SRC_FILES)

OBJ_NAMES := $(SRC_NAMES:.cc=.o)
CMD_OBJ_NAMES := $(CMD_SRC_NAMES:.cc=.o)
RENDER_OBJ_NAMES := $(RENDER_SRC_NAMES:.cc=.o)
VIEW_OBJ_NAMES := $(VIEW_SRC_NAMES:.cc=.o)
UNITTEST_OBJ_NAMES := $(UNITTEST_SRC_NAMES:.cc=.o)
LIBVIEWER_OBJ_NAMES := $(LIBVIEWER_SRC_NAMES:.cc=.o)
PYTHON_OBJ_NAMES := $(PYTHON_SRC_NAMES:.cc=.o)

LIB_MAJOR_VERSION=$(shell grep 'define OPENVDB_LIBRARY_MAJOR_VERSION_NUMBER ' \
    version.h | sed 's/[^0-9]*//g')
LIB_MINOR_VERSION=$(shell grep 'define OPENVDB_LIBRARY_MINOR_VERSION_NUMBER ' \
    version.h | sed 's/[^0-9]*//g')
LIB_PATCH_VERSION=$(shell grep 'define OPENVDB_LIBRARY_PATCH_VERSION_NUMBER ' \
    version.h | sed 's/[^0-9]*//g')

LIB_VERSION=$(LIB_MAJOR_VERSION).$(LIB_MINOR_VERSION).$(LIB_PATCH_VERSION)
SO_VERSION=$(LIB_MAJOR_VERSION).$(LIB_MINOR_VERSION)

LIBOPENVDB_NAME=openvdb_sesi
LIBOPENVDB_STATIC := $(STATICLIB_PREFIX)$(LIBOPENVDB_NAME)$(STATICLIB_SUFFIX)
LIBOPENVDB_SHARED := $(SHAREDLIB_PREFIX)$(LIBOPENVDB_NAME)$(SHAREDLIB_SUFFIX)
LIBOPENVDB_SHARED_VERSIONED := $(LIBOPENVDB_SHARED).$(LIB_VERSION)
LIBOPENVDB_SONAME := $(LIBOPENVDB_SHARED).$(SO_VERSION)
LIBOPENVDB_IMPORT := $(IMPORTLIB_PREFIX)$(LIBOPENVDB_NAME)$(IMPORTLIB_SUFFIX)
ifdef LINUX
LIBOPENVDB_SONAME_FLAGS := -Wl,-soname,$(LIBOPENVDB_SONAME)
else
LIBOPENVDB_SONAME_FLAGS :=
endif

PYTHON_MODULE_NAME=pyopenvdb
PYTHON_MODULE := $(SHAREDLIB_PREFIX)$(PYTHON_MODULE_NAME)$(SHAREDLIB_SUFFIX)
PYTHON_SONAME := $(PYTHON_MODULE).$(SO_VERSION)
ifdef LINUX
PYTHON_SONAME_FLAGS := -Wl,-soname,$(PYTHON_SONAME)
else
PYTHON_SONAME_FLAGS :=
endif

ifeq (no,$(strip $(shared)))
    LIBOPENVDB := $(LIBOPENVDB_STATIC)
    LIBOPENVDB_IMPORT := $(LIBOPENVDB_STATIC)
else
  ifeq (yes,$(strip $(VERSIONED_LIBS)))
    LIBOPENVDB := $(LIBOPENVDB_SHARED_VERSIONED)
  else
    LIBOPENVDB := $(LIBOPENVDB_SHARED)
  endif
  LIBOPENVDB_RPATH := -Wl,-rpath,$(DESTDIR)/lib $(LIBOPENVDB)
endif # shared

CPPUNIT_RPATH := -Wl,-rpath,$(CPPUNIT_LIB_DIR) $(LDDIROPT)$(CPPUNIT_LIB_DIR) $(CPPUNIT_LIB)

ifdef WINDOWS_NT
    LIBS_RPATH := $(subst -L ,$(LDDIROPT),$(LIBS))
    LIBOPENVDB_RPATH := $(LIBOPENVDB_IMPORT)
    CPPUNIT_RPATH := $(LDDIROPT)$(CPPUNIT_LIB_DIR) $(CPPUNIT_LIB)
    EXR_LIBS_RPATH := $(subst -L ,$(LDDIROPT),$(EXR_LIBS))
    ALLOC_LIBS_RPATH := $(subst -L ,$(LDDIROPT),$(ALLOC_LIBS))
    PYTHON_LIBS_RPATH := \
	$(LDDIROPT)$(PYTHON_LIB_DIR) $(PYTHON_LIB) \
	$(LDDIROPT)$(BOOST_PYTHON_LIB_DIR) $(BOOST_PYTHON_LIB)
endif

DEPEND := dependencies

ALL_PRODUCTS := \
    $(LIBOPENVDB_STATIC) \
    $(LIBOPENVDB_SHARED) \
    $(LIBOPENVDB_SHARED_VERSIONED) \
    $(LIBOPENVDB_IMPORT) \
    vdb_test$(EXE_SUFFIX) \
    vdb_test$(EXE_SUFFIX).manifest \
    vdb_test.{pdb,ilk} \
    vdb_print$(EXE_SUFFIX) \
    vdb_print$(EXE_SUFFIX).manifest \
    vdb_print.{pdb,ilk} \
    vdb_view$(EXE_SUFFIX) \
    vdb_view$(EXE_SUFFIX).manifest \
    vdb_view.{pdb,ilk} \
    vdb_render$(EXE_SUFFIX) \
    vdb_render$(EXE_SUFFIX).manifest \
    vdb_render.{pdb,ilk} \
    $(DEPEND) \
    $(PYTHON_MODULE) \
#

.SUFFIXES: .o .cc

.PHONY: all clean depend doc install lib pdfdoc pydoc pytest python test \
        sesi_archive install_headers

.cc.o:
	$(call CompileCXX,)

all: lib python vdb_print vdb_render vdb_test

$(OBJ_NAMES): %.o: %.cc
	$(call CompileCXX,-DOPENVDB_PRIVATE)

ifneq (no,$(strip $(shared)))

# Build shared library
lib: $(LIBOPENVDB_SHARED)

ifeq (yes,$(strip $(VERSIONED_LIBS)))
$(LIBOPENVDB_SHARED_VERSIONED): $(OBJ_NAMES)
	$(call BuildSharedLibrary,$(LIBS_RPATH) $(LIBOPENVDB_SONAME_FLAGS))
$(LIBOPENVDB_SONAME): $(LIBOPENVDB_SHARED_VERSIONED)
	$(LN) $< $@
$(LIBOPENVDB_SHARED): $(LIBOPENVDB_SHARED_VERSIONED) $(LIBOPENVDB_SONAME)
	$(LN) $(LIBOPENVDB_SHARED_VERSIONED) $@
else
$(LIBOPENVDB_SHARED): $(OBJ_NAMES)
	$(call BuildSharedLibrary,$(LIBS_RPATH))
endif # versioned libs

else

# Build static library
lib: $(LIBOPENVDB_STATIC)

$(LIBOPENVDB_STATIC): $(OBJ_NAMES)
	@echo "Building $@ because of $(list_deps)"
	$(AR) $(ARFLAGS) $(AROUTPUT)$@ $^

endif # shared

# BEGIN doc RULES
$(DOC_INDEX): doxygen-config $(INCLUDE_NAMES) $(SRC_NAMES) $(DOC_FILES)
	@echo "Generating documentation because of $(list_deps)"
	echo 'OUTPUT_DIRECTORY=./doc' | cat doxygen-config - | $(DOXYGEN) - $(QUIET)

$(DOC_PDF): doxygen-config $(INCLUDE_NAMES) $(SRC_NAMES) $(DOC_FILES)
	@echo "Generating documentation because of $(list_deps)"
	echo -e 'OUTPUT_DIRECTORY=./doc\nGENERATE_LATEX=YES\nGENERATE_HTML=NO' \
	    | cat doxygen-config - | $(DOXYGEN) - $(QUIET) \
	    && cd ./doc/latex && make refman.pdf $(QUIET) \
	    && echo 'Created doc/latex/refman.pdf'

ifneq (,$(strip $(DOXYGEN)))
doc: $(DOC_INDEX)
pdfdoc: $(DOC_PDF)
else
doc:
	@echo "$@"': $$DOXYGEN is undefined'
pdfdoc:
	@echo "$@"': $$DOXYGEN is undefined'
endif
# END doc RULES

# BEGIN vdb_print rules
vdb_print$(EXE_SUFFIX): $(LIBOPENVDB) $(CMD_OBJ_NAMES)
	$(call BuildExecutable, $(CMD_OBJ_NAMES) \
                                $(LIBS_RPATH) $(LIBOPENVDB_RPATH) \
				$(ALLOC_LIBS_RPATH))

ifneq (,$(strip $(EXE_SUFFIX)))
vdb_print: vdb_print$(EXE_SUFFIX)
endif
# END vdb_print rules

# BEGIN vdb_render rules
$(RENDER_OBJ_NAMES): %.o: %.cc
	$(call CompileCXX, $(CXXISYSTEM) $(EXR_INCL_DIR))

vdb_render$(EXE_SUFFIX): $(LIBOPENVDB) $(RENDER_OBJ_NAMES)
	$(call BuildExecutable, $(RENDER_OBJ_NAMES) \
                                $(LIBS_RPATH) \
                                $(EXR_LIBS_RPATH) \
				$(LIBOPENVDB_RPATH) \
				$(ALLOC_LIBS_RPATH))

ifneq (,$(strip $(EXE_SUFFIX)))
vdb_render: vdb_render$(EXE_SUFFIX)
endif
# END vdb_render rules

# BEGIN vdb_view rules
ifneq (yes,$(has_glfw))
vdb_view:
	@echo "$@"': GLFW is unavailable'
else
# Create an openvdb_viewer/ symlink to the viewer/ subdirectory,
# to mirror the DWA directory structure.
openvdb_viewer:
	$(LN) viewer openvdb_viewer
$(LIBVIEWER_INCLUDE_NAMES): openvdb_viewer

$(LIBVIEWER_OBJ_NAMES): $(LIBVIEWER_INCLUDE_NAMES)
$(LIBVIEWER_OBJ_NAMES): %.o: %.cc
	$(call CompileCXX,$(CXXISYSTEM) $(GLFW_INCL_DIR) -DGL_GLEXT_PROTOTYPES=1)

ifneq (,$(strip $(EXE_SUFFIX)))
vdb_view: vdb_view$(EXE_SUFFIX)
endif

vdb_view$(EXE_SUFFIX): $(LIBOPENVDB) $(LIBVIEWER_OBJ_NAMES) $(VIEW_OBJ_NAMES)
	$(call BuildExecutable, $(VIEW_OBJ_NAMES) $(LIBVIEWER_OBJ_NAMES) \
		$(GLFW_RPATH) \
		$(LIBVIEWER_FLAGS) $(LIBS_RPATH) $(BOOST_THREAD_LIB) \
		$(LIBOPENVDB_RPATH) \
		$(ALLOC_LIBS_RPATH))
endif
# END vdb_view RULES

# Build the Python module
$(PYTHON_OBJ_NAMES): $(PYTHON_INCLUDE_NAMES)
$(PYTHON_OBJ_NAMES): %.o: %.cc
	$(call CompileCXX, -I . $(PYCXXFLAGS))
$(PYTHON_MODULE): $(LIBOPENVDB) $(PYTHON_OBJ_NAMES)
	$(call BuildSharedLibrary, $(PYTHON_SONAME_FLAGS) $(PYTHON_OBJ_NAMES) \
		$(LIBS_RPATH) $(CONCURRENT_MALLOC_LIB) $(PYTHON_LIBS_RPATH))

ifeq (yes,$(has_python))
ifneq (,$(strip $(EPYDOC)))
pydoc: $(PYTHON_MODULE) $(LIBOPENVDB_SONAME)
	@echo "Generating Python module documentation because of $(list_deps)"
	pydocdir=doc/html/python; \
	/bin/mkdir -p $${pydocdir}; \
	echo "Created $${pydocdir}"; \
	export LD_LIBRARY_PATH=${LD_LIBRARY_PATH}:$(CURDIR); \
	export PYTHONPATH=${PYTHONPATH}:$(CURDIR); \
	    $(EPYDOC) --html -o $${pydocdir} $(PYTHON_MODULE_NAME) $(QUIET)
else
pydoc:
	@echo "$@"': $$EPYDOC is undefined'
endif

pytest: $(PYTHON_MODULE) $(LIBOPENVDB_SONAME)
	@echo "Testing Python module $(PYTHON_MODULE)"
	export LD_LIBRARY_PATH=${LD_LIBRARY_PATH}:$(CURDIR); \
	export PYTHONPATH=${PYTHONPATH}:$(CURDIR); \
	    python$(PYTHON_VERSION) ./python/test/TestOpenVDB.py $(QUIET_TEST)

python: $(PYTHON_MODULE)
else
python pytest pydoc:
	@echo "$@"': Python is unavailable'
endif

# BEGIN vdb_test RULES
ifeq (3,$(words $(strip $(CPPUNIT_LIB_DIR) $(CPPUNIT_INCL_DIR) $(CPPUNIT_LIB))))

$(UNITTEST_OBJ_NAMES): %.o: %.cc
	$(call CompileCXX, $(CXXISYSTEM) $(CPPUNIT_INCL_DIR))

vdb_test$(EXE_SUFFIX): $(LIBOPENVDB) $(UNITTEST_OBJ_NAMES)
	$(call BuildExecutable, $(UNITTEST_OBJ_NAMES) \
		$(CPPUNIT_RPATH) \
		$(LIBS_RPATH) $(LIBOPENVDB_RPATH) \
		$(ALLOC_LIBS_RPATH))

ifneq (,$(strip $(EXE_SUFFIX)))
vdb_test: vdb_test$(EXE_SUFFIX)
endif

ifdef MBSD
  APP_EXPORTS := export DYLD_LIBRARY_PATH=${LD_LIBRARY_PATH}:$(LOCAL_LIB):$(CURDIR)
else
  ifdef WINDOWS_NT
    APP_EXPORTS := true
  else
    APP_EXPORTS := export LD_LIBRARY_PATH=${LD_LIBRARY_PATH}:$(CURDIR)
  endif
endif

test: vdb_test
	@echo "Testing $(LIBOPENVDB_NAME)"
	$(APP_EXPORTS); ./vdb_test $(QUIET_TEST)
else
vdb_test:
	@echo "$@"': $$(CPPUNIT_INCL_DIR) is undefined'
test:
	@echo "$@"': $$(CPPUNIT_INCL_DIR) is undefined'
endif
# END vdb_test RULES

install_headers: $(INCLUDE_NAMES)
	/bin/mkdir -p $(DESTDIR)/include/openvdb
	@echo "Created $(DESTDIR)/include/openvdb"
	pushd $(DESTDIR)/include/openvdb > /dev/null; \
	    /bin/mkdir -p $(HEADER_SUBDIRS); popd > /dev/null
	for f in $(INCLUDE_NAMES); \
	    do /bin/cp -f $$f $(DESTDIR)/include/openvdb/$$f; done
	@#
	@echo "Copied header files to $(DESTDIR)/include"

install: install_headers lib python vdb_print vdb_render vdb_view doc pydoc
	mkdir -p $(DESTDIR)/lib
	@echo "Created $(DESTDIR)/lib/"
	cp -f $(LIBOPENVDB) $(LIBOPENVDB_IMPORT) $(DESTDIR)/lib
	pushd $(DESTDIR)/lib > /dev/null; \
	    if [ -f $(LIBOPENVDB_SHARED_VERSIONED) ]; then \
		    $(LN) $(LIBOPENVDB_SHARED_VERSIONED) $(LIBOPENVDB_SHARED); fi; \
	    popd > /dev/null
	@echo "Copied libopenvdb to $(DESTDIR)/lib/"
	@#
	if [ -f $(PYTHON_MODULE) ]; \
	then \
	    installdir=$(DESTDIR)/python/include/python$(PYTHON_VERSION); \
	    mkdir -p $${installdir}; \
	    echo "Created $${installdir}"; \
	    /bin/cp -f $(PYTHON_PUBLIC_INCLUDE_NAMES) $${installdir}/; \
	    echo "Copied Python header files to $${installdir}"; \
	    installdir=$(DESTDIR)/python/lib/python$(PYTHON_VERSION); \
	    mkdir -p $${installdir}; \
	    echo "Created $${installdir}"; \
	    /bin/cp -f $(PYTHON_MODULE) $${installdir}/; \
	    pushd $${installdir} > /dev/null; \
	    $(LN) $(PYTHON_MODULE) $(PYTHON_SONAME); \
	    popd > /dev/null; \
	    echo "Copied Python module to $${installdir}"; \
	fi
	@#
	mkdir -p $(DESTDIR)/bin
	@echo "Created $(DESTDIR)/bin/"
	cp -f vdb_print$(EXE_SUFFIX) $(DESTDIR)/bin
	@echo "Copied vdb_print$(EXE_SUFFIX) to $(DESTDIR)/bin/"
	cp -f vdb_render$(EXE_SUFFIX) $(DESTDIR)/bin
	@echo "Copied vdb_render to $(DESTDIR)/bin/"
	if [ -f vdb_view$(EXE_SUFFIX) ]; \
	then \
		cp -f vdb_view$(EXE_SUFFIX) $(DESTDIR)/bin; \
		echo "Copied vdb_view to $(DESTDIR)/bin/"; \
	fi
	@#
	if [ -d doc/html ]; \
	then \
	    mkdir -p $(DESTDIR)/share/doc/openvdb; \
	    echo "Created $(DESTDIR)/share/doc/openvdb/"; \
	    cp -r -f doc/html $(DESTDIR)/share/doc/openvdb; \
	    echo "Copied documentation to $(DESTDIR)/share/doc/openvdb/"; \
	fi

SESI_ARCHIVE_CONTENTS := \
    CHANGES \
    COPYRIGHT \
    INSTALL \
    LICENSE \
    README \
    Makefile \
    clmakedep.py \
    platform.mk \
    $(ALL_INCLUDE_FILES) \
    $(ALL_SRC_FILES) \
    $(DOC_FILES) \
    doxygen-config \
#
SESI_ARCHIVE_CONTENTS := $(addprefix openvdb/,$(SESI_ARCHIVE_CONTENTS))
SESI_ARCHIVE_NAME := openvdb-$(LIB_MAJOR_VERSION).$(LIB_MINOR_VERSION).$(LIB_PATCH_VERSION).tar.gz

sesi_archive:
	@echo "Creating $(SESI_ARCHIVE_NAME)"
	$(RM) $(SESI_ARCHIVE_NAME)
	$(LN) . openvdb
	/bin/tar cfz $(SESI_ARCHIVE_NAME) $(SESI_ARCHIVE_CONTENTS) || true
	$(RM) openvdb

ifdef WINDOWS_NT
HFS_DSOLIB := $(HFS)/bin
else
HFS_DSOLIB := $(HFS)/dsolib
endif

# Directly installs into $HFS for development builds
hfsinstall: $(LIBOPENVDB_SHARED)
	/bin/cp $(basename $(LIBOPENVDB_SHARED)).* $(HFS_DSOLIB)
hfsclean:
	$(RM) $(HFS_DSOLIB)/$(basename $(LIBOPENVDB_SHARED)).*

DEPFILES = $(call DepFromSrc,$(ALL_SRC_FILES))

# TODO: This accumulates all source file dependencies into a single file
# containing a rule for each *.o file.  Consider generating a separate
# dependency file for each *.o file instead.
$(DEPEND): $(ALL_INCLUDE_FILES) $(ALL_SRC_FILES)
	@echo "Generating dependencies because of $(list_deps)"
	$(RM) $(DEPEND)
	for f in $(SRC_NAMES) $(LIBVIEWER_SRC_NAMES) $(CMD_SRC_NAMES) $(RENDER_SRC_NAMES) $(VIEW_SRC_NAMES); \
		do $(CXX) $(CXXFLAGS) -O0 \
			-MM $$f -MT `echo $$f | sed 's%\.[^.]*%.o%'` >> $(DEPEND); \
	done
	if [ -d "$(CPPUNIT_INCL_DIR)" ]; \
	then \
	    for f in $(UNITTEST_SRC_NAMES); \
	        do $(CXX) $(CXXFLAGS) -O0 \
	            -MM $$f -MT `echo $$f | sed 's%\.[^.]*%.o%'` \
	            $(CXXISYSTEM) $(CPPUNIT_INCL_DIR) >> $(DEPEND); \
	    done; \
	fi

depend: $(DEPEND)

clean_obj:
	$(RM) $(OBJ_NAMES)
	$(RM) $(PYTHON_OBJ_NAMES)
	$(RM) $(UNITTEST_OBJ_NAMES)
	$(RM) $(CMD_OBJ_NAMES)
	$(RM) $(LIBVIEWER_OBJ_NAMES)
	$(RM) $(VIEW_OBJ_NAMES) 
	$(RM) $(RENDER_OBJ_NAMES) 

clean: clean_obj
	$(RM) $(ALL_PRODUCTS) $(DEPFILES)
	$(RM) $(LIBOPENVDB_STATIC)
	$(RM) $(LIBOPENVDB_SHARED_VERSIONED)
	$(RM) $(LIBOPENVDB_SHARED)
	$(RM) $(LIBOPENVDB_NAME).def
	$(RM) $(LIBOPENVDB_IMPORT)
	$(RM) *.lib *.exp *.ilk *.obj *.manifest *.pdb
	$(RM) -r ./doc/html
	$(RM) -r ./doc/html ./doc/latex

ifneq ($(DEPFILES),)
-include $(DEPFILES)
endif

# Copyright (c) 2012-2016 DreamWorks Animation LLC
# All rights reserved. This software is distributed under the
# Mozilla Public License 2.0 ( http://www.mozilla.org/MPL/2.0/ )
