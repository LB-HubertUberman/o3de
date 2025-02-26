#
# Copyright (c) Contributors to the Open 3D Engine Project.
# For complete copyright and license terms please see the LICENSE at the root of this distribution.
#
# SPDX-License-Identifier: Apache-2.0 OR MIT
#
#

# Currently we are in the Code folder: ${CMAKE_CURRENT_LIST_DIR}
# Get the platform specific folder ${pal_dir} for the current folder: ${CMAKE_CURRENT_LIST_DIR}/Platform/${PAL_PLATFORM_NAME}
# Note: o3de_pal_dir will take care of the details for us, as this may be a restricted platform
#       in which case it will see if that platform is present here or in the restricted folder.
#       i.e. It could here in our gem : Gems/StreamerProfiler/Code/Platform/<platorm_name>  or
#            <restricted_folder>/<platform_name>/Gems/StreamerProfiler/Code
o3de_pal_dir(pal_dir ${CMAKE_CURRENT_LIST_DIR}/Platform/${PAL_PLATFORM_NAME} "${gem_restricted_path}" "${gem_path}" "${gem_parent_relative_path}")

# Now that we have the platform abstraction layer (PAL) folder for this folder, thats where we will find the
# traits for this platform. Traits for a platform are defines for things like whether or not something in this gem
# is supported by this platform.
include(${pal_dir}/PAL_${PAL_PLATFORM_NAME_LOWERCASE}.cmake)

# The StreamerProfiler.Private.Object target is an internal target
# It should not be used outside of this Gems CMakeLists.txt
ly_add_target(
    NAME StreamerProfiler.Private.Object STATIC
    NAMESPACE Gem
    FILES_CMAKE
        streamerprofiler_private_files.cmake
        ${pal_dir}/streamerprofiler_private_files.cmake
    TARGET_PROPERTIES
        O3DE_PRIVATE_TARGET TRUE
    INCLUDE_DIRECTORIES
        PRIVATE
            Source
    BUILD_DEPENDENCIES
        PUBLIC
            AZ::AzCore
            Gem::ImGui.imguilib
)

# Here add StreamerProfiler target, it depends on the Private Object library and Public API interface
ly_add_target(
    NAME StreamerProfiler ${PAL_TRAIT_MONOLITHIC_DRIVEN_MODULE_TYPE}
    NAMESPACE Gem
    FILES_CMAKE
        streamerprofiler_shared_files.cmake
        ${pal_dir}/streamerprofiler_shared_files.cmake
    INCLUDE_DIRECTORIES
        PRIVATE
            Source
    BUILD_DEPENDENCIES
        PRIVATE
            Gem::StreamerProfiler.Private.Object
)

# By default, we will specify that the above target StreamerProfiler would be used by
# Client and Server type targets when this gem is enabled.  If you don't want it
# active in Clients or Servers by default, delete one of both of the following lines:
ly_create_alias(NAME StreamerProfiler.Clients NAMESPACE Gem TARGETS Gem::StreamerProfiler)
ly_create_alias(NAME StreamerProfiler.Servers NAMESPACE Gem TARGETS Gem::StreamerProfiler)

################################################################################
# Tests
################################################################################
# See if globally, tests are supported
if(PAL_TRAIT_BUILD_TESTS_SUPPORTED)
    # We globally support tests, see if we support tests on this platform for StreamerProfiler.Tests
    if(PAL_TRAIT_STREAMERPROFILER_TEST_SUPPORTED)
        # We support StreamerProfiler.Tests on this platform, add dependency on the Private Object target
        ly_add_target(
            NAME StreamerProfiler.Tests ${PAL_TRAIT_TEST_TARGET_TYPE}
            NAMESPACE Gem
            FILES_CMAKE
                streamerprofiler_tests_files.cmake
            INCLUDE_DIRECTORIES
                PRIVATE
                    Tests
                    Source
            BUILD_DEPENDENCIES
                PRIVATE
                    AZ::AzTest
                    AZ::AzFramework
                    Gem::StreamerProfiler.Private.Object
        )

        # Add StreamerProfiler.Tests to googletest
        # Commented out as currently there are no StreamerProfiler tests
        # ly_add_googletest(
            # NAME Gem::StreamerProfiler.Tests
        # )
    endif()
endif()
