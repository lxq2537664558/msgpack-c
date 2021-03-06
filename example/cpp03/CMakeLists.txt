LIST (APPEND exec_PROGRAMS
    class_intrusive.cpp
    class_intrusive_map.cpp
    class_non_intrusive.cpp
    custom.cpp
    enum.cpp
    map_based_versionup.cpp
    protocol_new.cpp
    reuse_zone.cpp
    simple.cpp
)

IF (MSGPACK_DEFAULT_API_VERSION EQUAL 1)
    LIST (APPEND exec_PROGRAMS
        protocol.cpp
    )
ENDIF ()

IF (NOT MSVC)
    LIST (APPEND with_pthread_PROGRAMS
        stream.cpp
    )
ENDIF ()

IF (MSGPACK_BOOST)
    LIST (APPEND with_boost_lib_PROGRAMS
        speed_test.cpp
        speed_test_nested_array.cpp
    )
ENDIF ()

FOREACH (source_file ${exec_PROGRAMS})
    GET_FILENAME_COMPONENT (source_file_we ${source_file} NAME_WE)
    ADD_EXECUTABLE (
        ${source_file_we}
        ${source_file}
    )
    TARGET_INCLUDE_DIRECTORIES (${source_file_we}
        PRIVATE
            $<TARGET_PROPERTY:msgpackc,INTERFACE_INCLUDE_DIRECTORIES>
    )
    IF ("${CMAKE_CXX_COMPILER_ID}" STREQUAL "Clang" OR "${CMAKE_CXX_COMPILER_ID}" STREQUAL "GNU")
        SET_PROPERTY (TARGET ${source_file_we} APPEND_STRING PROPERTY COMPILE_FLAGS "-Wall -Wextra -Werror -g -O3")
    ENDIF ()
    IF ("${CMAKE_CXX_COMPILER_ID}" STREQUAL "MSVC")
        IF (CMAKE_CXX_FLAGS MATCHES "/W[0-4]")
            STRING(REGEX REPLACE "/W[0-4]" "/W3 /WX" CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS}")
        ELSE ()
            SET(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} /W3 /WX")
        ENDIF ()
    ENDIF ()
ENDFOREACH ()

FOREACH (source_file ${with_pthread_PROGRAMS})
    GET_FILENAME_COMPONENT (source_file_we ${source_file} NAME_WE)
    ADD_EXECUTABLE (
        ${source_file_we}
        ${source_file}
    )
    TARGET_INCLUDE_DIRECTORIES (${source_file_we}
        PRIVATE
            $<TARGET_PROPERTY:msgpackc,INTERFACE_INCLUDE_DIRECTORIES>
    )
    TARGET_LINK_LIBRARIES (${source_file_we}
        ${CMAKE_THREAD_LIBS_INIT}
    )
    IF ("${CMAKE_CXX_COMPILER_ID}" STREQUAL "Clang" OR "${CMAKE_CXX_COMPILER_ID}" STREQUAL "GNU")
        SET_PROPERTY (TARGET ${source_file_we} APPEND_STRING PROPERTY COMPILE_FLAGS "-Wall -Wextra -Werror -g -O3")
    ENDIF ()
    IF ("${CMAKE_CXX_COMPILER_ID}" STREQUAL "MSVC")
        IF (CMAKE_CXX_FLAGS MATCHES "/W[0-4]")
            STRING(REGEX REPLACE "/W[0-4]" "/W3 /WX" CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS}")
        ELSE ()
            SET(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} /W3 /WX")
        ENDIF ()
    ENDIF ()
ENDFOREACH ()

FOREACH (source_file ${with_boost_lib_PROGRAMS})
    INCLUDE_DIRECTORIES (
        ../include
        ${Boost_INCLUDE_DIRS}
    )
    GET_FILENAME_COMPONENT (source_file_we ${source_file} NAME_WE)
    ADD_EXECUTABLE (
        ${source_file_we}
        ${source_file}
    )
    TARGET_INCLUDE_DIRECTORIES (${source_file_we}
        PRIVATE
            $<TARGET_PROPERTY:msgpackc,INTERFACE_INCLUDE_DIRECTORIES>
    )
    LINK_DIRECTORIES(${Boost_LIBRARY_DIRS})
    TARGET_LINK_LIBRARIES (${source_file_we}
        ${Boost_TIMER_LIBRARY}
        ${Boost_CHRONO_LIBRARY}
        ${Boost_SYSTEM_LIBRARY}
    )
    IF (NOT MSVC AND NOT APPLE)
        TARGET_LINK_LIBRARIES (${source_file_we}
            rt
        )
    ENDIF ()

    IF ("${CMAKE_CXX_COMPILER_ID}" STREQUAL "Clang" OR "${CMAKE_CXX_COMPILER_ID}" STREQUAL "GNU")
        SET_PROPERTY (TARGET ${source_file_we} APPEND_STRING PROPERTY COMPILE_FLAGS "-Wall -Wextra -Werror -O3")
    ENDIF ()

    IF ("${CMAKE_CXX_COMPILER_ID}" STREQUAL "Clang")
        SET_PROPERTY (TARGET ${source_file_we} APPEND_STRING PROPERTY COMPILE_FLAGS " -Wno-mismatched-tags")
    ENDIF ()

    IF ("${CMAKE_CXX_COMPILER_ID}" STREQUAL "MSVC")
        IF (CMAKE_CXX_FLAGS MATCHES "/W[0-4]")
            STRING(REGEX REPLACE "/W[0-4]" "/W3 /WX" CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS}")
        ELSE ()
            SET(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} /W3 /WX")
        ENDIF ()
    ENDIF ()
ENDFOREACH ()
