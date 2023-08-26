add_library(webview INTERFACE)

add_custom_target(
    webview-prep 
    COMMAND ${CMAKE_COMMAND} -E make_directory "${CMAKE_BINARY_DIR}/include" 
    COMMAND ${CMAKE_COMMAND} -E copy_if_different "${CMAKE_CURRENT_LIST_DIR}/webview.h" "${CMAKE_BINARY_DIR}/include/webview.h"
    COMMAND_EXPAND_LISTS
)

target_include_directories(webview INTERFACE "${CMAKE_BINARY_DIR}/include")
add_dependencies(webview webview-prep)

set_target_properties(webview PROPERTIES INTERFACE_SYSTEM_INCLUDE_DIRECTORIES $<TARGET_PROPERTY:webview,INTERFACE_INCLUDE_DIRECTORIES>)

target_link_libraries(
    webview
    INTERFACE
        $<$<PLATFORM_ID:Linux>:
            PkgConfig::webkit2
        >
)
target_link_libraries(
    webview
    INTERFACE
        $<$<PLATFORM_ID:Windows>:
            Version.lib
            WebView2Loader.lib
        >
)

if (WIN32)
    include(FetchContent)
    FetchContent_Declare(
        webview_binary
        URL ${NUI_WEBVIEW_BINARY_URL}
        DOWNLOAD_EXTRACT_TIMESTAMP true
    )
    FetchContent_GetProperties(webview_binary)
    if (webview_binary_POPULATED)
    else()
        FetchContent_Populate(webview_binary)
    endif()

    if (CMAKE_SIZEOF_VOID_P EQUAL 8)
        set(WEBVIEW_DLL_SUBDIRECTORY "x64")
    else()
        set(WEBVIEW_DLL_SUBDIRECTORY "x86")
    endif()

    target_include_directories(webview INTERFACE "${CMAKE_BINARY_DIR}/_deps/webview_binary-src/build/native/include")
    target_link_directories(
        webview 
        INTERFACE 
            "${CMAKE_BINARY_DIR}/_deps/webview_binary-src/build/native/${WEBVIEW_DLL_SUBDIRECTORY}"
            "${CMAKE_BINARY_DIR}/lib"
    )

    add_custom_command(
        OUTPUT "${CMAKE_BINARY_DIR}/bin/WebView2Loader.dll"
        COMMAND ${CMAKE_COMMAND} -E make_directory "${CMAKE_BINARY_DIR}/bin"
        COMMAND ${CMAKE_COMMAND} -E copy "${CMAKE_BINARY_DIR}/_deps/webview_binary-src/build/native/${WEBVIEW_DLL_SUBDIRECTORY}/WebView2Loader.dll" "${CMAKE_BINARY_DIR}/bin"
    )
    add_custom_command(
        OUTPUT "${CMAKE_BINARY_DIR}/lib/WebView2Loader.lib"
        COMMAND ${CMAKE_COMMAND} -E make_directory "${CMAKE_BINARY_DIR}/lib"
        COMMAND ${CMAKE_COMMAND} -E copy "${CMAKE_BINARY_DIR}/_deps/webview_binary-src/build/native/${WEBVIEW_DLL_SUBDIRECTORY}/WebView2Loader.dll.lib" "${CMAKE_BINARY_DIR}/lib/WebView2Loader.lib"
    )
    add_custom_target(
        webview2-win
        DEPENDS "${CMAKE_BINARY_DIR}/bin/WebView2Loader.dll"
        DEPENDS "${CMAKE_BINARY_DIR}/lib/WebView2Loader.lib"
    )

    add_dependencies(webview webview2-win)
endif()