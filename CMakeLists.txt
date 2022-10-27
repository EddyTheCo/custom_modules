cmake_minimum_required(VERSION 3.11 FATAL_ERROR)

project(custom_modules VERSION 0.1 DESCRIPTION "custom torch modules" LANGUAGES CXX)



include(FetchContent)



add_library(custom_modules INTERFACE include/custom_modules.hpp)

if(NOT CUSTOM_MODULES)
	set(CUSTOM_MODULES "MPS;TTN" CACHE STRING "")
	message(STATUS "CUSTOM_MODULES not set using default ${CUSTOM_MODULES}")
endif(NOT CUSTOM_MODULES)

foreach(module IN CUSTOM_MODULES)
	FetchContent_Declare(
		${module}
		GIT_REPOSITORY git@github.com:EddyTheCo/${module}.git
		GIT_TAG main
		)
	FetchContent_MakeAvailable(${module})
	target_compile_definitions(custom_modules USE_${module} INTERFACE USE_${module})
	target_link_libraries(custom_modules INTERFACE  ${module})
endforeach()

target_include_directories(custom_modules INTERFACE  $<BUILD_INTERFACE:${CMAKE_CURRENT_SOURCE_DIR}/include>
	"$<INSTALL_INTERFACE:include>")

install(TARGETS custom_modules EXPORT custom_modulesTargets DESTINATION lib)
install(DIRECTORY include/ DESTINATION include/)

install(EXPORT custom_modulesTargets
	FILE custom_modulesTargets.cmake
	DESTINATION lib/cmake/custom_modules
	)
include(CMakePackageConfigHelpers)
# generate the config file that is includes the exports
configure_package_config_file(${CMAKE_CURRENT_SOURCE_DIR}/Config.cmake.in
	"${CMAKE_CURRENT_BINARY_DIR}/custom_modulesConfig.cmake"
	INSTALL_DESTINATION "lib/cmake/custom_modules"
	NO_SET_AND_CHECK_MACRO
	NO_CHECK_REQUIRED_COMPONENTS_MACRO
	)
write_basic_package_version_file(
	"${CMAKE_CURRENT_BINARY_DIR}/custom_modulesConfigVersion.cmake"
	VERSION "0.1.1.0"
	COMPATIBILITY AnyNewerVersion
	)
install(FILES
	${CMAKE_CURRENT_BINARY_DIR}/custom_modulesConfig.cmake
	${CMAKE_CURRENT_BINARY_DIR}/custom_modulesConfigVersion.cmake
	DESTINATION lib/cmake/custom_modules
	)
export(EXPORT custom_modulesTargets
	FILE "${CMAKE_CURRENT_BINARY_DIR}/custom_modulesTargets.cmake"
	)
