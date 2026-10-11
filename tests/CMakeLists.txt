find_package(GTest CONFIG REQUIRED)

include(GoogleTest)

set(OTCLIENT_TEST_CORE otclient_core)
if(TARGET otclient_test_core)
    set(OTCLIENT_TEST_CORE otclient_test_core)
endif()

function(otclient_add_gtest TARGET_NAME)
    add_executable(${TARGET_NAME} ${ARGN})

    set_target_properties(${TARGET_NAME} PROPERTIES
        CXX_STANDARD 20
        CXX_STANDARD_REQUIRED ON
    )

    if(TOGGLE_PRE_COMPILED_HEADER)
        target_precompile_headers(${TARGET_NAME} PRIVATE ${CMAKE_SOURCE_DIR}/src/framework/pch.h)
    endif()

    target_link_libraries(${TARGET_NAME}
        PRIVATE
            ${OTCLIENT_TEST_CORE}
            GTest::gtest
            GTest::gtest_main
    )

    if(ANDROID)
        target_link_libraries(${TARGET_NAME}
            PRIVATE
                log
                c
        )
    endif()

    target_compile_definitions(${TARGET_NAME}
        PRIVATE
            CLIENT
            FRAMEWORK_GRAPHICS
            FRAMEWORK_NET
            FRAMEWORK_SOUND
            FRAMEWORK_XML
    )

    if(MSVC)
        target_compile_options(${TARGET_NAME} PRIVATE /utf-8)
        if(BUILD_STATIC_LIBRARY)
            set_property(TARGET ${TARGET_NAME} PROPERTY MSVC_RUNTIME_LIBRARY "MultiThreaded$<$<CONFIG:Debug>:Debug>")
        else()
            set_property(TARGET ${TARGET_NAME} PROPERTY MSVC_RUNTIME_LIBRARY "MultiThreaded$<$<CONFIG:Debug>:Debug>DLL")
        endif()
    endif()

    if(TARGET otclient_test_core)
        set_property(TARGET ${TARGET_NAME} PROPERTY INTERPROCEDURAL_OPTIMIZATION FALSE)
    endif()
    gtest_discover_tests(${TARGET_NAME})
endfunction()

add_subdirectory(map)
add_subdirectory(stdext)
add_subdirectory(otml)
