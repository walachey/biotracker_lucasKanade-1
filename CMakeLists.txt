cmake_minimum_required(VERSION 3.12)

exec_program("git" ${CMAKE_CURRENT_SOURCE_DIR} ARGS "describe --tags --abbrev=0" OUTPUT_VARIABLE GIT_TAG)

project(BT_lukaskanade.tracker)

if("${CMAKE_SYSTEM_NAME}" STREQUAL "Windows")
    option(PACKAGE_WINDOWS_MSI "Create .msi package" OFF)
endif()
option(PACKAGE_TXZ "Create .tar.xz package" OFF)

if( CMAKE_SOURCE_DIR STREQUAL PROJECT_SOURCE_DIR )
    find_package(BT_Utility 0.1.1 REQUIRED)
endif()

add_subdirectory(Src)

include(cmake/Locate.cmake)
if("${CMAKE_SYSTEM_NAME}" STREQUAL "Windows")
    install(TARGETS ${TN} DESTINATION .)

    option(INSTALL_REQUIRED_SYSTEM_LIBRARIES "Install required system libraries" OFF)
    if(INSTALL_REQUIRED_SYSTEM_LIBRARIES)
        set(CMAKE_INSTALL_UCRT_LIBRARIES TRUE)
        set(CMAKE_INSTALL_SYSTEM_RUNTIME_DESTINATION .)
        include(InstallRequiredSystemLibraries)
    endif()

	#-DINSTALL_SHARED_LIBRARIES='boost_filesystem*' 
	#-DINSTALL_OPENCV_COMPONENTS='core;imgproc;video' 
	#qt5core qt5widgets qt5gui
    set(INSTALL_OPENCV_COMPONENTS "" CACHE STRING "OpenCV components to install")
    locate_opencv(shared_libraries COMPONENTS ${INSTALL_OPENCV_COMPONENTS})
	
    set(INSTALL_SHARED_LIBRARIES "" CACHE STRING "Additional shared libraries to install")
    locate_shared_library(shared_libraries ${INSTALL_SHARED_LIBRARIES})

    install(FILES ${shared_libraries} DESTINATION .)
endif()

set(CPACK_PACKAGE_NAME "BioTracker Plugin LukasKanade")
set(CPACK_PACKAGE_VENDOR "Biorobotics Lab / FU Berlin")

set(CPACK_PACKAGE_VERSION ${GIT_TAG})

if(PACKAGE_WINDOWS_MSI)
    set(CPACK_GENERATOR "WIX")
    set(CPACK_WIX_UPGRADE_GUID "f3dd97dc-0b5e-411c-a991-4c7d423c06af")

    set(CPACK_PACKAGE_INSTALL_DIRECTORY "RoboFish/LukasKanade_Tracker")

    set(CPACK_WIX_PROGRAM_MENU_FOLDER "RoboFish")
	
	set(CPACK_WIX_LICENSE_RTF ${CMAKE_SOURCE_DIR}/cmake/WIX/licence.rtf)
	set(CPACK_WIX_TEMPLATE "${CMAKE_SOURCE_DIR}/cmake/WIX/WIX.template.in")
elseif(PACKAGE_TXZ)
    set(CPACK_GENERATOR "TXZ")
endif()

include(CPack)
