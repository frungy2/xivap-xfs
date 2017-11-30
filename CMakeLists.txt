cmake_minimum_required(VERSION 3.0)

project(xivap LANGUAGES C CXX)
set(XPSDK_DIR "${CMAKE_CURRENT_SOURCE_DIR}/includes/XPSDK")
set(CMAKE_MODULE_PATH "${CMAKE_CURRENT_SOURCE_DIR}/libxplanemp/cmake/Modules" ${CMAKE_MODULE_PATH})

set(CMAKE_BINARY_DIR ${CMAKE_SOURCE_DIR}-output/build/)
set(EXECUTABLE_OUTPUT_PATH ${CMAKE_BINARY_DIR})
set(LIBRARY_OUTPUT_PATH ${CMAKE_BINARY_DIR})
set(CMAKE_BUILD_WITH_INSTALL_RPATH TRUE)
SET(CMAKE_INSTALL_RPATH "$ORIGIN")

find_package(PNG REQUIRED)
find_package(XPSDK REQUIRED)
include(CMakeDependentOption)

add_library(fmod SHARED IMPORTED)

set(XIVAP_EXTRA_LINK_FLAGS "")
set(XIVAP_PLATFORM_SOURCES "")
set(XIVAP_LEGACY_NAME "")
if(CMAKE_SYSTEM_NAME MATCHES "Linux")
	set(XIVAP_INCLUDE_DIRS ${XIVAP_INCLUDE_DIRS} includes/FMOD/FMODLinux/api/inc)
	set_property(TARGET fmod PROPERTY IMPORTED_LOCATION ${CMAKE_SOURCE_DIR}/includes/FMOD/FMODLinux/api/lib/libfmod.so)
	set(XIVAP_DEFINES ${XIVAP_DEFINES} LIN=1 LINUX=1)
	set(XIVAP_PLATFORM_PLUGIN_DIR lin_x64)
	set(CMAKE_SHARED_LINKER_FLAGS ${CMAKE_SHARED_LINKER_FLAGS} "-Wl,--version-script=\"${CMAKE_SOURCE_DIR}/includes/linker.script\"")
	set(XIVAP_LEGACY_NAME "lin")
elseif(CMAKE_SYSTEM_NAME MATCHES "Windows")
	set(XIVAP_INCLUDE_DIRS ${XIVAP_INCLUDE_DIRS} includes/FMOD/FMODWindows/api/inc)
	include_directories(PUBLIC includes/WinIncludes includes/WinIncludes/GL ${PNG_INCLUDE_DIRS})
	set(XIVAP_LINK_LIBRARIES ${XIVAP_LINK_LIBRARIES} ws2_32)
	set(XIVAP_DEFINES ${XIVAP_DEFINES} IBM=1)
	set_property(TARGET fmod PROPERTY IMPORTED_LOCATION ${CMAKE_SOURCE_DIR}/includes/FMOD/FMODWindows/api/lib/fmod64.dll)
	set_property(TARGET fmod PROPERTY IMPORTED_IMPLIB ${CMAKE_SOURCE_DIR}/includes/FMOD/FMODWindows/api/lib/fmod64_vc.lib)
	set(XIVAP_PLATFORM_PLUGIN_DIR win_x64)
	set(XIVAP_EXTRA_LINK_FLAGS -NODEFAULTLIB:libcmt)
	set(XIVAP_PLATFORM_SOURCES ${XIVAP_PLATFORM_SOURCES} src/xivapWin.cpp)
	set(XIVAP_LEGACY_NAME "win")
elseif(CMAKE_SYSTEM_NAME MATCHES "Darwin")
	find_library(FRAMEWORK_FOUNDATION Foundation )
	set(XIVAP_LINK_LIBRARIES ${XIVAP_LINK_LIBRARIES} ${FRAMEWORK_FOUNDATION})
	set(XIVAP_INCLUDE_DIRS ${XIVAP_INCLUDE_DIRS} includes/FMOD/FMODMac/api/inc)
	set_property(TARGET fmod PROPERTY IMPORTED_LOCATION ${CMAKE_SOURCE_DIR}/includes/FMOD/FMODMac/api/lib/libfmod.dylib)
	set(XIVAP_DEFINES ${XIVAP_DEFINES} APL=1 APPLE=1)
	set(XIVAP_PLATFORM_PLUGIN_DIR mac_x64)
	set(XIVAP_LEGACY_NAME "mac")
endif()

add_subdirectory(ptypes)

set(XIVAP_PLUGIN_NAME "X-IvAp")

option(IVAOBUILD "Build for IVAO" OFF)
option(XFSBUILD "Build for XFS" ON)
option(LEGACY_BUILD_MODE "Build without using XPLM300 API" OFF)

CMAKE_DEPENDENT_OPTION(USE_TEAMSPEAK "Build teamspeak support" ON "IVAOBUILD" OFF)

unset(TEAMSPEAK_SRC)

if(IVAOBUILD)
	add_definitions(-DIVAO -DXPMP_CLIENT_NAME="xivap" -DXPMP_CLIENT_LONGNAME="X-IvAp")
endif()
if(USE_TEAMSPEAK)
	set(TEAMSPEAK_SRC src/TeamSpeak.cpp)
	add_definitions(-DUSE_TEAMSPEAK)
endif()

if(XFSBUILD)
	add_definitions(-DXFS -DXPMP_CLIENT_NAME="xfs" -DXPMP_CLIENT_LONGNAME="X-FlightServer")
	set(XIVAP_PLUGIN_NAME "X-FlightServer")
endif()

if(NOT LEGACY_BUILD_MODE)
	add_definitions(-DXPLM300=1)
	set(XIVAP_PLUGIN_FILE_NAME ${XIVAP_PLUGIN_NAME})
else()
	set(XIVAP_PLUGIN_FILE_NAME ${XIVAP_LEGACY_NAME})
	set(XIVAP_PLATFORM_PLUGIN_DIR 64)
endif()

add_subdirectory(libxplanemp)
set_property(TARGET xplanemp PROPERTY POSITION_INDEPENDENT_CODE ON)

add_definitions(-DXPLM200=1 -DXPLM210=1 -DRESOURCES_DIR="${XIVAP_PLUGIN_NAME}" -DX64=1)



add_library(xivap SHARED
	${XIVAP_PLATFORM_SOURCES}
	$<TARGET_OBJECTS:ptypes>
	src/aircraftDB.cpp
	src/airlinesDB.cpp
	src/atcPosition.cpp
	src/awyFinder.cpp
	src/chatWindow.cpp
	src/configFile.cpp
	src/connectForm.cpp
	src/disconnectForm.cpp
	#src/FlightLogger.cpp #Nothing seems to be using this
	src/flightplanForm.cpp
	src/FMCform.cpp
	src/fsdapi.cpp
	src/fsdmessage.cpp
	src/helpers.cpp
	src/HTTPclient.cpp
	src/interpolator.cpp
	src/IVAOsecurity.cpp
	src/messageHandler.cpp
	src/messageWindow.cpp
	src/multiplayer.cpp
	src/multiplayerPilot.cpp
	src/navQuery.cpp
	src/planeposition.cpp
	src/refInspector.cpp
	src/SoundGraphics.cpp
	src/systemcallbacks.cpp
	src/Tcas.cpp
	src/tcpsocket.cpp
	${TEAMSPEAK_SRC}
	src/TLVpacket.cpp
	src/uiWindow.cpp
	src/weatherGod.cpp
	src/weatherPosition.cpp
	src/WristWatch.cpp
	src/xivap.cpp
	${XIVAP_PLATFORM_SOURCES}
	src/XPmessageBox.cpp
	stund/stun.cxx
	stund/StunClient.cpp
	stund/udp.cxx
	widgetLib/XPListBox.cpp
	widgetLib/XPPopups.cpp
)

target_include_directories(xivap
	PUBLIC ${XIVAP_INCLUDE_DIRS}
	${XPSDK_INCLUDE_DIRS}
	stund
	ptypes/include
	libxplanemp/include
	widgetLib
	src
	includes
)

target_link_libraries(xivap
	PRIVATE ${XPSDK_XPLM_LIBRARIES}
	${XPSDK_XPWIDGETS_LIBRARIES}
	PNG::PNG
	${XIVAP_LINK_LIBRARIES}
	xplanemp
	fmod
	${XIVAP_EXTRA_LINK_FLAGS}
)
target_compile_definitions(xivap PRIVATE ${XIVAP_DEFINES})

set_target_properties(xivap PROPERTIES SUFFIX .xpl)
set_target_properties(xivap PROPERTIES PREFIX "")
set_target_properties(xivap PROPERTIES OUTPUT_NAME ${XIVAP_PLUGIN_FILE_NAME})
set_property(TARGET xivap PROPERTY CXX_STANDARD_REQUIRED 11)
set_property(TARGET xivap PROPERTY CXX_STANDARD 14)

add_custom_command(TARGET xivap POST_BUILD
COMMAND ${CMAKE_COMMAND} -E make_directory "${CMAKE_BINARY_DIR}/${XIVAP_PLUGIN_NAME}"
COMMAND ${CMAKE_COMMAND} -E make_directory "${CMAKE_BINARY_DIR}/${XIVAP_PLUGIN_NAME}/${XIVAP_PLATFORM_PLUGIN_DIR}"
COMMAND ${CMAKE_COMMAND} -E copy $<TARGET_FILE:xivap> "${CMAKE_BINARY_DIR}/${XIVAP_PLUGIN_NAME}/${XIVAP_PLATFORM_PLUGIN_DIR}/."
COMMAND ${CMAKE_COMMAND} -E copy_directory "${CMAKE_SOURCE_DIR}/build/X-IvAp Resources/textures" "${CMAKE_BINARY_DIR}/${XIVAP_PLUGIN_NAME}/textures"
COMMAND ${CMAKE_COMMAND} -E copy_directory "${CMAKE_SOURCE_DIR}/build/X-IvAp Resources/sound" "${CMAKE_BINARY_DIR}/${XIVAP_PLUGIN_NAME}/sound"
COMMAND ${CMAKE_COMMAND} -E copy_directory "${CMAKE_SOURCE_DIR}/build/X-IvAp Resources/SidStar" "${CMAKE_BINARY_DIR}/${XIVAP_PLUGIN_NAME}/SidStar"
COMMAND ${CMAKE_COMMAND} -E copy_directory "${CMAKE_SOURCE_DIR}/build/X-IvAp Resources/Flightplans" "${CMAKE_BINARY_DIR}/${XIVAP_PLUGIN_NAME}/flightplans"
COMMAND ${CMAKE_COMMAND} -E make_directory "${CMAKE_BINARY_DIR}/${XIVAP_PLUGIN_NAME}/CSL"
COMMAND ${CMAKE_COMMAND} -E make_directory "${CMAKE_BINARY_DIR}/${XIVAP_PLUGIN_NAME}/CSL8"
COMMAND ${CMAKE_COMMAND} -E copy "${CMAKE_SOURCE_DIR}/build/X-IvAp Resources/aircraft.dat" "${CMAKE_BINARY_DIR}/${XIVAP_PLUGIN_NAME}/."
COMMAND ${CMAKE_COMMAND} -E copy "${CMAKE_SOURCE_DIR}/build/X-IvAp Resources/airlines.dat" "${CMAKE_BINARY_DIR}/${XIVAP_PLUGIN_NAME}/."
COMMAND ${CMAKE_COMMAND} -E copy "${CMAKE_SOURCE_DIR}/build/X-IvAp Resources/airports.dat" "${CMAKE_BINARY_DIR}/${XIVAP_PLUGIN_NAME}/."
COMMAND ${CMAKE_COMMAND} -E copy "${CMAKE_SOURCE_DIR}/build/X-IvAp Resources/changelog.txt" "${CMAKE_BINARY_DIR}/${XIVAP_PLUGIN_NAME}/."
COMMAND ${CMAKE_COMMAND} -E copy "${CMAKE_SOURCE_DIR}/build/X-IvAp Resources/CSL-MODELS-README.txt" "${CMAKE_BINARY_DIR}/${XIVAP_PLUGIN_NAME}/."
COMMAND ${CMAKE_COMMAND} -E copy "${CMAKE_SOURCE_DIR}/build/X-IvAp Resources/Doc8643.txt" "${CMAKE_BINARY_DIR}/${XIVAP_PLUGIN_NAME}/."
COMMAND ${CMAKE_COMMAND} -E copy "${CMAKE_SOURCE_DIR}/build/X-IvAp Resources/icao.dat" "${CMAKE_BINARY_DIR}/${XIVAP_PLUGIN_NAME}/."
COMMAND ${CMAKE_COMMAND} -E copy "${CMAKE_SOURCE_DIR}/build/X-IvAp Resources/lights.png" "${CMAKE_BINARY_DIR}/${XIVAP_PLUGIN_NAME}/."
COMMAND ${CMAKE_COMMAND} -E copy "${CMAKE_SOURCE_DIR}/build/X-IvAp Resources/liveries.txt" "${CMAKE_BINARY_DIR}/${XIVAP_PLUGIN_NAME}/."
COMMAND ${CMAKE_COMMAND} -E copy "${CMAKE_SOURCE_DIR}/build/X-IvAp Resources/mtl.dat" "${CMAKE_BINARY_DIR}/${XIVAP_PLUGIN_NAME}/."
COMMAND ${CMAKE_COMMAND} -E copy "${CMAKE_SOURCE_DIR}/build/X-IvAp Resources/mtlalias.txt" "${CMAKE_BINARY_DIR}/${XIVAP_PLUGIN_NAME}/."
COMMAND ${CMAKE_COMMAND} -E copy "${CMAKE_SOURCE_DIR}/build/X-IvAp Resources/OBJ8ChangesInfo.txt" "${CMAKE_BINARY_DIR}/${XIVAP_PLUGIN_NAME}/."
COMMAND ${CMAKE_COMMAND} -E copy "${CMAKE_SOURCE_DIR}/build/X-IvAp Resources/related.txt" "${CMAKE_BINARY_DIR}/${XIVAP_PLUGIN_NAME}/."
COMMAND ${CMAKE_COMMAND} -E copy "${CMAKE_SOURCE_DIR}/build/X-IvAp Resources/serverlist.txt" "${CMAKE_BINARY_DIR}/${XIVAP_PLUGIN_NAME}/."
)

#Copy the linux libs over
if(CMAKE_SYSTEM_NAME MATCHES "Linux")
add_custom_command(TARGET xivap POST_BUILD
COMMAND ${CMAKE_COMMAND} -E copy "${CMAKE_SOURCE_DIR}/includes/FMOD/FMODLinux/api/lib/libfmod.so.9" "${CMAKE_BINARY_DIR}/${XIVAP_PLUGIN_NAME}/${XIVAP_PLATFORM_PLUGIN_DIR}/libfmod.so.9"
)
endif()

if(LEGACY_BUILD_MODE)
if(CMAKE_SYSTEM_NAME MATCHES "Windows")
add_custom_command(TARGET xivap POST_BUILD
COMMAND ${CMAKE_COMMAND} -E copy "${CMAKE_SOURCE_DIR}/includes/FMOD/FMODWindows/api/lib/fmod64.dll" "${CMAKE_BINARY_DIR}/${XIVAP_PLUGIN_NAME}/${XIVAP_PLATFORM_PLUGIN_DIR}/fmod64.dll"
)
endif()
if(CMAKE_SYSTEM_NAME MATCHES "Darwin")
add_custom_command(TARGET xivap POST_BUILD
COMMAND ${CMAKE_COMMAND} -E copy "${CMAKE_SOURCE_DIR}/includes/FMOD/FMODMac/api/lib/libfmod.dylib" "${CMAKE_BINARY_DIR}/${XIVAP_PLUGIN_NAME}/${XIVAP_PLATFORM_PLUGIN_DIR}/libfmod.dylib"
)
endif()
endif()

#Copy over the teamspeak dll only if required
if(USE_TEAMSPEAK)
add_custom_command(TARGET xivap POST_BUILD
COMMAND ${CMAKE_COMMAND} -E copy "${CMAKE_SOURCE_DIR}/build/X-IvAp Resources/TSRemote.dll" "${CMAKE_BINARY_DIR}/${XIVAP_PLUGIN_NAME}/."
)
endif()

#Copy the correct myservers.txt
if(IVAOBUILD)
add_custom_command(TARGET xivap POST_BUILD
COMMAND ${CMAKE_COMMAND} -E copy "${CMAKE_SOURCE_DIR}/build/X-IvAp Resources/myservers-ivao.txt" "${CMAKE_BINARY_DIR}/${XIVAP_PLUGIN_NAME}/myservers.txt"
)
elseif(XFSBUILD)
add_custom_command(TARGET xivap POST_BUILD
COMMAND ${CMAKE_COMMAND} -E copy "${CMAKE_SOURCE_DIR}/build/X-IvAp Resources/myservers-xfs.txt" "${CMAKE_BINARY_DIR}/${XIVAP_PLUGIN_NAME}/myservers.txt"
)
endif()
