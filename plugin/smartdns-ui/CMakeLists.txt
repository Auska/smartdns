# Check if we're building with Cargo
find_package(Cargo QUIET)

if(CARGO_FOUND)
    # Configure Cargo build
    set(CARGO_MANIFEST_PATH "${CMAKE_CURRENT_SOURCE_DIR}/Cargo.toml")
    
    # Set build type
    if(CMAKE_BUILD_TYPE STREQUAL "Debug")
        set(CARGO_BUILD_ARGS "")
        set(CARGO_BUILD_PATH "target/debug")
    else()
        set(CARGO_BUILD_ARGS "--release")
        set(CARGO_BUILD_PATH "target/release")
    endif()
    
    # Build the Rust plugin
    add_custom_command(
        OUTPUT "${CMAKE_CURRENT_BINARY_DIR}/libsmartdns_ui.so"
        COMMAND ${CMAKE_COMMAND} -E env
            CXXFLAGS= CFLAGS= MAKEFLAGS=
            cargo build ${CARGO_BUILD_ARGS} --features "build-release"
        COMMAND ${CMAKE_COMMAND} -E copy
            "${CMAKE_CURRENT_SOURCE_DIR}/${CARGO_BUILD_PATH}/libsmartdns_ui.so"
            "${CMAKE_CURRENT_BINARY_DIR}/libsmartdns_ui.so"
        WORKING_DIRECTORY ${CMAKE_CURRENT_SOURCE_DIR}
        COMMENT "Building smartdns-ui plugin with Cargo"
        VERBATIM
    )
    
    # Create custom target
    add_custom_target(smartdns_ui ALL
        DEPENDS "${CMAKE_CURRENT_BINARY_DIR}/libsmartdns_ui.so"
    )
    
    # Install the plugin
    install(FILES "${CMAKE_CURRENT_BINARY_DIR}/libsmartdns_ui.so"
            DESTINATION /usr/lib/smartdns)
else()
    message(WARNING "Cargo not found. Skipping smartdns-ui plugin build.")
endif()