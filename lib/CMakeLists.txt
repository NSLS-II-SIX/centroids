set(centroids_LIBRARY centroids PARENT_SCOPE)
set(SOURCE_FILES
	photons.cpp
	${CMAKE_SOURCE_DIR}/lmfit/lmmin.c
	${CMAKE_SOURCE_DIR}/lmfit/lminvert.c
	${CMAKE_BINARY_DIR}/version.cpp
)
set(PUBLIC_INCLUDE_FILES
	centroids.h
)
set(PRIVATE_INCLUDE_FILES
	photons.h
	${CMAKE_SOURCE_DIR}/lmfit/lmmin.h
	${CMAKE_SOURCE_DIR}/lmfit/lmstruct.h
	${CMAKE_SOURCE_DIR}/lmfit/lminvert.h
)

set_property(SOURCE ${CMAKE_BINARY_DIR}/version.cpp PROPERTY GENERATED 1)

add_library(centroids_obj OBJECT
	${SOURCE_FILES}
	${PUBLIC_INCLUDE_FILES}
	${PRIVATE_INCLUDE_FILES})

target_include_directories(centroids_obj PUBLIC
	$<BUILD_INTERFACE:${CMAKE_CURRENT_SOURCE_DIR}>
	$<BUILD_INTERFACE:${CMAKE_SOURCE_DIR}/lmfit>
	$<INSTALL_INTERFACE:${CMAKE_INSTALL_INCLUDEDIR}>
)

add_dependencies(centroids_obj version_info)

set_property(TARGET centroids_obj PROPERTY POSITION_INDEPENDENT_CODE 1)

add_library(centroids_shared SHARED
	$<TARGET_OBJECTS:centroids_obj>)

add_library(centroids_static STATIC
	$<TARGET_OBJECTS:centroids_obj>)

set_target_properties(
    centroids_shared PROPERTIES
    OUTPUT_NAME centroids
    VERSION ${centroids_VERSION}
	SOVERSION ${centroids_VERSION_MAJOR}
	PUBLIC_HEADER "${PUBLIC_HEADER_FILES}"
	PRIVATE_HEADER "${PRIVATE_HEADER_FILES}"
)

set_target_properties(
    centroids_static PROPERTIES
    OUTPUT_NAME centroids
)
if(BUILD_WITH_OPENMP)
	find_package(OpenMP)
	if(OpenMP_FOUND)
		target_link_libraries(centroids_obj PUBLIC OpenMP::OpenMP_CXX)
	endif()
endif()

if(CPPLINT_CHECK)
	add_dependencies(centroids_obj ${CPPLINT_TARGET})
	message(STATUS "Checking CXX Code via cpplint")
endif()

include(GNUInstallDirs)
include(CMakePackageConfigHelpers)

install(
    TARGETS centroids_shared
	EXPORT centroids-targets
	LIBRARY DESTINATION ${CMAKE_INSTALL_LIBDIR}
	PUBLIC_HEADER DESTINATION "${CMAKE_INSTALL_INCLUDEDIR}"
	COMPONENT Libraries
)

install(
    TARGETS centroids_static
	EXPORT centroids-targets
	LIBRARY DESTINATION ${CMAKE_INSTALL_LIBDIR}
	PUBLIC_HEADER DESTINATION "${CMAKE_INSTALL_INCLUDEDIR}"
    COMPONENT Libraries
)

install(
	EXPORT centroids-targets
	FILE centroids-targets.cmake
	NAMESPACE centroids::
	DESTINATION "${CMAKE_INSTALL_LIBDIR}/cmake/centroids"
)

configure_package_config_file(
	"${CMAKE_CURRENT_SOURCE_DIR}/centroids-config.cmake.in"
	"${CMAKE_CURRENT_BINARY_DIR}/centroids-config.cmake"
	INSTALL_DESTINATION "${CMAKE_INSTALL_LIBDIR}/cmake/centroids"
)

write_basic_package_version_file(
	"${CMAKE_CURRENT_BINARY_DIR}/centroids-config-version.cmake"
	VERSION ${centroids_VERSION}
	COMPATIBILITY AnyNewerVersion
)

install(
	FILES "${CMAKE_CURRENT_BINARY_DIR}/centroids-config.cmake"
	"${CMAKE_CURRENT_BINARY_DIR}/centroids-config-version.cmake"
	DESTINATION "${CMAKE_INSTALL_LIBDIR}/cmake/centroids"
)
