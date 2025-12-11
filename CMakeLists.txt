cmake_minimum_required(VERSION 3.16)

set(PROJECT_NAME ukncbtl-qt)

project(${PROJECT_NAME} VERSION 1.0.1 LANGUAGES C CXX)


option(UKNCBTL_ENABLE_SCRIPTING "Enable scripting engine" OFF)
option(UKNCBTL_ENABLE_50HZ      "Enable 50Hz framerate"   ON)
option(UKNCBTL_ENABLE_TESTING   "Enable testing"          OFF)

if (USE_STATIC_QT)
  set(CMAKE_FIND_LIBRARY_PREFIXES "${CMAKE_STATIC_LIBRARY_PREFIX}")
  set(CMAKE_FIND_LIBRARY_SUFFIXES "${CMAKE_STATIC_LIBRARY_SUFFIX}")
  message("-- Due to static Qt usage: ")
  message("--   CMAKE_FIND_LIBRARY_PREFIXES was set to '${CMAKE_FIND_LIBRARY_PREFIXES}'")
  message("--   CMAKE_FIND_LIBRARY_SUFFIXES was set to '${CMAKE_FIND_LIBRARY_SUFFIXES}'")
endif()

set(PROJECT_QT_COMPONENTS
  Core Gui Widgets Multimedia LinguistTools SerialPort
)

if (${UKNCBTL_ENABLE_SCRIPTING})
  list(APPEND PROJECT_QT_COMPONENTS Script)
endif()

if (${UKNCBTL_ENABLE_TESTING})
  list(APPEND PROJECT_QT_COMPONENTS Test)
endif()

find_package(QT NAMES Qt5 COMPONENTS ${PROJECT_QT_COMPONENTS} REQUIRED)
find_package(Qt${QT_VERSION_MAJOR} COMPONENTS ${PROJECT_QT_COMPONENTS} REQUIRED)

get_target_property(MY_QT_TARGET_TYPE Qt${QT_VERSION_MAJOR}::Core TYPE)
if(MY_QT_TARGET_TYPE STREQUAL STATIC_LIBRARY)
  message("-- Using Qt${QT_VERSION_MAJOR} as a static library")
  if (NOT USE_STATIC_QT)
    message(FATAL_ERROR "-- Please use -DUSE_STATIC_QT=ON")
  endif()
else()
  message("-- Using Qt${QT_VERSION_MAJOR} as a shared library")
  if (USE_STATIC_QT)
    message(FATAL_ERROR "-- Please use -DUSE_STATIC_QT=OFF")
  endif()
endif()

# if (${QT_VERSION_MAJOR} STREQUAL 5)
#   find_package(Qt6 COMPONENTS Widgets CONFIG QUIET)
#   if (Qt6_FOUND)
#     get_target_property(UIC5_EXEC Qt5::uic IMPORTED_LOCATION)
#     get_target_property(UIC6_EXEC Qt6::uic IMPORTED_LOCATION)
#
#     message("-- Qt5 UIC_EXE: ${UIC5_EXEC}")
#     message("-- Qt6 UIC_EXE: ${UIC6_EXEC}")
#
#     message("-- Replace Qt5 UIC with Qt6 UIC as workaround for *.ui files generated with qt-designer based on Qt6")
#     set_target_properties(Qt5::uic PROPERTIES
#       IMPORTED_LOCATION ${UIC6_EXEC})
#   endif()
# endif()

set(SRC_DIR "src/ukncbtl-qt")

set(PROJECT_HEADERS_QT
  src/ukncbtl-qt/emulator/mainwindow.h
  src/ukncbtl-qt/emulator/qconsoleview.h
  src/ukncbtl-qt/emulator/qscreen.h
  src/ukncbtl-qt/emulator/qsoundout.h
  src/ukncbtl-qt/emulator/qkeyboardview.h
  src/ukncbtl-qt/emulator/qdebugview.h
  src/ukncbtl-qt/emulator/qdisasmview.h
  src/ukncbtl-qt/emulator/qmemoryview.h
  src/ukncbtl-qt/emulator/qdialogs.h
)

set(PROJECT_HEADERS
  src/ukncbtl-qt/emulator/Common.h

  src/ukncbtl-qt/emulator/emubase/Processor.h
  src/ukncbtl-qt/emulator/emubase/Memory.h
  src/ukncbtl-qt/emulator/emubase/Emubase.h
  src/ukncbtl-qt/emulator/emubase/Defines.h
  src/ukncbtl-qt/emulator/emubase/Board.h

  src/ukncbtl-qt/emulator/Emulator.h
)

set(PROJECT_SOURCES
  src/ukncbtl-qt/emulator/main.cpp
  src/ukncbtl-qt/emulator/Common.cpp
  src/ukncbtl-qt/emulator/Settings.cpp

  src/ukncbtl-qt/emulator/emubase/Processor.cpp
  src/ukncbtl-qt/emulator/emubase/Memory.cpp
  src/ukncbtl-qt/emulator/emubase/Hard.cpp
  src/ukncbtl-qt/emulator/emubase/Floppy.cpp
  src/ukncbtl-qt/emulator/emubase/Disasm.cpp
  src/ukncbtl-qt/emulator/emubase/Board.cpp
  src/ukncbtl-qt/emulator/emubase/SoundAY.cpp

  src/ukncbtl-qt/emulator/Emulator.cpp

  src/ukncbtl-qt/emulator/mainwindow.cpp
  src/ukncbtl-qt/emulator/qconsoleview.cpp
  src/ukncbtl-qt/emulator/qscreen.cpp
  src/ukncbtl-qt/emulator/qsoundout.cpp
  src/ukncbtl-qt/emulator/qkeyboardview.cpp
  src/ukncbtl-qt/emulator/qdebugview.cpp
  src/ukncbtl-qt/emulator/qdisasmview.cpp
  src/ukncbtl-qt/emulator/qmemoryview.cpp
  src/ukncbtl-qt/emulator/qdialogs.cpp
)

if (${UKNCBTL_ENABLE_SCRIPTING})
  list(APPEND PROJECT_HEADERS_QT
    src/ukncbtl-qt/emulator/qscripting.h)
  list(APPEND PROJECT_SOURCES
    src/ukncbtl-qt/emulator/qscripting.cpp)
endif()

if(${UKNCBTL_ENABLE_TESTING})
  list(APPEND PROJECT_HEADERS_QT
    src/ukncbtl-qt/emulator/UnitTests.h)
  list(APPEND PROJECT_SOURCES
    src/ukncbtl-qt/emulator/UnitTests.cpp)
endif()

set(PROJECT_WINRC
  src/ukncbtl-qt.rc
)

set(PROJECT_UIS
  src/ukncbtl-qt/emulator/mainwindow.ui
)

set(PROJECT_QRCS
  src/ukncbtl-qt/emulator/QtUkncBtl.qrc
)

set(PROJECT_TSS
  src/ukncbtl-qt/emulator/lang/ukncbtl_en.ts
  src/ukncbtl-qt/emulator/lang/ukncbtl_ru.ts
)

qt_wrap_cpp(PROJECT_SRC_MOCS
  ${PROJECT_HEADERS_QT}
  TARGET ${PROJECT_NAME}
)

qt_wrap_ui(PROJECT_UIS_MOCS
  ${PROJECT_UIS}
)

configure_file(src/ukncbtl-qt-i18n.qrc ${CMAKE_CURRENT_BINARY_DIR} COPYONLY)
qt5_add_translation(PROJECT_QM
  ${PROJECT_TSS}
)

qt_add_resources(PROJECT_QRCS_DAT
  ${PROJECT_QRCS}
  ${CMAKE_CURRENT_BINARY_DIR}/ukncbtl-qt-i18n.qrc
)

add_executable(${PROJECT_NAME}
  ${PROJECT_SRC_MOCS}
  ${PROJECT_UIS_MOCS}
  ${PROJECT_HEADERS}
  ${PROJECT_HEADERS_QT}
  ${PROJECT_SOURCES}
  ${PROJECT_UIS}
  ${PROJECT_WINRC}
  ${PROJECT_QM}
  ${PROJECT_QRCS_DAT}
)

message("-- UKNCBTL scripting engine: " ${UKNCBTL_ENABLE_SCRIPTING})
message("-- UKNCBTL 50Hz framerate  : " ${UKNCBTL_ENABLE_50HZ})
message("-- UKNCBTL test module     : " ${UKNCBTL_ENABLE_TESTING})
target_compile_definitions(${PROJECT_NAME}
  PRIVATE
    UKNCBTL_ENABLE_SCRIPTING=$<BOOL:${UKNCBTL_ENABLE_SCRIPTING}>
    UKNCBTL_ENABLE_50HZ=$<BOOL:${UKNCBTL_ENABLE_50HZ}>
    UKNCBTL_ENABLE_TESTING=$<BOOL:${UKNCBTL_ENABLE_TESTING}>
)

target_compile_features(${PROJECT_NAME}
  PRIVATE
    cxx_std_17
)

set_target_properties(${PROJECT_NAME}
  PROPERTIES
    WIN32_EXECUTABLE ON
)

target_link_libraries(${PROJECT_NAME}
  PRIVATE
    Qt${QT_VERSION_MAJOR}::Gui
    Qt${QT_VERSION_MAJOR}::Widgets
    Qt${QT_VERSION_MAJOR}::Multimedia
    $<$<BOOL:${UKNCBTL_ENABLE_SCRIPTING}>:Qt${QT_VERSION_MAJOR}::Script>
    Qt${QT_VERSION_MAJOR}::SerialPort
    $<$<BOOL:${UKNCBTL_ENABLE_TESTING}>:Qt${QT_VERSION_MAJOR}::Test>
#    $<$<PLATFORM_ID:Windows>:ws2_32>
)

target_include_directories(${PROJECT_NAME}
  PRIVATE
    src/ukncbtl-qt/emulator/
    ${CMAKE_CURRENT_SOURCE_DIR}
    ${CMAKE_CURRENT_BINARY_DIR}
)

include(GNUInstallDirs)
install(TARGETS ${PROJECT_NAME})
