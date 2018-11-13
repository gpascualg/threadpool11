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
        utility predef winapi tti fusion
    )

    foreach (dep ${BOOST_DEPENDENCIES})
        RequireExternal(
            TARGET threadpool11
            EXCLUDE
            SKIP_CONFIGURE
            SKIP_BUILD
            MODULE "boostorg/${dep}:develop"
            INSTALL_INCLUDE
        )
    endforeach()
endif()

ResolveExternal(TARGET threadpool11)
if (threadpool11_IS_RESOLVED)
    AddToIncludes(
        TARGET threadpool11
        INC_PATH ${CMAKE_CURRENT_SOURCE_DIR}/include
    )

    AddToSources(
        TARGET threadpool11
        SOURCES ${CMAKE_CURRENT_SOURCE_DIR}/src
        GLOB_SEARCH ".hpp;.cpp"
    )

    AddLibrary(
        TARGET threadpool11
        PLATFORM UNIX
        LIBRARY pthread
    )

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

    # INSTALL
    set_property(TARGET threadpool11 PROPERTY INTERFACE_INCLUDE_DIRECTORIES "$<INSTALL_INTERFACE:include>")

    include(CMakePackageConfigHelpers)
    set(config_install_dir lib/cmake/threadpool11)
    set(version_config ${PROJECT_BINARY_DIR}/threadpool11-config-version.cmake)
    set(project_config ${PROJECT_BINARY_DIR}/threadpool11-config.cmake)
    set(targets_export_name threadpool11-targets)

    # Generate the version, config and target files into the build directory.
    write_basic_package_version_file(
        ${version_config}
        VERSION 0.0.1
        COMPATIBILITY AnyNewerVersion)
        configure_package_config_file(
        ${PROJECT_SOURCE_DIR}/config.cmake.in
        ${project_config}
        INSTALL_DESTINATION ${config_install_dir})
    export(TARGETS threadpool11 FILE ${PROJECT_BINARY_DIR}/${targets_export_name}.cmake)

    install(FILES include/threadpool11/threadpool11.hpp DESTINATION include/threadpool11)
    install(FILES include/threadpool11/worker.hpp DESTINATION include/threadpool11)
    install(FILES include/threadpool11/pool.hpp DESTINATION include/threadpool11)
    install(FILES include/threadpool11/utility.hpp DESTINATION include/threadpool11)
    install(TARGETS threadpool11 DESTINATION lib)
    install(FILES ${project_config} ${version_config} DESTINATION ${config_install_dir})
    install(TARGETS threadpool11 EXPORT threadpool11-targets DESTINATION lib)
    install(EXPORT threadpool11-targets DESTINATION ${config_install_dir})
endif()
