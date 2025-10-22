cmake_minimum_required(VERSION 3.10)

project(smartdns C CXX)

# Set version
set(SMARTDNS_VERSION "1.0.0")

# Set C standard
set(CMAKE_C_STANDARD 99)
set(CMAKE_C_STANDARD_REQUIRED ON)

# Set C++ standard
set(CMAKE_CXX_STANDARD 11)
set(CMAKE_CXX_STANDARD_REQUIRED ON)

# Default build type
if(NOT CMAKE_BUILD_TYPE)
    set(CMAKE_BUILD_TYPE "Release")
endif()

# Options
option(WITH_UI "Build with smartdns-ui plugin" OFF)
option(DEBUG "Enable debug build" OFF)
option(STATIC "Build static binary" OFF)

# Compiler flags
if(DEBUG)
    add_compile_options(-g -DDEBUG)
else()
    add_compile_options(-O2)
endif()

add_compile_options(
    -Wall
    -Wstrict-prototypes
    -fno-omit-frame-pointer
    -Wstrict-aliasing
    -funwind-tables
    -Wmissing-prototypes
    -Wshadow
    -Wextra
    -Wno-unused-parameter
    -Wno-implicit-fallthrough
)

# Check for unwind.h
include(CheckIncludeFile)
check_include_file(unwind.h HAVE_UNWIND_H)

if(HAVE_UNWIND_H)
    add_compile_definitions(HAVE_UNWIND_BACKTRACE)
endif()

# Check for atomic operations
include(CheckCSourceCompiles)
check_c_source_compiles("
    #include <stdint.h>
    int main() {
        uint64_t value = 0;
        __atomic_add_fetch(&value, 1, __ATOMIC_SEQ_CST);
        __atomic_load_n(&value, __ATOMIC_SEQ_CST);
        return 0;
    }
" HAVE_ATOMIC_BUILTINS)

if(NOT HAVE_ATOMIC_BUILTINS)
    add_compile_definitions(USE_ATOMIC)
    set(ATOMIC_LIBRARIES atomic)
endif()

# Add include directories
include_directories(src/include)

# Add subdirectory for main application
add_subdirectory(src)

# Add subdirectory for plugins
if(WITH_UI)
    add_subdirectory(plugin/smartdns-ui)
endif()

# Install targets
install(FILES etc/default/smartdns DESTINATION /etc/default)
install(FILES etc/init.d/smartdns DESTINATION /etc/init.d
        PERMISSIONS OWNER_READ OWNER_WRITE OWNER_EXECUTE 
                    GROUP_READ GROUP_EXECUTE 
                    WORLD_READ WORLD_EXECUTE)
install(FILES etc/smartdns/smartdns.conf DESTINATION /etc/smartdns)
install(FILES systemd/smartdns.service DESTINATION /lib/systemd/system)