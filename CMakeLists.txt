project(pvr.mythtv)

cmake_minimum_required(VERSION 2.6)

set(CMAKE_MODULE_PATH ${CMAKE_MODULE_PATH} ${PROJECT_SOURCE_DIR})

enable_language(CXX)
enable_language(C)

find_package(kodi REQUIRED)
find_package(kodiplatform REQUIRED)

include_directories(${kodiplatform_INCLUDE_DIRS}
                    ${KODI_INCLUDE_DIR}
                    ${PROJECT_SOURCE_DIR}/lib/cppmyth/jansson/src
                    ${PROJECT_SOURCE_DIR}/lib/cppmyth/cppmyth/src
                    ${CMAKE_BINARY_DIR}/lib/cppmyth/jansson/include)

add_definitions(-DUSE_DEMUX -D__STDC_FORMAT_MACROS)

if(NOT WIN32)
  add_options(ALL_LANGUAGES ALL_BUILDS "-fPIC")
endif()

set(MYTHTV_SOURCES src/avinfo.cpp
                   src/categories.cpp
                   src/demuxer/ES_AAC.cpp
                   src/demuxer/ES_MPEGAudio.cpp
                   src/demuxer/bitstream.cpp
                   src/demuxer/ES_Teletext.cpp
                   src/demuxer/debug.cpp
                   src/demuxer/ES_Subtitle.cpp
                   src/demuxer/ES_h264.cpp
                   src/demuxer/ES_AC3.cpp
                   src/demuxer/elementaryStream.cpp
                   src/demuxer/ES_MPEGVideo.cpp
                   src/demuxer/tsDemuxer.cpp
                   src/demux.cpp
                   src/guidialogbase.cpp
                   src/guidialogyesno.cpp
                   src/pvrclient-mythtv.cpp
                   src/cppmyth/MythScheduleManager.cpp
                   src/cppmyth/MythEPGInfo.cpp
                   src/cppmyth/MythRecordingRule.cpp
                   src/cppmyth/MythProgramInfo.cpp
                   src/cppmyth/MythChannel.cpp
                   src/client.cpp
                   src/fileOps.cpp)

set(BUILD_DOCS OFF CACHE BOOLEAN "")
set(BUILD_SHARED_LIBS OFF)
add_subdirectory(lib/cppmyth/cppmyth)
add_subdirectory(lib/cppmyth/jansson)

set(DEPLIBS ${kodiplatform_LIBRARIES} cppmyth jansson)

unset(__link_flags)
# Check if the linker supports version script (i.e. is GNU ld)
include(TestCXXAcceptsFlag)
if(CMAKE_COMPILER_IS_GNUCXX)
  set(__symbols "{ local: *; };")
  set(__version_script "${CMAKE_CURRENT_BINARY_DIR}/tmp.ver")
  file(WRITE ${__version_script} "${__symbols}\n")
  check_cxx_accepts_flag("-Wl,--version-script=${__version_script}" LD_ACCEPTS_VERSION_SCRIPT)
  if(LD_ACCEPTS_VERSION_SCRIPT)
    set(__link_flags "-Wl,--version-script='${PROJECT_SOURCE_DIR}/src/addon.map'")
  endif()
endif()

if(__link_flags)
  add_options(ALL_LANGUAGES ALL_BUILDS "${__link_flags}")
endif()

set(BUILD_SHARED_LIBS ON)
build_addon(pvr.mythtv MYTHTV DEPLIBS)

include(CPack)
