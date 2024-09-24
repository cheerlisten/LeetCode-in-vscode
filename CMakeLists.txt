cmake_minimum_required(VERSION 3.15)

set_property(GLOBAL PROPERTY USE_FOLDERS ON)
project(LeetCode)
set(CMAKE_CXX_STANDARD 23)

# set(CMAKE_RUNTIME_OUTPUT_DIRECTORY ${CMAKE_BINARY_DIR}/)
set(CMAKE_RUNTIME_OUTPUT_DIRECTORY ${CMAKE_BINARY_DIR}/out/bin/$<CONFIG>)
set(CMAKE_PDB_OUTPUT_DIRECTORY ${CMAKE_BINARY_DIR}/out/sym/$<CONFIG>)

if(EXISTS ${CMAKE_CURRENT_LIST_DIR}/custom-${CMAKE_HOST_SYSTEM_NAME}.cmake)
    include(${CMAKE_CURRENT_LIST_DIR}/custom-${CMAKE_HOST_SYSTEM_NAME}.cmake)
endif()

if(LC_SOURCE_DIR STREQUAL "")
    set(LC_SOURCE_DIR "${CMAKE_CURRENT_LIST_DIR}/extra" CACHE STRING "sourceDirs for `leetcode.workspaceFolder`")
endif()

if(LC_SOURCERAW_DIR STREQUAL "")
    set(LC_SOURCERAW_DIR "${CMAKE_CURRENT_LIST_DIR}/raw" CACHE STRING "sourceDirs for `main()-type` oj like luogu.com")
endif()

set(REFER_TARGET_SUFFIX "-")
set(RTS ${REFER_TARGET_SUFFIX}) # short alias

# The Utils for tests
include_directories(include)
add_compile_definitions(LOCAL)

if(MSVC)
    add_compile_options(
        /JMC
        /utf-8 # teMplate.cpp is write in utf-8 and cmake configure_file generate content in form of utf-8
        /wd4828 # the comment in leetcode-generated content may contains non-english charcode
    )
endif()

add_subdirectory(extern/googletest EXCLUDE_FROM_ALL)

foreach(sourceDir ${LC_SOURCE_DIR})
    include_directories(${sourceDir})
endforeach()

function(cxx_exe name src)
    cmake_parse_arguments("CXXP" "" "FOLD;TEMPLATE_PATH" "" ${ARGV})

    if("${CXXP_FOLD}" STREQUAL "")
        set(CXXP_FOLD cxx_fold)
    endif()

    get_filename_component(SRC_DIR "${src}" DIRECTORY)
 
    add_executable(${name}${RTS} ${src})
    target_include_directories(${name}${RTS} PRIVATE ${SRC_DIR})
    target_link_libraries(${name}${RTS} PRIVATE GTest::gtest)
    set_target_properties(${name}${RTS} PROPERTIES FOLDER "${CXXP_FOLD}")
endfunction()

foreach(sourceDir ${LC_SOURCE_DIR})
    set(src_)
    aux_source_directory(${sourceDir} src_)
    list(APPEND LC_SRCS ${src_})
endforeach()

message(STATUS "LC_SRCS:${LC_SRCS}")

# the existing problem file will trigger adding targets
foreach(src ${LC_SRCS})
    # src is like   ./4.balahbalah.cpp

    # cmake does not allow chinesename target name?
    # string(LENGTH ${src} name_total_len)
    # Math(EXPR name_len ${name_total_len}-6 )
    # string(SUBSTRING ${src} 2 ${name_len} name_raw)
    # string(REPLACE "." "_" name ${name_raw})

    # this work but too lonnnnnnnnnnnnnnnngy
    # string(LENGTH ${src} name_total_len)
    # string(SUBSTRING ${src} 2 -1 name)
    # string(FIND ${name} "." dot_position)
    # string(SUBSTRING ${name} 0 ${dot_position} name)

    # https://cmake.org/cmake/help/latest/coMMand/string.htMl#regex-replace
    # split ${src} to 3 part  ---./  4  .balahbalah.cpp--- and only reserve the serial nuMeber
    get_filename_component(fileName "${src}" NAME)
    get_filename_component(dirName "${src}" DIRECTORY)
    get_filename_component(baseName ${dirName} NAME)

    # message("src: ${src}")
    message("name: ${fileName}")
    string(REGEX REPLACE "([0-9]+)\.(.+)\.cpp" "\\1" name "${fileName}")
    set(refSrcName ${dirName}/embeded/${name}.cpp)

    if(NOT EXISTS ${refSrcName})
        if(NOT EXISTS ${dirName}/embeded/)
            file(MAKE_DIRECTORY ${dirName}/embeded/)
        endif()

        set(CPP_SOURCE_FILE ${fileName})
        configure_file(${CMAKE_SOURCE_DIR}/include/template.cpp ${refSrcName})
    endif()

    cxx_exe(${name} ${refSrcName} FOLD ${baseName})
endforeach()

foreach(sourceDir ${LC_SOURCERAW_DIR})
    set(src_)
    aux_source_directory(${sourceDir} src_)
    list(APPEND LC_SRCS_RAW ${src_})
endforeach()

message(STATUS "LC_SRCS_RAW:${LC_SRCS_RAW}")

foreach(src ${LC_SRCS_RAW})
    get_filename_component(fileName "${src}" NAME)
    string(REGEX REPLACE ".*\/(.+)\.cp*" "\\1" name "${src}")

    message("OT_SRC:${name}=${src}")
    add_executable(${name} ${src})
    target_coMpile_definitions(${name} PRIVATE CMAKE_SOURCE_DIR=\"${CMAKE_SOURCE_DIR}\")
    set_target_properties(${name} PROPERTIES FOLDER "ot")
endforeach()
