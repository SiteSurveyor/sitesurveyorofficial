cmake_minimum_required(VERSION 3.16)
project(SiteSurveyor VERSION 2.0.0 LANGUAGES CXX)

set(CMAKE_CXX_STANDARD 17)
set(CMAKE_CXX_STANDARD_REQUIRED ON)
set(CMAKE_AUTOMOC ON)
set(CMAKE_AUTORCC ON)
set(CMAKE_AUTOUIC ON)

find_package(Qt6 REQUIRED COMPONENTS Quick QuickControls2 Sql Positioning Location Network)

# Find SQLite3 and SpatiaLite
if(WIN32)
    # On Windows (vcpkg), use find_package which is better supported
    find_package(GDAL REQUIRED)
    find_package(PROJ REQUIRED)
    find_package(GEOS REQUIRED)
    find_package(SQLite3 REQUIRED)
    
    # Map targets to variables used later
    # Note: Modern CMake propagates includes via targets, so _INCLUDE_DIRS can be empty
    set(GDAL_LIBRARIES GDAL::GDAL)
    set(PROJ_LIBRARIES PROJ::proj)
    set(GEOS_LIBRARIES GEOS::geos_c)
    set(SQLITE3_LIBRARIES SQLite::SQLite3)
    
    # Spatialite is optional
    find_package(unofficial-spatialite CONFIG) 
    if(unofficial-spatialite_FOUND OR TARGET unofficial::spatialite::spatialite)
        set(SPATIALITE_LIBRARIES unofficial::spatialite::spatialite)
        set(SPATIALITE_FOUND TRUE)
    endif()
else()
    # On Linux, PkgConfig is standard
    find_package(PkgConfig REQUIRED)
    pkg_check_modules(SQLITE3 REQUIRED sqlite3)
    pkg_check_modules(SPATIALITE spatialite)
    pkg_check_modules(GDAL REQUIRED gdal)
    pkg_check_modules(PROJ REQUIRED proj)
    pkg_check_modules(GEOS REQUIRED geos)
endif()

qt_add_executable(SiteSurveyor
    src/main.cpp
    src/database/DatabaseManager.cpp
    src/database/DatabaseManager.h
    src/analysis/EarthworkEngine.cpp
    src/analysis/EarthworkEngine.h
    # New refactored analysis components
    src/analysis/GDALHelpers.h
    src/analysis/DTMGenerator.cpp
    src/analysis/DTMGenerator.h
    src/analysis/TINProcessor.cpp
    src/analysis/TINProcessor.h
    src/analysis/VolumeCalculator.cpp
    src/analysis/VolumeCalculator.h
    src/analysis/MeshExporter.cpp
    src/analysis/MeshExporter.h
    # Coordinate transformation utilities
    src/utilities/CoordinateTransformer.cpp
    src/utilities/CoordinateTransformer.h
    # Cloud sync
    src/cloud/CloudSyncManager.cpp
    src/cloud/CloudSyncManager.h
    resources/qml.qrc
)

target_include_directories(SiteSurveyor PRIVATE
    ${CMAKE_SOURCE_DIR}/src
    ${SQLITE3_INCLUDE_DIRS}
    ${GDAL_INCLUDE_DIRS}
    ${PROJ_INCLUDE_DIRS}
    ${GEOS_INCLUDE_DIRS}
    ${SPATIALITE_INCLUDE_DIRS}
)

target_link_libraries(SiteSurveyor PRIVATE
    Qt6::Quick
    Qt6::QuickControls2
    Qt6::Sql
    Qt6::Positioning
    Qt6::Location
    Qt6::Network
    ${GDAL_LIBRARIES}
    ${PROJ_LIBRARIES}
    ${GEOS_LIBRARIES}
    ${SQLITE3_LIBRARIES}
    ${SQLITE3_LIBRARIES}
    ${SPATIALITE_LIBRARIES}
)

target_compile_definitions(SiteSurveyor PRIVATE
    $<$<BOOL:${SPATIALITE_FOUND}>:WITH_SPATIALITE>
)

# Install
include(GNUInstallDirs)
install(TARGETS SiteSurveyor
    RUNTIME DESTINATION ${CMAKE_INSTALL_BINDIR}
    BUNDLE DESTINATION .
)
