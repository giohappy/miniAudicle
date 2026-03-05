cmake_minimum_required(VERSION 3.21)

project(miniAudicle LANGUAGES C CXX)

set(CMAKE_AUTOMOC ON)
set(CMAKE_AUTOUIC ON)
set(CMAKE_AUTORCC ON)

set(CMAKE_C_STANDARD 11)
set(CMAKE_CXX_STANDARD 17)
set(CMAKE_CXX_STANDARD_REQUIRED ON)
set(CMAKE_CXX_EXTENSIONS OFF)

find_package(Qt6 6.7 REQUIRED COMPONENTS Core Gui Network Widgets)

set(MINIAUDICLE_SOURCES
    miniAudicle.cpp
    miniAudicle_log.cpp
    util_rterror.cpp
    chuck/src/core/util_xforms.c
    chuck/src/core/util_thread.cpp
    chuck/src/core/util_string.cpp
    chuck/src/core/util_serial.cpp
    chuck/src/core/util_raw.c
    chuck/src/core/util_platforms.cpp
    chuck/src/core/util_opsc.cpp
    chuck/src/core/util_network.c
    chuck/src/core/util_math.cpp
    chuck/src/core/util_hid.cpp
    chuck/src/core/util_console.cpp
    chuck/src/core/util_buffers.cpp
    chuck/src/core/ulib_std.cpp
    chuck/src/core/ulib_opsc.cpp
    chuck/src/core/ulib_math.cpp
    chuck/src/core/ulib_machine.cpp
    chuck/src/core/ulib_doc.cpp
    chuck/src/core/ulib_ai.cpp
    chuck/src/core/ugen_xxx.cpp
    chuck/src/core/ugen_stk.cpp
    chuck/src/core/ugen_osc.cpp
    chuck/src/core/ugen_filter.cpp
    chuck/src/core/uana_xform.cpp
    chuck/src/core/uana_extract.cpp
    chuck/src/core/rtmidi.cpp
    chuck/src/core/midiio_rtmidi.cpp
    chuck/src/core/hidio_sdl.cpp
    chuck/src/core/chuck.cpp
    chuck/src/core/chuck_vm.cpp
    chuck/src/core/chuck_ugen.cpp
    chuck/src/core/chuck_type.cpp
    chuck/src/core/chuck_table.cpp
    chuck/src/core/chuck_symbol.cpp
    chuck/src/core/chuck_stats.cpp
    chuck/src/core/chuck_shell.cpp
    chuck/src/core/chuck_scan.cpp
    chuck/src/core/chuck_parse.cpp
    chuck/src/core/chuck_otf.cpp
    chuck/src/core/chuck_oo.cpp
    chuck/src/core/chuck_lang.cpp
    chuck/src/core/chuck_globals.cpp
    chuck/src/core/chuck_instr.cpp
    chuck/src/core/chuck_io.cpp
    chuck/src/core/chuck_frame.cpp
    chuck/src/core/chuck_errmsg.cpp
    chuck/src/core/chuck_emit.cpp
    chuck/src/core/chuck_dl.cpp
    chuck/src/core/chuck_compile.cpp
    chuck/src/core/chuck_carrier.cpp
    chuck/src/core/chuck_absyn.cpp
    chuck/src/core/lo/address.c
    chuck/src/core/lo/blob.c
    chuck/src/core/lo/bundle.c
    chuck/src/core/lo/message.c
    chuck/src/core/lo/method.c
    chuck/src/core/lo/pattern_match.c
    chuck/src/core/lo/send.c
    chuck/src/core/lo/server.c
    chuck/src/core/lo/timetag.c
    chuck/src/host/chuck_audio.cpp
    chuck/src/host/chuck_console.cpp
    chuck/src/host/RtAudio/RtAudio.cpp
    qt/ZSettings.cpp
    qt/mAMainWindow.cpp
    qt/madocumentview.cpp
    qt/mAConsoleMonitor.cpp
    qt/mAVMMonitor.cpp
    qt/mAsciLexerChucK.cpp
    qt/mAPreferencesWindow.cpp
    qt/mAExportDialog.cpp
    qt/mASocketManager.cpp
    qt/mADeviceBrowser.cpp
    qt/main.cpp
    qt/mAMainWindow.ui
    qt/madocumentview.ui
    qt/mAConsoleMonitor.ui
    qt/mAVMMonitor.ui
    qt/mAPreferencesWindow.ui
    qt/mAExportDialog.ui
    qt/mADeviceBrowser.ui
    qt/miniAudicle.qrc
)

list(APPEND MINIAUDICLE_SOURCES chuck/src/core/util_sndfile.c)
if(CMAKE_SYSTEM_NAME STREQUAL "Linux" AND CMAKE_CXX_COMPILER_ID STREQUAL "GNU")
    list(REMOVE_ITEM MINIAUDICLE_SOURCES chuck/src/core/util_sndfile.c)
endif()

if(WIN32)
    list(APPEND MINIAUDICLE_SOURCES
        chuck/src/core/chuck_yacc.c
        chuck/src/host/RtAudio/include/asio.cpp
        chuck/src/host/RtAudio/include/asiodrivers.cpp
        chuck/src/host/RtAudio/include/asiolist.cpp
        chuck/src/host/RtAudio/include/iasiothiscallresolver.cpp
        qt/icon/miniAudicle.rc
    )
endif()

add_executable(miniAudicle WIN32 ${MINIAUDICLE_SOURCES})

target_compile_definitions(miniAudicle PRIVATE
    __MA_COLOR_CONSOLE__
    HAVE_CONFIG_H
)

target_include_directories(miniAudicle PRIVATE
    ${CMAKE_CURRENT_SOURCE_DIR}
    ${CMAKE_CURRENT_SOURCE_DIR}/qt
    ${CMAKE_CURRENT_SOURCE_DIR}/chuck/src/core
    ${CMAKE_CURRENT_SOURCE_DIR}/chuck/src/host
    ${CMAKE_CURRENT_SOURCE_DIR}/chuck/src/core/lo
    ${CMAKE_CURRENT_SOURCE_DIR}/chuck/src/host/RtAudio/include
)

if(WIN32)
    target_compile_definitions(miniAudicle PRIVATE
        __WINDOWS_DS__
        __WINDOWS_WASAPI__
        __WINDOWS_ASIO__
        _WINSOCKAPI_
    )

    target_link_directories(miniAudicle PRIVATE ${CMAKE_CURRENT_SOURCE_DIR}/qt/lib)
    target_link_libraries(miniAudicle PRIVATE
        ws2_32
        dinput8
        advapi32
        kernel32
        user32
        gdi32
        dsound
        dxguid
        winmm
        ole32
        iphlpapi
    )

    target_link_libraries(miniAudicle PRIVATE
        $<$<CONFIG:Debug>:qscintilla2_qt6d>
        $<$<NOT:$<CONFIG:Debug>>:qscintilla2_qt6>
    )

    if(MSVC)
        target_compile_options(miniAudicle PRIVATE /UUNICODE /U_UNICODE)
    endif()

    set_source_files_properties(
        chuck/src/host/RtAudio/include/asio.cpp
        chuck/src/host/RtAudio/include/asiodrivers.cpp
        chuck/src/host/RtAudio/include/asiolist.cpp
        chuck/src/host/RtAudio/include/iasiothiscallresolver.cpp
        PROPERTIES COMPILE_DEFINITIONS "_HAS_STD_BYTE=0"
    )
else()
    find_package(FLEX)
    find_package(BISON)
    if(FLEX_FOUND AND BISON_FOUND)
        BISON_TARGET(chuck_parser chuck/src/core/chuck.y ${CMAKE_CURRENT_BINARY_DIR}/chuck.tab.c DEFINES_FILE ${CMAKE_CURRENT_BINARY_DIR}/chuck.tab.h)
        FLEX_TARGET(chuck_lexer chuck/src/core/chuck.lex ${CMAKE_CURRENT_BINARY_DIR}/chuck.yy.c)
        ADD_FLEX_BISON_DEPENDENCY(chuck_lexer chuck_parser)
        target_sources(miniAudicle PRIVATE ${BISON_chuck_parser_OUTPUTS} ${FLEX_chuck_lexer_OUTPUTS})
        target_include_directories(miniAudicle PRIVATE ${CMAKE_CURRENT_BINARY_DIR})
    endif()
endif()

target_link_libraries(miniAudicle PRIVATE Qt6::Core Qt6::Gui Qt6::Network Qt6::Widgets)

get_target_property(_qt_core_includes Qt6::Core INTERFACE_INCLUDE_DIRECTORIES)
set(_qsci_hint_roots "")
foreach(_inc IN LISTS _qt_core_includes)
    if(_inc MATCHES "(/|\\\\)QtCore$")
        cmake_path(GET _inc PARENT_PATH _qt_include_root)
        list(APPEND _qsci_hint_roots "${_qt_include_root}")
    endif()
endforeach()
if(Qt6_DIR)
    cmake_path(SET _qt6_dir NORMALIZE "${Qt6_DIR}")
    cmake_path(GET _qt6_dir PARENT_PATH _qt6_cmake_dir)
    cmake_path(GET _qt6_cmake_dir PARENT_PATH _qt6_lib_dir)
    cmake_path(GET _qt6_lib_dir PARENT_PATH _qt6_root_dir)
    list(APPEND _qsci_hint_roots "${_qt6_root_dir}/include")
endif()

find_path(QSCI_INCLUDE_DIR
    NAMES Qsci/qsciscintilla.h
    HINTS ${_qsci_hint_roots}
)
if(NOT QSCI_INCLUDE_DIR)
    message(FATAL_ERROR "QScintilla headers not found (expected Qsci/qsciscintilla.h).")
endif()
target_include_directories(miniAudicle PRIVATE ${QSCI_INCLUDE_DIR})
