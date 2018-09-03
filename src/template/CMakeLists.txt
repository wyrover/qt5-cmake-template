cmake_minimum_required(VERSION 3.8.2)
project(template)

option(UPDATE_TRANSLATIONS "Update source translation translations/*.ts files (WARNING: make clean will delete the source .ts files! Danger!)" OFF)

set(CMAKE_PREFIX_PATH $ENV{QTDIR571})
set(CMAKE_CXX_STANDARD 11)
set(CMAKE_CXX_STANDARD_REQUIRED ON)
set(CMAKE_CXX_FLAGS_RELEASE "${CMAKE_CXX_FLAGS_RELEASE} /MT")
set(CMAKE_CXX_FLAGS_DEBUG "${CMAKE_CXX_FLAGS_DEBUG} /MTd")
set(CMAKE_INCLUDE_CURRENT_DIR ON)

include_directories(${CMAKE_BINARY_DIR})
include_directories(${CMAKE_SOURCE_DIR})



win_qt5_static()

set(CMAKE_AUTOMOC ON)


# Qt5 设置
find_package(Qt5 COMPONENTS Core Gui Widgets Sql REQUIRED)
find_package(Qt5LinguistTools REQUIRED)



set(UI_FILES
  mainwindow.ui
)

set(SRC_FILES
  main.cpp 
  mainwindow.cpp  
)


# Generate code from ui files
qt5_wrap_ui(UI_HEADERS ${UI_FILES})


# 生成翻译文件
set(FILES_TO_TRANSLATE)

# file(GLOB FTT_HEADER_FILES *.h)
# file(GLOB FTT_CPP_FILES    *.cpp)
# file(GLOB FTT_UI_FILES     *.ui)

set(FILES_TO_TRANSLATE ${UI_FILES} ${SRC_FILES})





# 多语言文件定义
set(TS_FILES
  translations/ar.ts  
  translations/zh_CN.ts
)    


# 生成多语言文件
if (UPDATE_TRANSLATIONS)
  qt5_create_translation(${PROJECT_NAME}_QM_FILES ${FILES_TO_TRANSLATE} ${TS_FILES})
else() 
  qt5_add_translation(${PROJECT_NAME}_QM_FILES ${TS_FILES})
endif()  




# 将多语言文件嵌入资源
file(WRITE "${CMAKE_CURRENT_BINARY_DIR}/translations.qrc.in"
"<!DOCTYPE RCC>
<RCC version=\"1.0\">
  <qresource prefix=\"/translations\">
")
foreach(qm ${${PROJECT_NAME}_QM_FILES})
  get_filename_component(qm_name ${qm} NAME)
  file(APPEND "${CMAKE_CURRENT_BINARY_DIR}/translations.qrc.in"
    "    <file alias=\"${qm_name}\">${qm}</file>\n")
endforeach(qm)
file(APPEND "${CMAKE_CURRENT_BINARY_DIR}/translations.qrc.in"
"  </qresource>
</RCC>
")

execute_process(COMMAND ${CMAKE_COMMAND} -E copy_if_different
  ${CMAKE_CURRENT_BINARY_DIR}/translations.qrc.in
  ${CMAKE_CURRENT_BINARY_DIR}/translations.qrc)



# 添加资源文件
qt5_add_resources(QRCS 
  resources.qrc
  #${CMAKE_CURRENT_BINARY_DIR}/translations.qrc
  DEPENDS ${${PROJECT_NAME}_QM_FILES}
)

# Tell CMake to create the helloworld executable
add_executable(template main.cpp mainwindow.cpp ${UI_HEADERS} ${QRCS})

# Add the Qt5 Widgets for linking
target_link_libraries(template 
    Qt5::Core 
    Qt5::Gui 
    Qt5::Widgets 
    Qt5::Sql
    ${Qt5Platform_LIBRARIES} 
    ${Qt5QWindows_LIBRARIES} 
    ${Qt5PNG_LIBRARIES} 
    ${Qt5QICO_LIBRARIES} 
    ${Qt5HB_LIBRARIES} 
    ${Qt5FT_LIBRARIES} 
    ${Qt5PCRE_LIBRARIES} 
    ${Qt5QSVGICON_LIBRARIES} 
    ${Qt5SVG_LIBRARIES} 
    ${Qt5SQLITE_LIBRARIES}   
    ws2_32 
    Imm32 
    Winmm    
)

# 设置预编译头
set_target_properties(template PROPERTIES COTIRE_CXX_PREFIX_HEADER_INIT "stdafx.h")
cotire(template)


