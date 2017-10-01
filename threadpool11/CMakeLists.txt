CreateTarget(TARGET threadpool11)

if (USE_SYSTEM_BOOST)
    find_package(Boost 1.62 REQUIRED)
    set(BOOST_DEPENDENCIES "")

    AddDependency(
        TARGET threadpool11
        DEPENDENCY Boost
    )
else()
    set(BOOST_DEPENDENCIES
        # Lockfree dependencies
        lockfree core config assert static_assert type_traits mpl preprocessor
        array throw_exception functional align parameter atomic detail
        utility predef winapi
    )

    foreach (dep ${BOOST_DEPENDENCIES})
        RequireExternal(
            TARGET threadpool11
            EXCLUDE
            SKIP_BUILD
            SKIP_LINK
            MODULE "boostorg/${dep}:develop"
            INC_PATH "include"
        )
    endforeach()
endif()

AddToIncludes(
    TARGET threadpool11
    INC_PATH ${CMAKE_CURRENT_SOURCE_DIR}/include
)

AddToSources(
    TARGET threadpool11
    SRC_PATH ${CMAKE_CURRENT_SOURCE_DIR}/src
    GLOB_SEARCH ".hpp;.cpp"
)

if (CMAKE_COMPILER_IS_GNUCXX)
    AddDependency(
        TARGET threadpool11
        DEPENDENCY pthread
    )
endif()

set(DEFINES threadpool11_EXPORTING)

BuildNow(
    TARGET threadpool11
    DEFINES ${DEFINES}
    STATIC_LIB
    OUTPUT_NAME threadpool11
)

if("${CMAKE_CXX_COMPILER_ID}" STREQUAL "GNU" OR "${CMAKE_CXX_COMPILER_ID}" STREQUAL "Clang")
  set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -std=c++11 -pthread")
  set(CMAKE_CXX_FLAGS_RELEASE "${CMAKE_EXE_LINK_FLAGS_RELEASE} -O2")

  set(CMAKE_CXX_FLAGS_PERF "${CMAKE_CXX_FLAGS_RELEASE} -Wno-inline -pg")

  set(CMAKE_STATIC_LINKER_FLAGS_PERF "${CMAKE_LINKER_FLAGS_RELEASE} -pg")

  set(CMAKE_EXE_LINK_FLAGS "${CMAKE_EXE_LINK_FLAGS} -pthread")
  set(CMAKE_EXE_LINK_FLAGS_RELEASE "${CMAKE_EXE_LINK_FLAGS_RELEASE} -pthread -O2")
elseif(MSVC)
  set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS_RELEASE} /GL")
  set(CMAKE_STATIC_LINKER_FLAGS "${CMAKE_STATIC_LINKER_FLAGS} /LTCG")
endif()