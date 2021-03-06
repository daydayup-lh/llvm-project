##===----------------------------------------------------------------------===##
#
# Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
# See https://llvm.org/LICENSE.txt for license information.
# SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
#
##===----------------------------------------------------------------------===##
#
# Build offloading library libomptarget.so.
#
##===----------------------------------------------------------------------===##

libomptarget_say("Building offloading runtime library libomptarget.")

set(LIBOMPTARGET_SRC_FILES
  ${CMAKE_CURRENT_SOURCE_DIR}/api.cpp
  ${CMAKE_CURRENT_SOURCE_DIR}/device.cpp
  ${CMAKE_CURRENT_SOURCE_DIR}/interface.cpp
  ${CMAKE_CURRENT_SOURCE_DIR}/MemoryManager.cpp
  ${CMAKE_CURRENT_SOURCE_DIR}/rtl.cpp
  ${CMAKE_CURRENT_SOURCE_DIR}/omptarget.cpp
)

# Build libomptarget library with libdl dependency. Add LLVMSupport
# dependency if building in-tree with profiling enabled.
if(OPENMP_STANDALONE_BUILD OR (NOT OPENMP_ENABLE_LIBOMPTARGET_PROFILING))
  add_library(omptarget SHARED ${LIBOMPTARGET_SRC_FILES})
  target_link_libraries(omptarget
    ${CMAKE_DL_LIBS}
    "-Wl,--version-script=${CMAKE_CURRENT_SOURCE_DIR}/exports")
else()
  set(LLVM_LINK_COMPONENTS
    Support
    )
  add_llvm_library(omptarget SHARED ${LIBOMPTARGET_SRC_FILES}
      LINK_LIBS ${CMAKE_DL_LIBS}
      "-Wl,--version-script=${CMAKE_CURRENT_SOURCE_DIR}/exports"
      )
  target_compile_definitions(omptarget PUBLIC OMPTARGET_PROFILE_ENABLED)
endif()

# Install libomptarget under the lib destination folder.
install(TARGETS omptarget LIBRARY COMPONENT omptarget
  DESTINATION "${OPENMP_INSTALL_LIBDIR}")
