cmake_minimum_required(VERSION 3.0.2)

include(ExternalProject)
set(CMAKE_CXX_STANDARD 14)

project(quicxia)

# XIA codebase
ExternalProject_Add(xia
	GIT_REPOSITORY git@github.com:XIA-Project/xia-core.git
	GIT_TAG xia-v2-overlay
	CONFIGURE_COMMAND ./configure
	BUILD_COMMAND make DEBUG=1
	BUILD_IN_SOURCE 1
	INSTALL_COMMAND ""
	)

# PicoTLS codebase - as a git submodule
add_subdirectory(picotls)
#set(PTLS_HINTS ${CMAKE_BINARY_DIR}/picotls)

# PicoQUIC codebase - as a git submodule
add_subdirectory(picoquic)

# QUIC based Xcache
add_executable(quicxcache xcache.cc)
add_dependencies(quicxcache xia)

set(XIADIR ${CMAKE_SOURCE_DIR}/build/xia-prefix/src/xia)
set(PICOQUICDIR ${CMAKE_SOURCE_DIR}/picoquic)
# XIA includes added to quicxcache
include_directories(quicxcache ${XIADIR}/api/include)
include_directories(quicxcache ${XIADIR}/api/xcache)
include_directories(quicxcache ${PICOQUICDIR}/picoquic)
# XIA libraries added to quicxcache
find_library(DAG_LIBRARY dagaddr HINTS ${XIADIR}/api/lib)
find_library(XCACHE_HEADERS_LIBRARY xcachehdr HINTS ${XIADIR}/api/lib)
find_library(XCACHE_LIBRARY xcache HINTS ${XIADIR}/api/lib)
target_link_libraries(quicxcache
	${DAG_LIBRARY}
	${XCACHE_HEADERS_LIBRARY}
	${XCACHE_LIBRARY}
	picoquic-core
	localconfig
	picoquicxia
	)
