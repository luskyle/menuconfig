cmake_minimum_required(VERSION 3.10)

project(menuconfig LANGUAGES C)

# 宽字符版 ncursesw: 窄字符 ncurses 对 UTF-8 逐字节做 isprint 判断, 中文会被
# 转义成 M-x 记号(乱码), 宽字符库才有多字节支持
set(CURSES_NEED_WIDE TRUE)
find_package(Curses REQUIRED)

# 宽字符模式下 FindCurses 把 CURSES_HAVE_NCURSES_NCURSES_H / _CURSES_H 指向
# ncursesw/ 下的头文件, 换成独立宏名, 与窄字符模式区分开
if (CURSES_HAVE_NCURSES_NCURSES_H)
   add_definitions(-DHAVE_NCURSESW_NCURSES_H=1)
endif ()
if (CURSES_HAVE_NCURSES_CURSES_H)
   add_definitions(-DHAVE_NCURSESW_CURSES_H=1)
endif ()
if (CURSES_HAVE_NCURSES_H)
   add_definitions(-DHAVE_NCURSES_H=1)
endif ()
if (CURSES_HAVE_CURSES_H)
   add_definitions(-DHAVE_CURSES_H=1)
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