# Copyright (c) Meta Platforms, Inc. and affiliates.
# All rights reserved.
#
# This source code is licensed under the BSD-style license found in the
# LICENSE file in the root directory of this source tree.

# Kernel library for Cortex-M operators. Please keep this file formatted by running:
# ~~~
# cmake-format -i CMakeLists.txt
# ~~~
cmake_minimum_required(VERSION 3.19)

set(CMAKE_EXPORT_COMPILE_COMMANDS ON)
if(NOT CMAKE_CXX_STANDARD)
  set(CMAKE_CXX_STANDARD 17)
endif()

# Source root directory for executorch.
if(NOT EXECUTORCH_ROOT)
  set(EXECUTORCH_ROOT ${CMAKE_CURRENT_SOURCE_DIR}/../..)
endif()

include(${EXECUTORCH_ROOT}/tools/cmake/Utils.cmake)
include(${EXECUTORCH_ROOT}/tools/cmake/Codegen.cmake)

if(NOT PYTHON_EXECUTABLE)
  resolve_python_executable()
endif()

# Cortex-M ops kernel sources
set(_cortex_m_kernels__srcs
    ${CMAKE_CURRENT_SOURCE_DIR}/ops/op_quantize_per_tensor.cpp
    ${CMAKE_CURRENT_SOURCE_DIR}/ops/op_dequantize_per_tensor.cpp
)

# Generate C++ bindings to register kernels into Executorch (for runtime).
# Here select all ops in operators.yaml
set(_yaml_file ${CMAKE_CURRENT_LIST_DIR}/ops/operators.yaml)
gen_selected_ops(LIB_NAME "cortex_m_ops_lib" OPS_SCHEMA_YAML "${_yaml_file}")

# Generate bindings for the kernels
generate_bindings_for_kernels(
  LIB_NAME "cortex_m_ops_lib" CUSTOM_OPS_YAML "${_yaml_file}"
)
message("Generated files ${gen_command_sources}")

# Build a library for _cortex_m_kernels_srcs
add_library(cortex_m_kernels ${_cortex_m_kernels__srcs})
target_link_libraries(cortex_m_kernels PRIVATE executorch)
target_compile_options(cortex_m_kernels PUBLIC ${_common_compile_options})

# cortex_m_ops_lib: Register Cortex-M ops kernels into Executorch runtime
gen_operators_lib(
  LIB_NAME "cortex_m_ops_lib" KERNEL_LIBS cortex_m_kernels DEPS executorch
)

install(
  TARGETS cortex_m_kernels cortex_m_ops_lib
  DESTINATION lib
  PUBLIC_HEADER DESTINATION include/executorch/backends/cortex_m/ops/
)
