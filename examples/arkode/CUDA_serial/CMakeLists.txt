# ---------------------------------------------------------------
# Programmer(s): Cody J. Balos @ LLNL
# ---------------------------------------------------------------
# SUNDIALS Copyright Start
# Copyright (c) 2002-2019, Lawrence Livermore National Security
# and Southern Methodist University.
# All rights reserved.
#
# See the top-level LICENSE and NOTICE files for details.
#
# SPDX-License-Identifier: BSD-3-Clause
# SUNDIALS Copyright End
# ---------------------------------------------------------------
# CMakeLists.txt file for ARKode CUDA examples
# ---------------------------------------------------------------

# Example lists are tuples "name\;nodes\;tasks\;type" where the
# type is develop for examples excluded from 'make test' in releases

if(SUNDIALS_INDEX_SIZE MATCHES "32")
  # Examples using cuSolverSP linear solvers
  set(ARKODE_examples_cusp
    "ark_brusselator1D_FEM_cusolver\;\;\;exclude"
    )
endif()

# Specify libraries to link against (through the target that was used to
# generate them) based on the value of the variable LINK_LIBRARY_TYPE
if(LINK_LIBRARY_TYPE MATCHES "static")
  set(ARKODE_LIB sundials_arkode_static)
  set(SUNLS_LIB sundials_sunlinsolcusolversp_static)
else()
  set(ARKODE_LIB sundials_arkode_shared)
  set(SUNLS_LIB sundials_sunlinsolcusolversp_shared)
endif()

# Set-up linker flags and link libraries
set(SUNDIALS_LIBS ${ARKODE_LIB} ${SUNLS_LIB} ${EXTRA_LINK_LIBS})

# Add the build and install targets for each ARKode example
foreach(example_tuple ${ARKODE_examples_cusp})

  # parse the example tuple
  list(GET example_tuple 0 example)
  list(GET example_tuple 1 number_of_nodes)
  list(GET example_tuple 2 number_of_tasks)
  list(GET example_tuple 3 example_type)

  # add example source files
  add_executable(${example} ${example}.cu)

  set_target_properties(${example} PROPERTIES FOLDER "Examples")

  # add example to regression tests
  sundials_add_test(${example} ${example}
    ANSWER_DIR ${CMAKE_CURRENT_SOURCE_DIR}
    ANSWER_FILE ${example}.out
    EXAMPLE_TYPE ${example_type})

  # libraries to link against
  target_link_libraries(${example} PRIVATE ${SUNDIALS_LIBS})

  # install example source and out files
  if(EXAMPLES_INSTALL)
    install(FILES ${example}.cu ${example}.out
      DESTINATION ${EXAMPLES_INSTALL_PATH}/arkode/CUDA_serial)
  endif()

endforeach()


# create Makfile and CMakeLists.txt for examples
if(EXAMPLES_INSTALL)

  # Install the README file
  install(FILES README DESTINATION ${EXAMPLES_INSTALL_PATH}/arkode/CUDA_serial)

  # Prepare substitution variables for Makefile and/or CMakeLists templates
  set(SOLVER "ARKODE")
  set(SOLVER_LIB "sundials_arkode")
  set(NVECTOR_LIB "sundials_nveccuda")

  examples2string(ARKODE_examples_cusp EXAMPLES_CUSP)

  # Regardless of the platform we're on, we will generate and install
  # CMakeLists.txt file for building the examples. This file  can then
  # be used as a template for the user's own programs.

  # generate CMakelists.txt in the binary directory
  configure_file(
    ${PROJECT_SOURCE_DIR}/examples/templates/cmakelists_serial_CUDA_ex.in
    ${PROJECT_BINARY_DIR}/examples/arkode/CUDA_serial/CMakeLists.txt
    @ONLY
    )

  # install CMakelists.txt
  install(
    FILES ${PROJECT_BINARY_DIR}/examples/arkode/CUDA_serial/CMakeLists.txt
    DESTINATION ${EXAMPLES_INSTALL_PATH}/arkode/CUDA_serial
    )

  # On UNIX-type platforms, we also  generate and install a makefile for
  # building the examples. This makefile can then be used as a template
  # for the user's own programs.

  if(UNIX)
    # generate Makefile and place it in the binary dir
    configure_file(
      ${PROJECT_SOURCE_DIR}/examples/templates/makefile_serial_CUDA_ex.in
      ${PROJECT_BINARY_DIR}/examples/arkode/CUDA_serial/Makefile_ex
      @ONLY
      )
    # install the configured Makefile_ex as Makefile
    install(
      FILES ${PROJECT_BINARY_DIR}/examples/arkode/CUDA_serial/Makefile_ex
      DESTINATION ${EXAMPLES_INSTALL_PATH}/arkode/CUDA_serial
      RENAME Makefile
      )
  endif(UNIX)

  # add test_install target
  sundials_add_test_install(arkode CUDA_serial)

endif(EXAMPLES_INSTALL)