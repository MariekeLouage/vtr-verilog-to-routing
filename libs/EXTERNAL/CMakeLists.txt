#Manually synchronized external libraries
add_subdirectory(libpugixml)

#External libraries synchronized with 'git subtree'
add_subdirectory(libargparse)
add_subdirectory(libsdcparse)
add_subdirectory(libblifparse)
add_subdirectory(libtatum)
add_subdirectory(libcatch2)

#VPR_USE_EZGL is initialized in the root CMakeLists.
#compile libezgl only if the user asks for or has its dependencies installed.
if(VPR_USE_EZGL STREQUAL "on")
    add_subdirectory(libezgl)
endif()

# The VTR root CMakeFile initializes the ODIN_USE_YOSYS or WITH_YOSYS
# Yosys is compiled only if the user ask for it
if(${ODIN_USE_YOSYS} OR ${WITH_YOSYS})
    add_subdirectory(libyosys)
endif()

if(${VTR_ENABLE_CAPNPROTO})
    # Override default policy for capnproto (CMake policy version 3.1)
    # Enable new IPO variables
    set(CMAKE_POLICY_DEFAULT_CMP0069 NEW)

    # Enable option overrides via variables
    set(CMAKE_POLICY_DEFAULT_CMP0077 NEW)

    # Re-enable CXX extensions for capnproto.
    set(CMAKE_CXX_EXTENSIONS ON)

    # Disable capnproto tests
    set(BUILD_TESTING OFF)

    #Since capnproto is an externally developed library/tool, we suppress all compiler warnings
    CHECK_CXX_COMPILER_FLAG("-w" CXX_COMPILER_SUPPORTS_-w)
    if(CXX_COMPILER_SUPPORTS_-w)
        add_compile_options("-w")
    endif()

    add_subdirectory(capnproto EXCLUDE_FROM_ALL)

    #Some capnproto kj headers (e.g. filesystem.h) generate warnings, treat them as system headers to suppress warnings
    #We suppress them here since we include the capnproto sub-tree as is and do not modify its CMakeLists.txts
    target_include_directories(kj SYSTEM INTERFACE
      $<BUILD_INTERFACE:${CMAKE_CURRENT_SOURCE_DIR}/capnproto/c++/src>
      $<INSTALL_INTERFACE:include>
    )
endif()

# Some catch2 headers generate warnings, so treat them as system headers to suppress warnings
target_include_directories(Catch2
  SYSTEM
  PUBLIC
    $<BUILD_INTERFACE:${CMAKE_CURRENT_SOURCE_DIR}/libcatch2/src>
    $<INSTALL_INTERFACE:include>
)

