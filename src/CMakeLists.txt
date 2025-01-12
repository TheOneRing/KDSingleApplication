# This file is part of KDSingleApplication.
#
# SPDX-FileCopyrightText: 2020-2023 Klarälvdalens Datakonsult AB, a KDAB Group company <info@kdab.com>
#
# SPDX-License-Identifier: MIT
#
# Contact KDAB at <info@kdab.com> for commercial licensing options.
#
set(KDSINGLEAPPLICATION_SRCS kdsingleapplication.cpp kdsingleapplication_localsocket.cpp)

set(KDSINGLEAPPLICATION_INSTALLABLE_INCLUDES kdsingleapplication.h kdsingleapplication_lib.h)

set(KDSINGLEAPPLICATION_NON_INSTALLABLE_INCLUDES kdsingleapplication_localsocket_p.h)

if(${PROJECT_NAME}_STATIC)
    add_library(kdsingleapplication STATIC ${KDSINGLEAPPLICATION_INSTALLABLE_INCLUDES} ${KDSINGLEAPPLICATION_SRCS})
    target_compile_definitions(kdsingleapplication PUBLIC KDSINGLEAPPLICATION_STATIC_BUILD)
else()
    add_library(kdsingleapplication SHARED ${KDSINGLEAPPLICATION_INSTALLABLE_INCLUDES} ${KDSINGLEAPPLICATION_SRCS})
    target_compile_definitions(kdsingleapplication PRIVATE KDSINGLEAPPLICATION_SHARED_BUILD)
endif()
add_library(
    KDAB::kdsingleapplication ALIAS kdsingleapplication
)

set(KDSINGLEAPPLICATION_INCLUDEDIR ${INSTALL_INCLUDE_DIR}/kdsingleapplication)

target_include_directories(
    kdsingleapplication
    PUBLIC $<INSTALL_INTERFACE:${KDSINGLEAPPLICATION_INCLUDEDIR}> $<BUILD_INTERFACE:${CMAKE_CURRENT_SOURCE_DIR}>
    PRIVATE ${CMAKE_CURRENT_SOURCE_DIR}
)

if(WIN32)
    target_link_libraries(kdsingleapplication kernel32)
endif()
target_link_libraries(
    kdsingleapplication Qt::Core Qt::Network
)

install(
    FILES ${KDSINGLEAPPLICATION_INSTALLABLE_INCLUDES}
    DESTINATION ${KDSINGLEAPPLICATION_INCLUDEDIR}
)

install(
    TARGETS kdsingleapplication
    EXPORT kdsingleapplicationTargets
    RUNTIME DESTINATION ${INSTALL_RUNTIME_DIR}
    LIBRARY DESTINATION ${INSTALL_LIBRARY_DIR}
    ARCHIVE DESTINATION ${INSTALL_ARCHIVE_DIR}
)

if(${PROJECT_NAME}_IS_ROOT_PROJECT)
    install(
        EXPORT kdsingleapplicationTargets
        FILE KDSingleApplicationTargets.cmake
        NAMESPACE KDAB::
        DESTINATION "${INSTALL_LIBRARY_DIR}/cmake/KDSingleApplication"
    )
    configure_file(KDSingleApplicationConfig.cmake.in KDSingleApplicationConfig.cmake @ONLY)
    install(FILES "${CMAKE_CURRENT_BINARY_DIR}/KDSingleApplicationConfig.cmake"
                  "${CMAKE_CURRENT_BINARY_DIR}/KDSingleApplicationConfigVersion.cmake"
            DESTINATION "${INSTALL_LIBRARY_DIR}/cmake/KDSingleApplication"
    )

    if(MSVC AND NOT ${PROJECT_NAME}_STATIC)
        install(
            FILES "$<TARGET_PDB_FILE_DIR:kdsingleapplication>/$<TARGET_PDB_FILE_NAME:kdsingleapplication>"
            DESTINATION ${INSTALL_LIBRARY_DIR}
            CONFIGURATIONS Debug RelWithDebInfo
        )
    endif()
endif()

# Generate library version files
include(ECMSetupVersion)
ecm_setup_version(
    ${${PROJECT_NAME}_VERSION}
    VARIABLE_PREFIX
    KDSINGLEAPPLICATION
    VERSION_HEADER
    "${CMAKE_CURRENT_BINARY_DIR}/kdsingleapplication_version.h"
    PACKAGE_VERSION_FILE
    "${CMAKE_CURRENT_BINARY_DIR}/KDSingleApplicationConfigVersion.cmake"
    SOVERSION
    ${${PROJECT_NAME}_SOVERSION}
    COMPATIBILITY
    AnyNewerVersion
)
install(
    FILES "${CMAKE_CURRENT_BINARY_DIR}/kdsingleapplication_version.h"
    DESTINATION ${INSTALL_INCLUDE_DIR}/kdsingleapplication
)
