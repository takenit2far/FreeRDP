# WinPR: Windows Portable Runtime
# libwinpr-sspi cmake build script
#
# Copyright 2011 Marc-Andre Moreau <marcandre.moreau@gmail.com>
#
# Licensed under the Apache License, Version 2.0 (the "License");
# you may not use this file except in compliance with the License.
# You may obtain a copy of the License at
#
#     http://www.apache.org/licenses/LICENSE-2.0
#
# Unless required by applicable law or agreed to in writing, software
# distributed under the License is distributed on an "AS IS" BASIS,
# WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
# See the License for the specific language governing permissions and
# limitations under the License.

set(MODULE_NAME "winpr-sspi")
set(MODULE_PREFIX "WINPR_SSPI")

set(${MODULE_PREFIX}_NTLM_SRCS
	NTLM/ntlm_av_pairs.c
	NTLM/ntlm_av_pairs.h
	NTLM/ntlm_compute.c
	NTLM/ntlm_compute.h
	NTLM/ntlm_message.c
	NTLM/ntlm_message.h
	NTLM/ntlm.c
	NTLM/ntlm.h)

set(${MODULE_PREFIX}_NEGOTIATE_SRCS
	Negotiate/negotiate.c
	Negotiate/negotiate.h)

set(${MODULE_PREFIX}_SCHANNEL_SRCS
	Schannel/schannel_openssl.c
	Schannel/schannel_openssl.h
	Schannel/schannel.c
	Schannel/schannel.h)

set(${MODULE_PREFIX}_CREDSSP_SRCS
	CredSSP/credssp.c)

set(${MODULE_PREFIX}_SRCS
	${${MODULE_PREFIX}_CREDSSP_SRCS}
	sspi_winpr.c
	sspi_winpr.h
	sspi_export.c
	sspi.c
	sspi.h)

set(${MODULE_PREFIX}_SRCS
	${${MODULE_PREFIX}_NTLM_SRCS}
	${${MODULE_PREFIX}_KERBEROS_SRCS}
	${${MODULE_PREFIX}_NEGOTIATE_SRCS}
	${${MODULE_PREFIX}_SCHANNEL_SRCS}
	${${MODULE_PREFIX}_SRCS})

if(MSVC AND (NOT MONOLITHIC_BUILD))
	#set(${MODULE_PREFIX}_SRCS ${${MODULE_PREFIX}_SRCS} module.def)
endif()

add_complex_library(MODULE ${MODULE_NAME} TYPE "OBJECT"
	MONOLITHIC ${MONOLITHIC_BUILD}
	SOURCES ${${MODULE_PREFIX}_SRCS}
	EXPORT)

include_directories(${ZLIB_INCLUDE_DIRS})
include_directories(${OPENSSL_INCLUDE_DIR})

set_target_properties(${MODULE_NAME} PROPERTIES VERSION ${WINPR_VERSION_FULL} SOVERSION ${WINPR_VERSION} PREFIX "lib")

set(${MODULE_PREFIX}_LIBS
	${ZLIB_LIBRARIES}
	${OPENSSL_LIBRARIES})

if(WIN32)
	set(${MODULE_PREFIX}_LIBS ${${MODULE_PREFIX}_LIBS} ws2_32)
endif()

set_complex_link_libraries(VARIABLE ${MODULE_PREFIX}_LIBS
	MONOLITHIC ${MONOLITHIC_BUILD} INTERNAL
	MODULE winpr
	MODULES winpr-crt winpr-sysinfo winpr-registry winpr-crypto winpr-library winpr-utils)

if(MONOLITHIC_BUILD)
	set(WINPR_LIBS ${WINPR_LIBS} ${${MODULE_PREFIX}_LIBS} PARENT_SCOPE)
else()
	target_link_libraries(${MODULE_NAME} ${${MODULE_PREFIX}_LIBS})
	install(TARGETS ${MODULE_NAME} DESTINATION ${CMAKE_INSTALL_LIBDIR} EXPORT WinPRTargets)
endif()

set_property(TARGET ${MODULE_NAME} PROPERTY FOLDER "WinPR")

if(BUILD_TESTING)
	add_subdirectory(test)
endif()
