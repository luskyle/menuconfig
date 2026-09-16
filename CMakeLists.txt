cmake_minimum_required(VERSION 3.10)

project(menuconfig LANGUAGES C)

set(CURSES_NEED_NCURSES TRUE)
find_package(Curses REQUIRED)

if (CURSES_HAVE_CURSES_H)
   add_definitions(-DHAVE_CURSES_H=1)
endif ()
if (CURSES_HAVE_NCURSES_H)
   add_definitions(-DHAVE_NCURSES_H=1)
endif ()
if (CURSES_HAVE_NCURSES_CURSES_H)
   add_definitions(-DHAVE_NCURSES_CURSES_H=1)
endif ()
if (CURSES_HAVE_NCURSES_NCURSES_H)
   add_definitions(-DHAVE_NCURSES_NCURSES_H=1)
endif ()

option(DEBUG "是否保留调试信息以便调试" ON)

# 用户显式指定了 CMAKE_BUILD_TYPE 就尊重它, 不要再无条件覆盖
if(NOT CMAKE_BUILD_TYPE AND NOT CMAKE_CONFIGURATION_TYPES)
    if(DEBUG)
        set(CMAKE_BUILD_TYPE Debug)
    else()
        set(CMAKE_BUILD_TYPE Release)
    endif()
endif()
message(STATUS "DEBUG=${DEBUG}, CMAKE_BUILD_TYPE=${CMAKE_BUILD_TYPE}")

include_directories(
   "${PROJECT_SOURCE_DIR}/lxdialog"
   ${CURSES_INCLUDE_DIRS}
)

add_subdirectory(lxdialog)


add_library(zconf STATIC zconf.tab.c)

add_executable(mconf mconf.c)
add_executable(conf conf.c)

target_link_libraries(mconf PRIVATE lxdialog ${CURSES_LIBRARIES} zconf)
target_link_libraries(conf PRIVATE zconf)