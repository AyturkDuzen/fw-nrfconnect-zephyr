# SPDX-License-Identifier: Apache-2.0

# '-mcmse' enables the generation of code for the Secure state of the ARMv8-M
# Security Extensions. This option is required when building a Secure firmware.
zephyr_compile_options_ifdef(CONFIG_ARM_SECURE_FIRMWARE -mcmse)

set(veneers_lib_path ${CMAKE_BINARY_DIR}/${CONFIG_ARM_ENTRY_VENEERS_LIB_NAME})

if(CONFIG_ARM_FIRMWARE_HAS_SECURE_ENTRY_FUNCS)

  # --out-implib and --cmse-implib instruct the linker to produce
  # an import library that consists of a relocatable file containing
  # only a symbol table with the entry veneers. The library may be used
  # when building a Non-Secure image which shall have access to Secure
  # Entry functions.
  zephyr_ld_options(
    ${LINKERFLAGPREFIX},--out-implib=${veneers_lib_path}
  )

  zephyr_ld_options(
    ${LINKERFLAGPREFIX},--cmse-implib
  )

  # Indicate that the entry veneers library file is created during linking of this firmware.
  set_property(
    GLOBAL APPEND PROPERTY
    ${IMAGE}extra_post_build_byproducts
    ${veneers_lib_path}
    )

  # Add wrapper target for the benefit of make.
  add_custom_target(nsc_veneers
    DEPENDS
    ${veneers_lib_path}
    ${ZEPHYR_FINAL_EXECUTABLE})
endif()

if (CONFIG_ARM_FIRMWARE_USES_SECURE_ENTRY_FUNCS)
  # Link the entry veneers library file with the Non-Secure Firmware that needs it.
  zephyr_link_libraries(${veneers_lib_path})

  # Add dependency for the benefit of make.
  add_dependencies(${IMAGE}zephyr_interface nsc_veneers)
endif()

zephyr_sources_ifdef(CONFIG_ARM_SECURE_FIRMWARE arm_core_tz.c)
