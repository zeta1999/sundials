# ---------------------------------------------------------------
# Programmer(s): Daniel R. Reynolds @ SMU
#                David J. Gardner @ LLNL
# ---------------------------------------------------------------
# SUNDIALS Copyright Start
# Copyright (c) 2002-2020, Lawrence Livermore National Security
# and Southern Methodist University.
# All rights reserved.
#
# See the top-level LICENSE and NOTICE files for details.
#
# SPDX-License-Identifier: BSD-3-Clause
# SUNDIALS Copyright End
# ---------------------------------------------------------------
# CMakeLists.txt file for ARKODE serial examples
# ---------------------------------------------------------------

# Example lists are tuples "name\;type" where the type is
# 'develop' for examples excluded from 'make test' in releases

# Examples using SUNDIALS linear solvers
set(ARKODE_examples
  "ark_analytic\;"
  "ark_analytic_nonlin\;develop"
  "ark_brusselator\;develop"
  "ark_brusselator_fp\;develop"
  "ark_brusselator1D\;develop"
  "ark_heat1D\;develop"
  "ark_heat1D_adapt\;develop"
  "ark_KrylovDemo_prec\;develop"
  "ark_robertson\;develop"
  "ark_robertson_constraints\;develop"
  "ark_robertson_root\;develop"
  "ark_brusselator_mri\;develop"
  "ark_onewaycouple_mri\;develop"
  "ark_twowaycouple_mri\;develop"
  "ark_reaction_diffusion_mri\;develop"
  "ark_brusselator_1D_mri\;develop"
  )

# Examples using LAPACK linear solvers
set(ARKODE_examples_BL
  )

# Examples using KLU linear solver
set(ARKODE_examples_KLU
  "ark_brusselator1D_klu\;develop"
  )

# Examples using SuperLU_MT linear solver
set(ARKODE_examples_SUPERLUMT
  "ark_brusselator1D_FEM_slu\;develop"
  )

# Auxiliary files to install
set(ARKODE_extras
  plot_brusselator1D.py
  plot_brusselator1D_FEM.py
  plot_heat1D.py
  plot_heat1D_adapt.py
  plot_sol.py
  plot_sol_log.py
  )

# Specify libraries to link against (through the target that was used to
# generate them) based on the value of the variable LINK_LIBRARY_TYPE
if(LINK_LIBRARY_TYPE MATCHES "static")
  set(ARKODE_LIB sundials_arkode_static)
  set(NVECS_LIB sundials_nvecserial_static)
else()
  set(ARKODE_LIB sundials_arkode_shared)
  set(NVECS_LIB sundials_nvecserial_shared)
endif()

# Set-up linker flags and link libraries
set(SUNDIALS_LIBS ${ARKODE_LIB} ${NVECS_LIB} ${EXTRA_LINK_LIBS})


# Add the build and install targets for each ARKODE example
foreach(example_tuple ${ARKODE_examples})

  # parse the example tuple
  list(GET example_tuple 0 example)
  list(GET example_tuple 1 example_type)

  # example source files
  add_executable(${example} ${example}.c)

  set_target_properties(${example} PROPERTIES FOLDER "Examples")

  # add example to regression tests
  sundials_add_test(${example} ${example}
    ANSWER_DIR ${CMAKE_CURRENT_SOURCE_DIR}
    ANSWER_FILE ${example}.out
    EXAMPLE_TYPE ${example_type})

  # libraries to link against
  target_link_libraries(${example} ${SUNDIALS_LIBS})

  # install example source and out files
  if(EXAMPLES_INSTALL)
    install(FILES ${example}.c ${example}.out
      DESTINATION ${EXAMPLES_INSTALL_PATH}/arkode/C_serial)
  endif()

endforeach(example_tuple ${ARKODE_examples})


# Add the build and install targets for each LAPACK example (if needed)
if(LAPACK_FOUND)

  # Sundials LAPACK linear solver modules
  if(LINK_LIBRARY_TYPE MATCHES "static")
    set(SUNLINSOLLAPACK_LIBS
      sundials_sunlinsollapackband_static
      sundials_sunlinsollapackdense_static)
  else()
    set(SUNLINSOLLAPACK_LIBS
      sundials_sunlinsollapackband_shared
      sundials_sunlinsollapackdense_shared)
  endif()

  # LAPACK libraries
  list(APPEND SUNLINSOLLAPACK_LIBS ${LAPACK_LIBRARIES})

  # BLAS libraries
  if(BLAS_FOUND)
    list(APPEND SUNLINSOLLAPACK_LIBS ${BLAS_LIBRARIES})
  endif(BLAS_FOUND)

  foreach(example_tuple ${ARKODE_examples_BL})

    # parse the example tuple
    list(GET example_tuple 0 example)
    list(GET example_tuple 1 example_type)

    # example source files
    add_executable(${example} ${example}.c)

    set_target_properties(${example} PROPERTIES FOLDER "Examples")

    # add example to regression tests
    sundials_add_test(${example} ${example}
      ANSWER_DIR ${CMAKE_CURRENT_SOURCE_DIR}
      ANSWER_FILE ${example}.out
      EXAMPLE_TYPE ${example_type})

    # libraries to link against
    target_link_libraries(${example} ${SUNDIALS_LIBS} ${SUNLINSOLLAPACK_LIBS})

    # install example source and out files
    if(EXAMPLES_INSTALL)
      install(FILES ${example}.c ${example}.out
        DESTINATION ${EXAMPLES_INSTALL_PATH}/arkode/C_serial)
    endif()

  endforeach(example_tuple ${ARKODE_examples_BL})

endif(LAPACK_FOUND)


# Add the build and install targets for each KLU example (if needed)
if(KLU_FOUND)

  # Sundials KLU linear solver module
  if(LINK_LIBRARY_TYPE MATCHES "static")
    set(SUNLINSOLKLU_LIBS sundials_sunlinsolklu_static)
  else()
    set(SUNLINSOLKLU_LIBS sundials_sunlinsolklu_shared)
  endif()

  # KLU libraries
  list(APPEND SUNLINSOLKLU_LIBS ${KLU_LIBRARIES})

  foreach(example_tuple ${ARKODE_examples_KLU})

    # parse the example tuple
    list(GET example_tuple 0 example)
    list(GET example_tuple 1 example_type)

    # example source files
    add_executable(${example} ${example}.c)

    set_target_properties(${example} PROPERTIES FOLDER "Examples")

    # add example to regression tests
    sundials_add_test(${example} ${example}
      ANSWER_DIR ${CMAKE_CURRENT_SOURCE_DIR}
      ANSWER_FILE ${example}.out
      EXAMPLE_TYPE ${example_type})

    # libraries to link against
    target_link_libraries(${example} ${SUNDIALS_LIBS} ${SUNLINSOLKLU_LIBS})

    # install example source and out files
    if(EXAMPLES_INSTALL)
      install(FILES ${example}.c ${example}.out
        DESTINATION ${EXAMPLES_INSTALL_PATH}/arkode/C_serial)
    endif()

  endforeach(example_tuple ${ARKODE_examples_KLU})

endif(KLU_FOUND)


# Add the build and install targets for each SuperLU_MT example (if needed)
if(SUPERLUMT_FOUND)

  # Sundials SuperLU_MT linear solver module
  if(LINK_LIBRARY_TYPE MATCHES "static")
    set(SUNLINSOLSLUMT_LIBS sundials_sunlinsolsuperlumt_static)
  else()
    set(SUNLINSOLSLUMT_LIBS sundials_sunlinsolsuperlumt_shared)
  endif()

  # SuperLU_MT libraries
  list(APPEND SUNLINSOLSLUMT_LIBS ${SUPERLUMT_LIBRARIES})

  # BLAS libraries
  if(BLAS_FOUND)
    list(APPEND SUNLINSOLSLUMT_LIBS ${BLAS_LIBRARIES})
  endif(BLAS_FOUND)

  foreach(example_tuple ${ARKODE_examples_SUPERLUMT})

    # parse the example tuple
    list(GET example_tuple 0 example)
    list(GET example_tuple 1 example_type)

    # example source files
    add_executable(${example} ${example}.c)

    set_target_properties(${example} PROPERTIES FOLDER "Examples")

    # add example to regression tests
    sundials_add_test(${example} ${example}
      ANSWER_DIR ${CMAKE_CURRENT_SOURCE_DIR}
      ANSWER_FILE ${example}.out
      EXAMPLE_TYPE ${example_type})

    # libraries to link against
    target_link_libraries(${example} ${SUNDIALS_LIBS} ${SUNLINSOLSLUMT_LIBS})

    # install example source and out files
    if(EXAMPLES_INSTALL)
      install(FILES ${example}.c ${example}.out
        DESTINATION ${EXAMPLES_INSTALL_PATH}/arkode/C_serial)
    endif()

  endforeach(example_tuple ${ARKODE_examples_SUPERLUMT})

endif(SUPERLUMT_FOUND)


# create Makfile and CMakeLists.txt for examples
if(EXAMPLES_INSTALL)

  # Install the README file
  install(FILES README DESTINATION ${EXAMPLES_INSTALL_PATH}/arkode/C_serial)

  # Install the extra files
  foreach(extrafile ${ARKODE_extras})
    install(FILES ${extrafile} DESTINATION ${EXAMPLES_INSTALL_PATH}/arkode/C_serial)
  endforeach(extrafile ${ARKODE_extras})

  # Prepare substitution variables for Makefile and/or CMakeLists templates
  set(SOLVER "ARKODE")
  set(SOLVER_LIB "sundials_arkode")

  examples2string(ARKODE_examples EXAMPLES)

  if(LAPACK_FOUND)
    examples2string(ARKODE_examples_BL EXAMPLES_BL)
  else()
    set(EXAMPLES_BL "")
  endif()

  if(KLU_FOUND)
    examples2string(ARKODE_examples_KLU EXAMPLES_KLU)
  else()
    set(EXAMPLES_KLU "")
  endif()

  if(SUPERLUMT_FOUND)
    examples2string(ARKODE_examples_SUPERLUMT EXAMPLES_SLUMT)
    if (SUNDIALS_SUPERLUMT_THREAD_TYPE STREQUAL "PTHREAD")
      set(THREAD_LIBRARY_SLUMT ${CMAKE_THREAD_LIBS_INIT})
    else()
      set(THREAD_LIBRARY_SLUMT "")
    endif()
  else()
    set(EXAMPLES_SLUMT "")
    set(THREAD_LIBRARY_SLUMT "")
  endif()

  # Regardless of the platform we're on, we will generate and install
  # CMakeLists.txt file for building the examples. This file  can then
  # be used as a template for the user's own programs.

  # generate CMakelists.txt in the binary directory
  configure_file(
    ${PROJECT_SOURCE_DIR}/examples/templates/cmakelists_serial_C_ex.in
    ${PROJECT_BINARY_DIR}/examples/arkode/C_serial/CMakeLists.txt
    @ONLY
    )

  # install CMakelists.txt
  install(
    FILES ${PROJECT_BINARY_DIR}/examples/arkode/C_serial/CMakeLists.txt
    DESTINATION ${EXAMPLES_INSTALL_PATH}/arkode/C_serial
    )

  # On UNIX-type platforms, we also  generate and install a makefile for
  # building the examples. This makefile can then be used as a template
  # for the user's own programs.

  if(UNIX)
    # generate Makefile and place it in the binary dir
    configure_file(
      ${PROJECT_SOURCE_DIR}/examples/templates/makefile_serial_C_ex.in
      ${PROJECT_BINARY_DIR}/examples/arkode/C_serial/Makefile_ex
      @ONLY
      )
    # install the configured Makefile_ex as Makefile
    install(
      FILES ${PROJECT_BINARY_DIR}/examples/arkode/C_serial/Makefile_ex
      DESTINATION ${EXAMPLES_INSTALL_PATH}/arkode/C_serial
      RENAME Makefile
      )
  endif(UNIX)

  # add test_install target
  sundials_add_test_install(arkode C_serial EXECUTABLE ark_analytic)

endif(EXAMPLES_INSTALL)
