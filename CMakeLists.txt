cmake_minimum_required(VERSION 2.6)

PROJECT(netifd C)

IF(NOT ${CMAKE_VERSION} LESS 3.0)
  include(CheckCCompilerFlag)
  check_c_compiler_flag(-Wimplicit-fallthrough HAS_IMPLICIT_FALLTHROUGH)
ENDIF()

ADD_DEFINITIONS(-Os --std=gnu99 -Wmissing-declarations -Wno-unknown-warning-option -Wno-format-truncation)

IF(HAS_IMPLICIT_FALLTHROUGH)
  ADD_DEFINITIONS(-Wimplicit-fallthrough)
ENDIF()

SET(CMAKE_SHARED_LIBRARY_LINK_C_FLAGS "")

SET(SOURCES
	main.c utils.c system.c tunnel.c handler.c
	interface.c interface-ip.c interface-event.c
	iprule.c proto.c proto-static.c proto-shell.c
	config.c device.c bridge.c bond.c veth.c vlan.c alias.c
	macvlan.c ubus.c vlandev.c wireless.c
	extdev.c)


SET(LIBS
	ubox ubus uci json-c blobmsg_json)

INCLUDE(FindPkgConfig)
PKG_SEARCH_MODULE(LIBNL libnl-3.0 linl-3 linl nl-3 nl)
INCLUDE_DIRECTORIES(${LIBNL_INCLUDE_DIRS})

IF("${CMAKE_SYSTEM_NAME}" MATCHES "Linux" AND NOT DUMMY_MODE)
	SET(SOURCES ${SOURCES} system-linux.c)
	SET(LIBS ${LIBS} ${LIBNL_LIBRARIES})
ELSE()
	ADD_DEFINITIONS(-DDUMMY_MODE=1)
	SET(SOURCES ${SOURCES} system-dummy.c)
ENDIF()

IF(DEBUG)
  ADD_DEFINITIONS(-DDEBUG -g3)
  IF(NO_OPTIMIZE)
    ADD_DEFINITIONS(-O0)
  ENDIF()
ENDIF()


ADD_EXECUTABLE(netifd ${SOURCES})

TARGET_LINK_LIBRARIES(netifd ${LIBS})

INSTALL(TARGETS netifd
	RUNTIME DESTINATION sbin
)
