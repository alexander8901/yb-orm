
cmake_minimum_required (VERSION 2.8)
project (tutorial)


if (UNIX)
    find_path (BOOST_INCLUDEDIR boost/lexical_cast.hpp /usr/include)
else ()
    find_path (BOOST_INCLUDEDIR boost/lexical_cast.hpp ${YBORM_ROOT}/include)
endif ()

# find boost + libxml2

if (MSVC)
    set (Boost_USE_STATIC_LIBS ON)
else ()
    set (Boost_USE_STATIC_LIBS OFF)
endif ()
set (Boost_USE_MULTITHREADED ON)
set (Boost_NO_SYSTEM_PATHS ON)
#set (Boost_DEBUG ON)
set (Boost_ADDITIONAL_VERSIONS "1.33.1" "1.38" "1.30.0" "1.46.1")
if (UNIX)
    set (BOOST_LIBRARYDIR "/usr/lib")
else ()
    set (BOOST_LIBRARYDIR "${YBORM_ROOT}/lib")
endif ()

find_package (Boost 1.33.1 COMPONENTS system thread date_time chrono)
if ((NOT Boost_DATE_TIME_LIBRARY) OR (NOT Boost_THREAD_LIBRARY))
    message (FATAL_ERROR "Boost not found")
endif ()
if (Boost_SYSTEM_LIBRARY)
    set (YB_BOOST_LIBS ${Boost_SYSTEM_LIBRARY})
endif ()
if (Boost_CHRONO_LIBRARY)
    set (YB_BOOST_LIBS ${YB_BOOST_LIBS} ${Boost_CHRONO_LIBRARY})
endif ()
set (YB_BOOST_LIBS ${YB_BOOST_LIBS} ${Boost_THREAD_LIBRARY}
                   ${Boost_DATE_TIME_LIBRARY})

# find includes

if (UNIX)
    find_path (ICONV_INCLUDES iconv.h /usr/include)
else ()
    find_path (ICONV_INCLUDES iconv.h ${YBORM_ROOT}/include)
endif ()

if (UNIX)
    find_path (LIBXML2_INCLUDES libxml/xmlwriter.h /usr/include/libxml2)
else ()
    find_path (LIBXML2_INCLUDES libxml/xmlwriter.h ${YBORM_ROOT}/include)
endif ()

# find libs

if (MSVC)
    find_library (ICONV_LIBS iconv ${YBORM_ROOT}/lib)
elseif (BORLAND)
    find_library (ICONV_LIBS iconvomf ${YBORM_ROOT}/lib)
endif ()

if (UNIX)
    find_library (LIBXML2_LIBS xml2)
elseif (MSVC)
    find_library (LIBXML2_LIBS libxml2 ${YBORM_ROOT}/lib)
elseif (BORLAND)
    find_library (LIBXML2_LIBS libxml2omf ${YBORM_ROOT}/lib)
else ()
    find_library (LIBXML2_LIBS libxml2 ${YBORM_ROOT}/bin)
endif ()

if (UNIX)
    find_library (ODBC_LIBS odbc)
elseif (BORLAND)
    find_library (ODBC_LIBS odbc32 c:/Borland/BCC55/Lib/PSDK)
elseif (MSVC)
    find_library (ODBC_LIBS odbc32)
elseif (MINGW)
    set (ODBC_LIBS -lodbc32 CACHE STRING "odbc32 lib" FORCE)
endif ()

if (SQLITE3_INCLUDEDIR)
    find_path (SQLITE3_INCLUDEDIR sqlite3.h ${SQLITE3_INCLUDEDIR})
elseif (UNIX)
    find_path (SQLITE3_INCLUDEDIR sqlite3.h /usr/include)
else ()
    find_path (SQLITE3_INCLUDEDIR sqlite3.h ${YBORM_ROOT}/include)
endif ()

if (SQLITE3_LIBDIR)
    find_library (SQLITE3_LIBS sqlite3 ${SQLITE3_LIBDIR})
elseif (UNIX)
    find_library (SQLITE3_LIBS sqlite3)
elseif (BORLAND)
    find_library (SQLITE3_LIBS sqlite3omf)
else ()
    find_library (SQLITE3_LIBS sqlite3 ${YBORM_ROOT}/lib ${YBORM_ROOT}/bin)
endif ()

if (SQLITE3_INCLUDEDIR AND SQLITE3_LIBS)
if (BORLAND)
    add_definitions (-DYB_USE_SQLITE3)
else ()
    add_definitions (-D YB_USE_SQLITE3)
endif ()
    set (USE_SQLITE3 ON)
else ()
    set (SQLITE3_LIBS "")
    set (SQLITE3_INCLUDEDIR "")
    set (USE_SQLITE3 OFF)
endif ()

if (SOCI_INCLUDEDIR)
    find_path (SOCI_INCLUDEDIR soci.h ${SOCI_INCLUDEDIR})
elseif (UNIX)
    find_path (SOCI_INCLUDEDIR soci.h /usr/include/soci /usr/local/include/soci)
else ()
    find_path (SOCI_INCLUDEDIR soci.h ${YBORM_ROOT}/include)
endif ()

if (SOCI_LIBDIR)
    find_library (SOCI_LIBS soci_core ${SOCI_LIBDIR})
elseif (UNIX)
    find_library (SOCI_LIBS soci_core)
else ()
    find_library (SOCI_LIBS soci ${YBORM_ROOT}/lib ${YBORM_ROOT}/bin)
endif ()

if (SOCI_INCLUDEDIR AND SOCI_LIBS)
    add_definitions (-D YB_USE_SOCI)
    set (USE_SOCI ON)
else ()
    set (SOCI_LIBS "")
    set (SOCI_INCLUDEDIR "")
    set (USE_SOCI OFF)
endif ()

if (BOOST_INCLUDEDIR AND (NOT BORLAND))
    add_definitions (-D YB_USE_TUPLE)
endif()

function (check_yborm_root YBORM_ROOT)
    if (EXISTS ${YBORM_ROOT})
        find_path (YBORM_INCLUDES orm/sql_driver.h ${YBORM_ROOT}/include/yb)
        find_library (YBUTIL_LIB ybutil ${YBORM_ROOT}/lib)
        find_library (YBORM_LIB yborm ${YBORM_ROOT}/lib)
        if (YBORM_INCLUDES AND YBUTIL_LIB AND YBORM_LIB)
	    message (STATUS "YB.ORM found: ${YBORM_ROOT}")
        endif ()
    endif ()
endfunction ()

if (((NOT YBORM_INCLUDES) OR (NOT YBUTIL_LIB) OR (NOT YBORM_LIB))
        AND YBORM_ROOT)
    check_yborm_root (${YBORM_ROOT})
endif ()
if ((NOT YBORM_INCLUDES) OR (NOT YBUTIL_LIB) OR (NOT YBORM_LIB))
    if (UNIX)
        check_yborm_root (/usr/local)
    else ()
        check_yborm_root (c:/yborm)
    endif ()
endif ()
if ((NOT YBORM_INCLUDES) OR (NOT YBUTIL_LIB) OR (NOT YBORM_LIB))
    message(SEND_ERROR "YB.ORM not found")
endif ()

if (YBUTIL_LIB)
    find_library (YBUTIL_LIB ${YBUTIL_LIB})
endif ()

if (YBORM_LIB)
    find_library (YBORM_LIB ${YBORM_LIB})
endif ()

add_subdirectory (domain)


include_directories (
    ${ICONV_INCLUDES}
    ${LIBXML2_INCLUDES}
	${BOOST_INCLUDEDIR}
    ${YBORM_INCLUDES}
    ${CMAKE_CURRENT_BINARY_DIR})

add_executable (tut1 tut1.cpp
    ${CMAKE_CURRENT_BINARY_DIR}/domain/Client.cpp
    ${CMAKE_CURRENT_BINARY_DIR}/domain/Order.cpp)

add_executable (tut2 tut2.cpp
    ${CMAKE_CURRENT_BINARY_DIR}/domain/Client.cpp
    ${CMAKE_CURRENT_BINARY_DIR}/domain/Order.cpp)

add_executable (tut2_new tut2_new.cpp)

add_executable (tut3 tut3.cpp
    ${CMAKE_CURRENT_BINARY_DIR}/domain/Client.cpp
    ${CMAKE_CURRENT_BINARY_DIR}/domain/Order.cpp)

add_executable (tut4 tut4.cpp
    ${CMAKE_CURRENT_BINARY_DIR}/domain/ProductGroup.cpp
    ${CMAKE_CURRENT_BINARY_DIR}/domain/Product.cpp)

add_executable (tut4_new tut4_new.cpp)

set_source_files_properties (
    ${CMAKE_CURRENT_BINARY_DIR}/domain/Client.cpp
    ${CMAKE_CURRENT_BINARY_DIR}/domain/Order.cpp
    ${CMAKE_CURRENT_BINARY_DIR}/domain/ProductGroup.cpp
    ${CMAKE_CURRENT_BINARY_DIR}/domain/Product.cpp
    ${CMAKE_CURRENT_BINARY_DIR}/tut1.sql
    ${CMAKE_CURRENT_BINARY_DIR}/tut4.sql
    PROPERTIES GENERATED 1)

add_custom_command (
    OUTPUT tut1.sql
    COMMAND ${YBORM_ROOT}/bin/yborm_gen --ddl ${PROJECT_SOURCE_DIR}/tut1.xml SQLITE ${CMAKE_CURRENT_BINARY_DIR}/tut1.sql
    DEPENDS ${PROJECT_SOURCE_DIR}/tut1.xml
)

add_custom_command (
    OUTPUT tut4.sql
    COMMAND ${YBORM_ROOT}/bin/yborm_gen --ddl ${PROJECT_SOURCE_DIR}/tut4.xml SQLITE ${CMAKE_CURRENT_BINARY_DIR}/tut4.sql
    DEPENDS ${PROJECT_SOURCE_DIR}/tut4.xml
)

add_custom_target (gen_sql_tut1 DEPENDS ${CMAKE_CURRENT_BINARY_DIR}/tut1.sql)
add_custom_target (gen_sql_tut4 DEPENDS ${CMAKE_CURRENT_BINARY_DIR}/tut4.sql)

add_dependencies (tut1 gen_cpp_tut1)
add_dependencies (tut1 gen_sql_tut1)
add_dependencies (tut2 gen_cpp_tut1)
add_dependencies (tut2 gen_sql_tut1)
add_dependencies (tut3 gen_cpp_tut1)
add_dependencies (tut3 gen_sql_tut1)
add_dependencies (tut4 gen_cpp_tut4)
add_dependencies (tut4 gen_sql_tut4)

target_link_libraries (tut1 ${YBORM_LIB} ${YBUTIL_LIB} ${LIBXML2_LIBS}
    ${YB_BOOST_LIBS} ${ODBC_LIBS} ${SQLITE3_LIBS} ${SOCI_LIBS})

target_link_libraries (tut2 ${YBORM_LIB} ${YBUTIL_LIB} ${LIBXML2_LIBS}
    ${YB_BOOST_LIBS} ${ODBC_LIBS} ${SQLITE3_LIBS} ${SOCI_LIBS})

target_link_libraries (tut2_new ${YBORM_LIB} ${YBUTIL_LIB} ${LIBXML2_LIBS}
    ${YB_BOOST_LIBS} ${ODBC_LIBS} ${SQLITE3_LIBS} ${SOCI_LIBS})

target_link_libraries (tut3 ${YBORM_LIB} ${YBUTIL_LIB} ${LIBXML2_LIBS}
    ${YB_BOOST_LIBS} ${ODBC_LIBS} ${SQLITE3_LIBS} ${SOCI_LIBS})

target_link_libraries (tut4 ${YBORM_LIB} ${YBUTIL_LIB} ${LIBXML2_LIBS}
    ${YB_BOOST_LIBS} ${ODBC_LIBS} ${SQLITE3_LIBS} ${SOCI_LIBS})

target_link_libraries (tut4_new ${YBORM_LIB} ${YBUTIL_LIB} ${LIBXML2_LIBS}
    ${YB_BOOST_LIBS} ${ODBC_LIBS} ${SQLITE3_LIBS} ${SOCI_LIBS})

install (TARGETS tut1 tut2 tut2_new tut3 tut4 tut4_new DESTINATION examples)
install (FILES tut1.xml tut4.xml ${CMAKE_CURRENT_BINARY_DIR}/tut1.sql
    ${CMAKE_CURRENT_BINARY_DIR}/tut4.sql  DESTINATION examples)

