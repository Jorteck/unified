find_package(SWIG 4.0 COMPONENTS csharp)

function(add_swig_plugin target)
    set(interfaceFile ${target}.i)
    set(interopFile ${target}Interop.cpp)

    add_custom_command(
        OUTPUT ${CMAKE_CURRENT_SOURCE_DIR}/${interopFile}
        DEPENDS Core
        DEPENDS ${CMAKE_CURRENT_SOURCE_DIR}/${interfaceFile}
        DEPENDS ${CMAKE_CURRENT_SOURCE_DIR}/generate.sh
        COMMAND chmod a+x ./generate.sh && mkdir -p ./${target} && ./generate.sh -v -c++ -fcompact -o ${interopFile} -outdir ./${target} -I../../NWNXLib/API ${ARGN} ${interfaceFile}
        WORKING_DIRECTORY ${CMAKE_CURRENT_SOURCE_DIR}
    )

    add_library(${target} SHARED ${CMAKE_CURRENT_SOURCE_DIR}/${interopFile} ${CMAKE_CURRENT_SOURCE_DIR}/${target}.cpp)
    add_sanitizers(${target})
    target_link_libraries(${target} Core "-Wl,--no-undefined")
    set_target_properties(${target} PROPERTIES PREFIX "${PLUGIN_PREFIX}")
    target_include_directories(${target} PRIVATE "${CMAKE_CURRENT_SOURCE_DIR}")
    target_compile_definitions(${target} PRIVATE "-DPLUGIN_NAME=\"${PLUGIN_PREFIX}${target}\"")
endfunction()

if(SWIG_FOUND)
    add_swig_plugin(SWIG_DotNET -csharp -dllimport "${PLUGIN_PREFIX}SWIG_DotNET" -namespace "NWN.Native.API")
endif(SWIG_FOUND)
