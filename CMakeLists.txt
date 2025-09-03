cmake_minimum_required(VERSION 3.15)

project(Template)

# Require C++20 for std::format and other modern features
set(CMAKE_CXX_STANDARD 20)
set(CMAKE_CXX_STANDARD_REQUIRED ON)
set(CMAKE_CXX_EXTENSIONS OFF)

# Set the path to the Adobe After Effects SDK
set(AE_SDK_PATH "${CMAKE_CURRENT_SOURCE_DIR}/ae")
get_filename_component(AE_SDK_PATH "${AE_SDK_PATH}" ABSOLUTE)

# Organize source files by project structure - auto match
file(GLOB SRC_CPP_FILES ${CMAKE_CURRENT_SOURCE_DIR}/src/*.cpp)
file(GLOB SRC_H_FILES ${CMAKE_CURRENT_SOURCE_DIR}/src/*.h)
set(SRC_FILES ${SRC_CPP_FILES} ${SRC_H_FILES})

set(AE_UTIL_FILES
  ${AE_SDK_PATH}/Util/AEGP_SuiteHandler.cpp
  ${AE_SDK_PATH}/Util/MissingSuiteError.cpp
)

set(PIPL_FILES
  ${CMAKE_CURRENT_SOURCE_DIR}/PiPL/PiPL.r
)

set(SOURCE_FILES ${SRC_FILES} ${AE_UTIL_FILES})

set(AESDK_INCLUDE
	${AE_SDK_PATH}/Headers
	${AE_SDK_PATH}/Headers/SP
	${AE_SDK_PATH}/Util
	${AE_SDK_PATH}/Resources
)

message(STATUS "AE_SDK_PATH: ${AE_SDK_PATH}")

include_directories(
  ${AESDK_INCLUDE}
)

add_compile_definitions(__cplusplus)

# Specify the plugin target
add_library(${PROJECT_NAME} MODULE ${SOURCE_FILES} ${PIPL_FILES})

# Create source groups for Visual Studio (按项目实际结构)
source_group("src" FILES ${SRC_FILES})
source_group("ae\\Util" FILES ${AE_UTIL_FILES})
source_group("PiPL" FILES ${PIPL_FILES})

# If we are generating an xcode project 
if(CMAKE_GENERATOR STREQUAL "Xcode")
	set( CMAKE_OSX_DEPLOYMENT_TARGET "11.0" CACHE STRING "Minimum MacOs Version" )
	set( CMAKE_OSX_ARCHITECTURES "arm64;x86_64" )

  target_link_libraries(${PROJECT_NAME} "-framework Cocoa")

    set_target_properties(${PROJECT_NAME} PROPERTIES
        XCODE_ATTRIBUTE_ENABLE_HARDENED_RUNTIME YES
    )

    find_program( AFX_REZ rez /Developer/Tools )

    add_custom_command(
  		OUTPUT ${CMAKE_CURRENT_BINARY_DIR}/${PROJECT_NAME}.rsrc
  		COMMAND ${AFX_REZ}
  		ARGS ${CMAKE_CURRENT_SOURCE_DIR}/PiPL/PiPL.r
  			-o ${CMAKE_CURRENT_BINARY_DIR}/${PROJECT_NAME}.rsrc
  			-useDF
			${PIPL_INCLUDES}
      	-i ${AE_SDK_PATH}/Headers
      	-i ${AE_SDK_PATH}/Headers/SP
      	-i ${AE_SDK_PATH}/Util
      	-i ${AE_SDK_PATH}/Resources
  			-D __MACH__
  	)

	set_source_files_properties(
		${CMAKE_CURRENT_BINARY_DIR}/${PROJECT_NAME}.rsrc
		PROPERTIES
		MACOSX_PACKAGE_LOCATION Resources/
	)
    
  target_sources( ${PROJECT_NAME} PRIVATE ${CMAKE_CURRENT_BINARY_DIR}/${PROJECT_NAME}.rsrc )

	set_target_properties( ${PROJECT_NAME} PROPERTIES PREFIX "" )
	set_target_properties( ${PROJECT_NAME} PROPERTIES SUFFIX "" )
	string( TIMESTAMP BUILD_YEAR "%Y" )
	set_target_properties( ${PROJECT_NAME} PROPERTIES 
    BUNDLE True
    MACOSX_BUNDLE_BUNDLE_NAME ${PROJECT_NAME}
    BUNDLE_EXTENSION "plugin"
    MACOSX_BUNDLE_INFO_STRING "${PROJECT_NAME}"
    MACOSX_BUNDLE_GUI_IDENTIFIER "com.adobe.AfterEffects.${PROJECT_NAME}"
    MACOSX_BUNDLE_COPYRIGHT "MIT LICENSED ${BUILD_YEAR}."
  )

	# PkgInfo
	add_custom_command(
		OUTPUT ${CMAKE_CURRENT_BINARY_DIR}/PkgInfo
		COMMAND echo "eFKTFXTC" >> ${CMAKE_CURRENT_BINARY_DIR}/PkgInfo
	)

	set_source_files_properties(
		${CMAKE_CURRENT_BINARY_DIR}/PkgInfo
		PROPERTIES
		MACOSX_PACKAGE_LOCATION ""
	)

	target_sources( ${PROJECT_NAME} PRIVATE ${CMAKE_CURRENT_BINARY_DIR}/PkgInfo )


endif()

# For Visual Studio project generation
if(CMAKE_SYSTEM_NAME STREQUAL "Windows")
	add_compile_definitions(
		_CRT_SECURE_NO_WARNINGS
	)

	set_target_properties(${PROJECT_NAME} PROPERTIES SUFFIX ".aex")
	get_filename_component(
		AFX_REZ
		${AE_SDK_PATH}/Resources/PiPLtool.exe
		ABSOLUTE
	)

	# Adobe is disgusting so we have to compile a resource format from the 90's into our project
	add_custom_command(
			OUTPUT ${CMAKE_CURRENT_SOURCE_DIR}/PiPL/PiPL.rr
					COMMAND
		cl 
		   /I \"${AE_SDK_PATH}/Headers\" 
		   /D \"AE_PROC_INTELx64\" 
		   /EP \"${CMAKE_CURRENT_SOURCE_DIR}/PiPL/PiPL.r\" > \"${CMAKE_CURRENT_SOURCE_DIR}/PiPL/PiPL.rr\"
	)

	add_custom_command(
		DEPENDS ${CMAKE_CURRENT_SOURCE_DIR}/PiPL/PiPL.rr
		OUTPUT ${CMAKE_CURRENT_SOURCE_DIR}/PiPL/PiPL.rrc
		COMMAND ${AFX_REZ} \"${CMAKE_CURRENT_SOURCE_DIR}/PiPL/PiPL.rr\" \"${CMAKE_CURRENT_SOURCE_DIR}/PiPL/PiPL.rrc\"
	)

	add_custom_command(
		DEPENDS ${CMAKE_CURRENT_SOURCE_DIR}/PiPL/PiPL.rrc
		OUTPUT ${CMAKE_CURRENT_SOURCE_DIR}/PiPL/PiPL.rc
		COMMAND cl /D "MSWindows" /EP \"${CMAKE_CURRENT_SOURCE_DIR}/PiPL/PiPL.rrc\" > \"${CMAKE_CURRENT_SOURCE_DIR}/PiPL/PiPL.rc\"
	)

	target_sources(
		${PROJECT_NAME} PRIVATE
		${CMAKE_CURRENT_SOURCE_DIR}/PiPL/PiPL.rc
	)

endif()

