### QCefView 
################################################################################################
set(CMAKE_GLOBAL_AUTOGEN_TARGET OFF)
set(CMAKE_AUTOMOC ON)
set(CMAKE_INCLUDE_CURRENT_DIR ON)
find_package(Qt5 COMPONENTS Core GUI Widgets REQUIRED)

file(GLOB_RECURSE QCefView_INCLUDE_FILES
  "${CMAKE_CURRENT_SOURCE_DIR}/../include/*.h"
)
source_group(
  TREE "${CMAKE_CURRENT_SOURCE_DIR}/../include"
  PREFIX Include 
  FILES ${QCefView_INCLUDE_FILES}
)

file(GLOB_RECURSE QCefView_SRC_FILES
  "${CMAKE_CURRENT_SOURCE_DIR}/*.h"
  "${CMAKE_CURRENT_SOURCE_DIR}/*.cpp"
)
source_group(
  TREE ${CMAKE_CURRENT_SOURCE_DIR} 
  PREFIX Source 
  FILES ${QCefView_SRC_FILES}
)

if (OS_WINDOWS)
  file(GLOB_RECURSE QCefView_windows_SRC_FILES
    "${CMAKE_CURRENT_SOURCE_DIR}/win/*.h"
    "${CMAKE_CURRENT_SOURCE_DIR}/win/*.cpp"
  )
  source_group(
    TREE "${CMAKE_CURRENT_SOURCE_DIR}/win" 
    PREFIX Source 
    FILES ${QCefView_windows_SRC_FILES}
  )

  add_library(QCefView SHARED
    ${QCefView_INCLUDE_FILES}
    ${QCefView_SRC_FILES}
    ${QCefView_windows_SRC_FILES}
  )

  target_compile_definitions(QCefView PRIVATE
    UNICODE
    _UNICODE
    QCEFVIEW_LIB
  )
endif()  # OS_WINDOWS

if (OS_MACOS)
  file(GLOB_RECURSE QCefView_macOS_SRC_FILES
    "${CMAKE_CURRENT_SOURCE_DIR}/mac/*.h"
    "${CMAKE_CURRENT_SOURCE_DIR}/mac/*.cpp"
    "${CMAKE_CURRENT_SOURCE_DIR}/mac/*.mm"
  )
  source_group(
    TREE "${CMAKE_CURRENT_SOURCE_DIR}/mac" 
    PREFIX Source 
    FILES ${QCefView_macOS_SRC_FILES}
  )
  set(QCefView_INFO_PLIST_FILE "${CMAKE_CURRENT_LIST_DIR}/mac/Info.plist")

  add_library(QCefView SHARED
    ${QCefView_INCLUDE_FILES}
    ${QCefView_SRC_FILES}
    ${QCefView_macOS_SRC_FILES}
    ${QCefView_INFO_PLIST_FILE}
  )

  set_target_properties(QCefView
    PROPERTIES
      FRAMEWORK TRUE
      COMPILE_FLAGS                                               "-fobjc-arc"
      PUBLIC_HEADER                                               "${PerduCef_PUBLIC_HEADERS}"
      MACOSX_FRAMEWORK_INFO_PLIST                                 "${PerduCef_INFO_PLIST_FILE}"
      MACOSX_FRAMEWORK_IDENTIFIER                                 "com.perducef.perducefview"
      CLANG_ENABLE_OBJC_ARC                                       "YES"
      XCODE_ATTRIBUTE_CLANG_CXX_LANGUAGE_STANDARD                 "gnu++11"     # -std=gnu++11
      XCODE_ATTRIBUTE_CLANG_LINK_OBJC_RUNTIME                     "NO"          # -fno-objc-link-runtime
      XCODE_ATTRIBUTE_COPY_PHASE_STRIP                            "NO"
      XCODE_ATTRIBUTE_DEAD_CODE_STRIPPING[variant=Release]        "YES"         # -Wl,-dead_strip
      XCODE_ATTRIBUTE_GCC_C_LANGUAGE_STANDARD                     "c99"         # -std=c99
  )

  add_custom_command(
    TARGET QCefView
    PRE_BUILD

    #copy the cef framework to resource directory
    COMMAND ${CMAKE_COMMAND} -E copy_directory
    "$<TARGET_BUNDLE_DIR:QCefView>/../Chromium Embedded Framework.framework"
    "$<TARGET_FILE_DIR:QCefView>/Resources/Chromium Embedded Framework.framework"

    #copy the cef framework to resource directory
    COMMAND ${CMAKE_COMMAND} -E copy_directory
    "$<TARGET_BUNDLE_DIR:QCefView>/../CefViewWing.app"
    "$<TARGET_FILE_DIR:QCefView>/Resources/CefViewWing.app"
    VERBATIM
  )
endif()  # OS_MACOS

target_include_directories(QCefView PRIVATE
  "${CMAKE_CURRENT_SOURCE_DIR}/../include"
  ${CefViewCore_EXPORT_INCLUDE_PATH}
)

add_dependencies(QCefView
  CefViewCore
)

target_link_libraries(QCefView PRIVATE
  Qt5::Core
  Qt5::Gui 
  Qt5::Widgets
  CefViewCore
)

