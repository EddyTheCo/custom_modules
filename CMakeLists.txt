cmake_minimum_required(VERSION 3.11 FATAL_ERROR)

project(custom_modules VERSION 0.1 DESCRIPTION "custom torch modules" LANGUAGES CXX)



include(FetchContent)



add_library(custom_modules INTERFACE )

foreach(module ${CUSTOM_MODULES})

	message(STATUS "Loading module  ${module}")
	FetchContent_Declare(
		${module}
		GIT_REPOSITORY git@github.com:EddyTheCo/${module}.git
		GIT_TAG main
		)
	FetchContent_MakeAvailable(${module})
	target_compile_definitions(custom_modules INTERFACE USE_${module})
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
execute_process(COMMAND "git" "log" "--format='%H'" "-n 1"
        WORKING_DIRECTORY ${PROJECT_SOURCE_DIR}
        OUTPUT_VARIABLE  commit)
execute_process(COMMAND "git" "rev-parse" "--abbrev-ref" "HEAD" 
        WORKING_DIRECTORY ${PROJECT_SOURCE_DIR}
        OUTPUT_VARIABLE  branch)

FILE(WRITE ${CMAKE_CURRENT_BINARY_DIR}/.${PROJECT_NAME}.txt  ${PROJECT_NAME}:\n)
FILE(APPEND ${CMAKE_CURRENT_BINARY_DIR}/.${PROJECT_NAME}.txt  \t commit=${commit})
FILE(APPEND ${CMAKE_CURRENT_BINARY_DIR}/.${PROJECT_NAME}.txt  \t branch=${branch}\n)
install(FILES ${CMAKE_CURRENT_BINARY_DIR}/.${PROJECT_NAME}.txt DESTINATION .conf)
