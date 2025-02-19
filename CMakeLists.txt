CMAKE_MINIMUM_REQUIRED (VERSION 2.8.12)

IF ((CMAKE_VERSION VERSION_GREATER 3.1) OR
    (CMAKE_VERSION VERSION_EQUAL 3.1))
    CMAKE_POLICY(SET CMP0054 NEW)
ENDIF ()

PROJECT (msgpack)

FILE (READ ${CMAKE_CURRENT_SOURCE_DIR}/include/msgpack/version_master.h contents)
STRING (REGEX MATCH "#define MSGPACK_VERSION_MAJOR *([0-9a-zA-Z_]*)" NULL_OUT ${contents})
SET (VERSION_MAJOR ${CMAKE_MATCH_1})
STRING (REGEX MATCH "#define MSGPACK_VERSION_MINOR *([0-9a-zA-Z_]*)" NULL_OUT ${contents})
SET (VERSION_MINOR ${CMAKE_MATCH_1})
STRING (REGEX MATCH "#define MSGPACK_VERSION_REVISION *([0-9a-zA-Z_]*)" NULL_OUT ${contents})
SET (VERSION_REVISION ${CMAKE_MATCH_1})
SET (VERSION ${VERSION_MAJOR}.${VERSION_MINOR}.${VERSION_REVISION})

SET (prefix ${CMAKE_INSTALL_PREFIX})
SET (exec_prefix "\${prefix}")
SET (libdir "\${exec_prefix}/lib")
SET (includedir "\${prefix}/include")
SET (GNUCXX_STD_SUPPORT_VERSION "4.3")

OPTION (MSGPACK_CXX11 "Using c++11 compiler" OFF)
OPTION (MSGPACK_32BIT "32bit compile" OFF)
OPTION (MSGPACK_BOOST "Using boost libraries" OFF)

IF (APPLE)
    SET(CMAKE_MACOSX_RPATH ON)
    SET(CMAKE_SKIP_BUILD_RPATH FALSE)
    SET(CMAKE_BUILD_WITH_INSTALL_RPATH FALSE)
    SET(CMAKE_INSTALL_RPATH "${CMAKE_INSTALL_PREFIX}/lib")
    SET(CMAKE_INSTALL_RPATH_USE_LINK_PATH TRUE)
    LIST(FIND CMAKE_PLATFORM_IMPLICIT_LINK_DIRECTORIES "${CMAKE_INSTALL_PREFIX}/lib" isSystemDir)
    IF ("${isSystemDir}" STREQUAL "-1")
        SET(CMAKE_INSTALL_RPATH "${CMAKE_INSTALL_PREFIX}/lib")
    ENDIF ()
ENDIF ()

IF (MSGPACK_USE_X3_PARSE)
   IF ("${CMAKE_CXX_COMPILER_ID}" STREQUAL "GNU")
      SET (CMAKE_CXX_FLAGS "-DMSGPACK_USE_X3_PARSE -std=c++14 ${CMAKE_CXX_FLAGS}")
   ELSEIF ("${CMAKE_CXX_COMPILER_ID}" STREQUAL "Clang")
      SET (CMAKE_CXX_FLAGS "-DMSGPACK_USE_X3_PARSE -std=c++14 ${CMAKE_CXX_FLAGS}")
   ELSEIF ("${CMAKE_CXX_COMPILER_ID}" STREQUAL "MSVC")
      SET (CMAKE_CXX_FLAGS "-DMSGPACK_USE_X3_PARSE ${CMAKE_CXX_FLAGS}")
      IF (CMAKE_CXX_COMPILER_VERSION VERSION_LESS 19)
          MESSAGE ( FATAL_ERROR "MSVC doesn't support C++14.")
      ENDIF ()
   ENDIF ()
ELSE ()
   IF (MSGPACK_CXX17)
      IF ("${CMAKE_CXX_COMPILER_ID}" STREQUAL "GNU")
         SET (CMAKE_CXX_FLAGS "-std=c++17 ${CMAKE_CXX_FLAGS}")
      ELSEIF ("${CMAKE_CXX_COMPILER_ID}" STREQUAL "Clang")
         SET (CMAKE_CXX_FLAGS "-std=c++17 ${CMAKE_CXX_FLAGS}")
      ELSEIF ("${CMAKE_CXX_COMPILER_ID}" STREQUAL "MSVC")
         MESSAGE ( FATAL_ERROR "MSVC doesn't support C++17.")
      ENDIF ()
   ELSEIF (MSGPACK_CXX11)
      IF ("${CMAKE_CXX_COMPILER_ID}" STREQUAL "GNU")
         SET (CMAKE_CXX_FLAGS "-std=c++11 ${CMAKE_CXX_FLAGS}")
      ELSEIF ("${CMAKE_CXX_COMPILER_ID}" STREQUAL "Clang")
         SET (CMAKE_CXX_FLAGS "-std=c++11 ${CMAKE_CXX_FLAGS}")
      ELSEIF ("${CMAKE_CXX_COMPILER_ID}" STREQUAL "MSVC")
         IF (CMAKE_CXX_COMPILER_VERSION VERSION_LESS 19)
             MESSAGE ( FATAL_ERROR "MSVC doesn't support C++11.")
         ENDIF ()
      ENDIF ()
   ELSE ()
      IF ("${CMAKE_CXX_COMPILER_ID}" STREQUAL "GNU")
         IF ((CMAKE_CXX_COMPILER_VERSION VERSION_GREATER ${GNUCXX_STD_SUPPORT_VERSION}) OR
             (CMAKE_CXX_COMPILER_VERSION VERSION_EQUAL   ${GNUCXX_STD_SUPPORT_VERSION}))
             SET (CMAKE_CXX_FLAGS "-std=c++98 ${CMAKE_CXX_FLAGS}")
         ENDIF ()
      ELSEIF ("${CMAKE_CXX_COMPILER_ID}" STREQUAL "Clang")
         SET (CMAKE_CXX_FLAGS "-std=c++98 ${CMAKE_CXX_FLAGS}")
      ELSEIF ("${CMAKE_CXX_COMPILER_ID}" STREQUAL "MSVC")
         IF (CMAKE_CXX_COMPILER_VERSION VERSION_GREATER 18)
             SET (CMAKE_CXX_FLAGS "-DMSGPACK_USE_CPP03 ${CMAKE_CXX_FLAGS}")
         ENDIF ()
      ENDIF ()
   ENDIF ()
ENDIF ()

IF (MSGPACK_32BIT)
   IF ("${CMAKE_CXX_COMPILER_ID}" STREQUAL "GNU")
      SET (CMAKE_CXX_FLAGS "-m32 ${CMAKE_CXX_FLAGS}")
      SET (CMAKE_C_FLAGS "-m32 ${CMAKE_C_FLAGS}")
      SET (CMAKE_EXE_LINKER_FLAGS "-m32 ${CMAKE_EXE_LINKER_FLAGS}")
   ELSEIF ("${CMAKE_CXX_COMPILER_ID}" STREQUAL "Clang")
      SET (CMAKE_CXX_FLAGS "-m32 ${CMAKE_CXX_FLAGS}")
      SET (CMAKE_C_FLAGS "-m32 ${CMAKE_C_FLAGS}")
      SET (CMAKE_EXE_LINKER_FLAGS "-m32 ${CMAKE_EXE_LINKER_FLAGS}")
   ENDIF ()
ENDIF ()

OPTION (MSGPACK_BUILD_EXAMPLES "Build msgpack examples." ON)

IF (MSGPACK_BOOST)
   SET (CMAKE_CXX_FLAGS "-DMSGPACK_USE_BOOST ${CMAKE_CXX_FLAGS}")
   SET (Boost_USE_MULTITHREADED      ON)
   SET (Boost_USE_STATIC_RUNTIME    OFF)
   FIND_PACKAGE (Boost REQUIRED COMPONENTS chrono context system timer)
   INCLUDE_DIRECTORIES (
      ${Boost_INCLUDE_DIRS}
   )
   LINK_DIRECTORIES (
      ${Boost_LIBRARY_DIRS}
   )
   IF (MSGPACK_BOOST_DIR)
      INCLUDE_DIRECTORIES (
         ${MSGPACK_BOOST_DIR}
      )
   ENDIF ()
ENDIF ()

IF (MSGPACK_CHAR_SIGN)
   SET (CMAKE_C_FLAGS "-f${MSGPACK_CHAR_SIGN}-char ${CMAKE_C_FLAGS}")
   SET (CMAKE_CXX_FLAGS "-f${MSGPACK_CHAR_SIGN}-char ${CMAKE_CXX_FLAGS}")
ENDIF ()

IF (MSGPACK_DEFAULT_API_VERSION)
   SET (CMAKE_CXX_FLAGS "-DMSGPACK_DEFAULT_API_VERSION=${MSGPACK_DEFAULT_API_VERSION} ${CMAKE_CXX_FLAGS}")
ELSE ()
   SET (CMAKE_CXX_FLAGS "-DMSGPACK_DEFAULT_API_VERSION=3 ${CMAKE_CXX_FLAGS}")
ENDIF ()

FILE (GLOB_RECURSE PREDEF_FILES RELATIVE ${CMAKE_CURRENT_SOURCE_DIR}/external/boost/predef/include/boost ${CMAKE_CURRENT_SOURCE_DIR}/external/boost/predef/include/boost/*.h)
FOREACH (F ${PREDEF_FILES})
   SET(M "Converting ${F}")
   MESSAGE(STATUS ${M})
   FILE (READ ${CMAKE_CURRENT_SOURCE_DIR}/external/boost/predef/include/boost/${F} CONTENT)
   STRING(REPLACE "BOOST_" "MSGPACK_" CONTENT ${CONTENT})
   STRING(REPLACE "boost/" "msgpack/" CONTENT ${CONTENT})
   FILE (GENERATE OUTPUT ${CMAKE_CURRENT_SOURCE_DIR}/include/msgpack/${F} CONTENT ${CONTENT})
ENDFOREACH ()

FILE (GLOB_RECURSE PREPROCESSOR_FILES RELATIVE ${CMAKE_CURRENT_SOURCE_DIR}/external/boost/preprocessor/include/boost ${CMAKE_CURRENT_SOURCE_DIR}/external/boost/preprocessor/include/boost/*.hpp)
FOREACH (F ${PREPROCESSOR_FILES})
   SET(M "Converting ${F}")
   MESSAGE(STATUS ${M})
   FILE (READ ${CMAKE_CURRENT_SOURCE_DIR}/external/boost/preprocessor/include/boost/${F} CONTENT)
   STRING(REPLACE "BOOST_" "MSGPACK_" CONTENT ${CONTENT})
   STRING(REPLACE "boost/" "msgpack/" CONTENT ${CONTENT})
   FILE (GENERATE OUTPUT ${CMAKE_CURRENT_SOURCE_DIR}/include/msgpack/${F} CONTENT ${CONTENT})
ENDFOREACH ()

FIND_PACKAGE (GTest)
FIND_PACKAGE (ZLIB)
FIND_PACKAGE (Threads)
IF (GTEST_FOUND AND ZLIB_FOUND AND THREADS_FOUND AND NOT "${MSGPACK_FUZZ_REGRESSION}" STREQUAL "ON")
    OPTION (MSGPACK_BUILD_TESTS "Build msgpack tests." ON)
ENDIF ()

IF (DEFINED BUILD_SHARED_LIBS)
    IF (BUILD_SHARED_LIBS)
        IF (DEFINED MSGPACK_ENABLE_SHARED AND NOT MSGPACK_ENABLE_SHARED)
            MESSAGE(WARNING "MSGPACK_ENABLE_SHARED is overridden to ON by BUILD_SHARED_LIBS")
        ENDIF ()
        SET (MSGPACK_ENABLE_SHARED ON)
        IF (DEFINED MSGPACK_ENABLE_STATIC AND MSGPACK_ENABLE_STATIC)
            MESSAGE(WARNING "MSGPACK_ENABLE_STATIC is overridden to OFF by BUILD_SHARED_LIBS")
        ENDIF ()
        SET (MSGPACK_ENABLE_STATIC OFF)
    ELSE ()
        IF (DEFINED MSGPACK_ENABLE_SHARED AND MSGPACK_ENABLE_SHARED)
            MESSAGE(WARNING "MSGPACK_ENABLE_SHARED is overridden to OFF by BUILD_SHARED_LIBS")
        ENDIF ()
        SET (MSGPACK_ENABLE_SHARED OFF)
        IF (DEFINED MSGPACK_ENABLE_STATIC AND NOT MSGPACK_ENABLE_STATIC)
            MESSAGE(WARNING "MSGPACK_ENABLE_STATIC is overridden to ON by BUILD_SHARED_LIBS")
        ENDIF ()
        SET (MSGPACK_ENABLE_STATIC ON)
    ENDIF ()
ELSE ()
    IF (NOT DEFINED MSGPACK_ENABLE_SHARED)
        SET (MSGPACK_ENABLE_SHARED ON)
    ENDIF ()
    IF (NOT DEFINED MSGPACK_ENABLE_STATIC)
        SET (MSGPACK_ENABLE_STATIC ON)
    ENDIF ()
ENDIF ()

OPTION (MSGPACK_ENABLE_CXX "Enable C++ interface." ON)

INCLUDE (CheckCXXSourceCompiles)
CHECK_CXX_SOURCE_COMPILES ("
#include <bits/atomicity.h>
int atomic_sub(int i) { return __gnu_cxx::__exchange_and_add(&i, -1) - 1; }
int atomic_add(int i) { return __gnu_cxx::__exchange_and_add(&i, 1) + 1; }
int main(int argc, char * argv[])
{
    atomic_sub(1);
    atomic_add(1);
}
" MSGPACK_ENABLE_GCC_CXX_ATOMIC)

INCLUDE (Files.cmake)

EXECUTE_PROCESS (
    COMMAND ${CMAKE_COMMAND} -E make_directory ${CMAKE_CURRENT_BINARY_DIR}/src/msgpack
)

CONFIGURE_FILE (
    msgpack.pc.in
    msgpack.pc
    @ONLY
)

IF (MSGPACK_ENABLE_SHARED)
    ADD_LIBRARY (msgpackc SHARED
        ${msgpackc_SOURCES}
        ${msgpackc_HEADERS}
    )

    SET_TARGET_PROPERTIES (msgpackc PROPERTIES SOVERSION 2 VERSION 2.0.0)

    TARGET_INCLUDE_DIRECTORIES (msgpackc
        PUBLIC
            $<BUILD_INTERFACE:${CMAKE_CURRENT_SOURCE_DIR}/include>
            $<BUILD_INTERFACE:${CMAKE_CURRENT_BINARY_DIR}/include>
            $<INSTALL_INTERFACE:include>
        PRIVATE
            ${CMAKE_CURRENT_SOURCE_DIR}
    )
ENDIF ()

IF (MSGPACK_ENABLE_CXX AND ((CMAKE_VERSION VERSION_GREATER 3.0) OR (CMAKE_VERSION VERSION_EQUAL 3.0)))
    ADD_LIBRARY (msgpackc-cxx INTERFACE)

    TARGET_INCLUDE_DIRECTORIES (msgpackc-cxx
        INTERFACE
            $<BUILD_INTERFACE:${CMAKE_CURRENT_SOURCE_DIR}/include>
            $<BUILD_INTERFACE:${CMAKE_CURRENT_BINARY_DIR}/include>
            $<INSTALL_INTERFACE:include>
    )
ENDIF ()

IF (MSGPACK_ENABLE_STATIC)
    ADD_LIBRARY (msgpackc-static STATIC
        ${msgpackc_SOURCES}
        ${msgpackc_HEADERS}
    )

    TARGET_INCLUDE_DIRECTORIES (msgpackc-static
        PUBLIC
            $<BUILD_INTERFACE:${CMAKE_CURRENT_SOURCE_DIR}/include>
            $<BUILD_INTERFACE:${CMAKE_CURRENT_BINARY_DIR}/include>
            $<INSTALL_INTERFACE:include>
        PRIVATE
            ${CMAKE_CURRENT_SOURCE_DIR}
    )

    IF (NOT MSGPACK_ENABLE_SHARED)
        # Add alias for subdirectories
        ADD_LIBRARY (msgpackc ALIAS msgpackc-static)
    ENDIF ()

    SET_TARGET_PROPERTIES (msgpackc-static PROPERTIES OUTPUT_NAME "msgpackc")

    IF (MSGPACK_ENABLE_SHARED)
        IF (MSVC)
            SET_TARGET_PROPERTIES (msgpackc PROPERTIES IMPORT_SUFFIX "_import.lib")
        ELSEIF (MINGW)
            SET_TARGET_PROPERTIES (msgpackc PROPERTIES IMPORT_SUFFIX ".dll.a")
        ENDIF ()
    ENDIF ()
ENDIF ()

# enable regression testing
IF ("${MSGPACK_FUZZ_REGRESSION}" STREQUAL "ON" AND "${CMAKE_CXX_COMPILER_ID}" STREQUAL "Clang")
   SET (CMAKE_CXX_FLAGS "-DMSGPACK_USE_BOOST ${CMAKE_CXX_FLAGS}")
   SET (Boost_USE_MULTITHREADED      ON)
   SET (Boost_USE_STATIC_RUNTIME    OFF)

   enable_testing ()
   ADD_SUBDIRECTORY (fuzz)
   SET (MSGPACK_BUILD_EXAMPLES OFF)
ENDIF ()


IF (MSGPACK_BUILD_TESTS)
    ENABLE_TESTING ()
    # MEMORYCHECK_COMMAND_OPTIONS needs to place prior to CTEST_MEMORYCHECK_COMMAND
    SET (MEMORYCHECK_COMMAND_OPTIONS "--leak-check=full --show-leak-kinds=definite,possible --error-exitcode=1")
    FIND_PROGRAM(CTEST_MEMORYCHECK_COMMAND NAMES valgrind)
    INCLUDE(Dart)
    ADD_SUBDIRECTORY (test)
ENDIF ()

IF ("${CMAKE_CXX_COMPILER_ID}" STREQUAL "Clang" OR "${CMAKE_CXX_COMPILER_ID}" STREQUAL "GNU")
    IF (MSGPACK_ENABLE_SHARED)
        SET_PROPERTY (TARGET msgpackc APPEND_STRING PROPERTY COMPILE_FLAGS " -Wall -Wextra -DPIC")
    ENDIF ()
    IF (MSGPACK_ENABLE_STATIC)
       SET_PROPERTY (TARGET msgpackc-static APPEND_STRING PROPERTY COMPILE_FLAGS " -Wall -Wextra" )
    ENDIF ()
ENDIF ()

IF ("${CMAKE_CXX_COMPILER_ID}" STREQUAL "Clang")
    IF (MSGPACK_ENABLE_SHARED)
        SET_PROPERTY (TARGET msgpackc APPEND_STRING PROPERTY COMPILE_FLAGS " -Wno-mismatched-tags")
    ENDIF ()
    IF (MSGPACK_ENABLE_STATIC)
        SET_PROPERTY (TARGET msgpackc-static APPEND_STRING PROPERTY COMPILE_FLAGS " -Wno-mismatched-tags")
    ENDIF ()
ENDIF ()

IF ("${CMAKE_CXX_COMPILER_ID}" STREQUAL "MSVC")
    IF (CMAKE_CXX_FLAGS MATCHES "/W[0-4]")
        STRING(REGEX REPLACE "/W[0-4]" "/W3 /WX" CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS}")
    ELSE ()
        SET(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} /W3 /WX")
    ENDIF ()
ENDIF ()

IF ("${CMAKE_CXX_COMPILER_ID}" STREQUAL "MSVC90" OR "${CMAKE_CXX_COMPILER_ID}" STREQUAL "MSVC10")
    SET_SOURCE_FILES_PROPERTIES(${msgpackc_SOURCES} PROPERTIES LANGUAGE CXX)
ENDIF ()

IF ("${CMAKE_SYSTEM_PROCESSOR}" STREQUAL "sparc")
    SET (CMAKE_C_FLAGS   "-DMSGPACK_ZONE_ALIGN=8 ${CMAKE_C_FLAGS}")
    SET (CMAKE_CXX_FLAGS "-DMSGPACK_ZONE_ALIGN=8 ${CMAKE_CXX_FLAGS}")
ENDIF ()

IF (NOT DEFINED CMAKE_INSTALL_BINDIR)
    SET(CMAKE_INSTALL_BINDIR bin)
ENDIF ()

IF (NOT DEFINED CMAKE_INSTALL_LIBDIR)
    SET(CMAKE_INSTALL_LIBDIR lib)
ENDIF ()

IF (MSGPACK_BUILD_EXAMPLES)
    ADD_SUBDIRECTORY (example)
ENDIF ()

IF (MSGPACK_ENABLE_SHARED)
    SET (MSGPACK_INSTALLTARGETS msgpackc)
ENDIF ()

IF (MSGPACK_ENABLE_CXX AND ((CMAKE_VERSION VERSION_GREATER 3.0) OR (CMAKE_VERSION VERSION_EQUAL 3.0)))
    LIST (APPEND MSGPACK_INSTALLTARGETS msgpackc-cxx)
ENDIF ()

IF (MSGPACK_ENABLE_STATIC)
    LIST (APPEND MSGPACK_INSTALLTARGETS msgpackc-static)
ENDIF ()

INSTALL (TARGETS ${MSGPACK_INSTALLTARGETS} EXPORT msgpack-targets
    RUNTIME DESTINATION ${CMAKE_INSTALL_BINDIR}
    LIBRARY DESTINATION ${CMAKE_INSTALL_LIBDIR}
    ARCHIVE DESTINATION ${CMAKE_INSTALL_LIBDIR}
)
FOREACH (file ${msgpackc_HEADERS})
    GET_FILENAME_COMPONENT (dir ${file} PATH)
    INSTALL (FILES ${file} DESTINATION ${CMAKE_INSTALL_PREFIX}/${dir})
ENDFOREACH ()
IF (NOT MSVC)
    INSTALL (FILES ${CMAKE_CURRENT_BINARY_DIR}/msgpack.pc DESTINATION ${CMAKE_INSTALL_LIBDIR}/pkgconfig)
ENDIF ()

# Doxygen
FIND_PACKAGE (Doxygen)
IF (DOXYGEN_FOUND)
    LIST (APPEND Doxyfile_c_CONTENT
        COMMAND ${CMAKE_COMMAND} -E copy ${CMAKE_CURRENT_SOURCE_DIR}/Doxyfile ${CMAKE_CURRENT_BINARY_DIR}/Doxyfile_c
        COMMAND ${CMAKE_COMMAND} -E echo "FILE_PATTERNS      = *.h" >> ${CMAKE_CURRENT_BINARY_DIR}/Doxyfile_c
        COMMAND ${CMAKE_COMMAND} -E echo "OUTPUT_DIRECTORY   = doc_c" >> ${CMAKE_CURRENT_BINARY_DIR}/Doxyfile_c
        COMMAND ${CMAKE_COMMAND} -E echo "INPUT              = ${CMAKE_CURRENT_SOURCE_DIR}/include" >> ${CMAKE_CURRENT_BINARY_DIR}/Doxyfile_c
        COMMAND ${CMAKE_COMMAND} -E echo "EXTRACT_ALL        = YES" >> ${CMAKE_CURRENT_BINARY_DIR}/Doxyfile_c
        COMMAND ${CMAKE_COMMAND} -E echo "PROJECT_NAME       = \"MessagePack for C\"" >> ${CMAKE_CURRENT_BINARY_DIR}/Doxyfile_c
        COMMAND ${CMAKE_COMMAND} -E echo "STRIP_FROM_PATH    = ${CMAKE_CURRENT_SOURCE_DIR}/include" >> ${CMAKE_CURRENT_BINARY_DIR}/Doxyfile_c
    )
    IF (DOXYGEN_DOT_FOUND)
        LIST (APPEND Doxyfile_c_CONTENT
            COMMAND ${CMAKE_COMMAND} -E echo "HAVE_DOT       = YES" >> ${CMAKE_CURRENT_BINARY_DIR}/Doxyfile_c
        )
    ENDIF ()
    ADD_CUSTOM_TARGET (
        doxygen_c
        ${Doxyfile_c_CONTENT}
        COMMAND ${DOXYGEN_EXECUTABLE} ${CMAKE_CURRENT_BINARY_DIR}/Doxyfile_c
        VERBATIM
    )
    LIST (APPEND Doxyfile_cpp_CONTENT
        COMMAND ${CMAKE_COMMAND} -E copy ${CMAKE_CURRENT_SOURCE_DIR}/Doxyfile ${CMAKE_CURRENT_BINARY_DIR}/Doxyfile_cpp
        COMMAND ${CMAKE_COMMAND} -E echo "FILE_PATTERNS      = *.hpp" >> ${CMAKE_CURRENT_BINARY_DIR}/Doxyfile_cpp
        COMMAND ${CMAKE_COMMAND} -E echo "OUTPUT_DIRECTORY   = doc_cpp" >> ${CMAKE_CURRENT_BINARY_DIR}/Doxyfile_cpp
        COMMAND ${CMAKE_COMMAND} -E echo "INPUT              = ${CMAKE_CURRENT_SOURCE_DIR}/include" >> ${CMAKE_CURRENT_BINARY_DIR}/Doxyfile_cpp
        COMMAND ${CMAKE_COMMAND} -E echo "EXTRACT_ALL        = YES" >> ${CMAKE_CURRENT_BINARY_DIR}/Doxyfile_cpp
        COMMAND ${CMAKE_COMMAND} -E echo "STRIP_FROM_PATH    = ${CMAKE_CURRENT_SOURCE_DIR}/include" >> ${CMAKE_CURRENT_BINARY_DIR}/Doxyfile_cpp
    )
    IF (DOXYGEN_DOT_FOUND)
        LIST (APPEND Doxyfile_cpp_CONTENT
            COMMAND ${CMAKE_COMMAND} -E echo "HAVE_DOT       = YES" >> ${CMAKE_CURRENT_BINARY_DIR}/Doxyfile_cpp
        )
    ENDIF ()
    ADD_CUSTOM_TARGET (
        doxygen_cpp
        ${Doxyfile_cpp_CONTENT}
        COMMAND ${CMAKE_COMMAND} -E echo "PROJECT_NAME       = \"MessagePack for C++\"" >> ${CMAKE_CURRENT_BINARY_DIR}/Doxyfile_cpp
        COMMAND ${DOXYGEN_EXECUTABLE} ${CMAKE_CURRENT_BINARY_DIR}/Doxyfile_cpp
        VERBATIM
    )
    ADD_CUSTOM_TARGET (
        doxygen
        DEPENDS doxygen_c doxygen_cpp
    )
ENDIF ()

INCLUDE (CMakePackageConfigHelpers)

SET (CMAKE_INSTALL_CMAKEDIR "${CMAKE_INSTALL_LIBDIR}/cmake/msgpack")

WRITE_BASIC_PACKAGE_VERSION_FILE (
    msgpack-config-version.cmake
    VERSION ${VERSION}
    COMPATIBILITY SameMajorVersion
)

# IF (NOT CMAKE_VERSION VERSION_LESS 3.0)
#     EXPORT (EXPORT msgpack-targets
#         NAMESPACE Msgpack::
#         FILE "${CMAKE_CURRENT_BINARY_DIR}/msgpack-targets.cmake"
#     )
# ENDIF ()

CONFIGURE_PACKAGE_CONFIG_FILE (msgpack-config.cmake.in
    msgpack-config.cmake
    INSTALL_DESTINATION "${CMAKE_INSTALL_CMAKEDIR}"
)

INSTALL (EXPORT msgpack-targets
    NAMESPACE Msgpack::
    FILE
        msgpack-targets.cmake
    DESTINATION
        "${CMAKE_INSTALL_CMAKEDIR}"
)

INSTALL (
    FILES
        "${CMAKE_CURRENT_BINARY_DIR}/msgpack-config.cmake"
        "${CMAKE_CURRENT_BINARY_DIR}/msgpack-config-version.cmake"
    DESTINATION
        "${CMAKE_INSTALL_CMAKEDIR}"
)
