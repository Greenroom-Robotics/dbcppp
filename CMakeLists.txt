cmake_minimum_required(VERSION 3.12)

project("dbcppp" VERSION 3.8.0)

# CONFIGURATION

set(CMAKE_CXX_STANDARD 17)
option(build_kcd "Enable support for KCD parsing" ON)
option(build_tools "Build dbcppp utility application" OFF)

option(build_tests "Build tests" OFF)
option(build_examples "Build examples" OFF)


# DEPENDENCIES & Requirements

find_package(ament_cmake REQUIRED)
find_package(Boost REQUIRED)
find_package(cxxopts REQUIRED)

# kcd is an xml based can database
message("kcd enabled: ${build_kcd}")

if(build_kcd)
    add_compile_definitions(ENABLE_KCD)

    # LibXml2

    find_package(LibXml2)

    include_directories(${LIBXML2_INCLUDE_DIR})

    if (NOT LibXml2_FOUND)
        message(WARNING "LibXml2 was not found. Using libdbcppp LibXml2 (third-party/libxml2)")
        set(LIBXML2_WITH_ICONV OFF)
        set(LIBXML2_WITH_LZMA OFF)
        set(LIBXML2_WITH_PYTHON OFF)
        set(LIBXML2_WITH_ZLIB OFF)
        set(LIBXML2_WITH_TESTS OFF)
        add_subdirectory(third-party/libxml2)
    endif()

    # libxmlmm (no find package because it is certainly not installed
    find_package(libxmlmm REQUIRED)

    # add_subdirectory("third-party/libxmlmm")
    # target_include_directories(libxmlmm PUBLIC third-party/libxmlmm/libxmlmm ${LIBXML2_INCLUDE_DIR})
endif()


# CREATE LIBRARY

file(GLOB include "include/dbcppp/*.h")
file(GLOB headers "src/*.h")
file(GLOB sources "src/*.cpp")

add_library(${PROJECT_NAME} SHARED ${include} ${headers} ${sources})


# CONFIGURE LIBRARY

target_link_libraries(${PROJECT_NAME} ${LIBXML2_LIBRARIES} )

if (build_kcd)

ament_target_dependencies(${PROJECT_NAME}
    libxmlmm
)

endif()

set_target_properties(${PROJECT_NAME} PROPERTIES
    SOVERSION ${PROJECT_VERSION}
    PUBLIC_HEADER "${include}"
)

target_include_directories(${PROJECT_NAME} PUBLIC
    $<BUILD_INTERFACE:${CMAKE_CURRENT_SOURCE_DIR}/include>
    $<INSTALL_INTERFACE:include>
)

# INSTALL LIBRARY

install(TARGETS ${PROJECT_NAME} EXPORT ${PROJECT_NAME}
  ARCHIVE DESTINATION lib
  LIBRARY DESTINATION lib
  RUNTIME DESTINATION lib/${PROJECT_NAME}
)

install(
    DIRECTORY include/
    DESTINATION include
)
# ADDITIONAL: Tools, Tests & Examples

if (build_tools)
    # add_subdirectory(third-party/cxxopts)
    add_subdirectory(tools/dbcppp)
endif()

if (build_tests)
  add_subdirectory(tests)
endif()

if (build_examples)
  add_subdirectory(examples)
endif()

# PACKAGE (useful for debugging install, use make package)

set(CPACK_INCLUDE_TOPLEVEL_DIRECTORY NO)
set(CPACK_VERBATIM_VARIABLES YES)
set(CPACK_SOURCE_GENERATOR "TGZ")
set(CPACK_SOURCE_IGNORE_FILES \\.git/ build/ ".*~$")
include(CPack)

ament_export_include_directories(include)
ament_export_targets(${PROJECT_NAME})
ament_package()
