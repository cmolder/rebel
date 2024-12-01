cmake_minimum_required(VERSION 3.15 FATAL_ERROR)
project(cfr LANGUAGES CXX)
add_definitions(-D_GLIBCXX_USE_CXX11_ABI=0)

set(CMAKE_CXX_STANDARD 17)

set(CMAKE_CXX_FLAGS
  "${CMAKE_CXX_FLAGS} -I .. -Wall -mcmodel=medium -march=native -Wextra -Wno-register -fPIC -Wfatal-errors -O3")
set(CMAKE_CXX_FLAGS
  "${CMAKE_CXX_FLAGS} -g -fno-omit-frame-pointer")

# Build GTest.
# Prevent overriding the parent project's compiler/linker
# settings on Windows
# set(gtest_force_shared_crt ON CACHE BOOL "" FORCE)

# Add googletest directly to our build. This defines
# the gtest and gtest_main targets.
# add_subdirectory(${CMAKE_SOURCE_DIR}/../../third_party/googletest third_party/googletest EXCLUDE_FROM_ALL)

# find packages & third_party
find_package(PythonInterp 3.7 REQUIRED)
find_package(PythonLibs 3.7 REQUIRED)
find_package(pybind11 REQUIRED)
# find_package(GTest REQUIRED)
# find_package(CUDAToolkit 12.4 REQUIRED)

include_directories(${PYTHON_INCLUDE_DIRS})

# Set conda environment paths
if (DEFINED ENV{CONDA_PREFIX})
  set(CONDA_PREFIX $ENV{CONDA_PREFIX})
  list(APPEND CMAKE_PREFIX_PATH "${CONDA_PREFIX}")
  list(APPEND CMAKE_INCLUDE_PATH "${CONDA_PREFIX}/include")
  list(APPEND CMAKE_LIBRARY_PATH "${CONDA_PREFIX}/lib")

  # set(CUDA_HOME "${CONDA_PREFIX}")
  # set(CUDA_TOOLKIT_ROOT_DIR "${CONDA_PREFIX}")
  # set(CUDA_INCLUDE_DIRS "${CONDA_PREFIX}/include")
  # set(CUDA_NVCC_EXECUTABLE "${CONDA_PREFIX}/bin/nvcc")

  set(CUDAToolkit_ROOT "${CONDA_PREFIX}")
  set(CUDAToolkit_INCLUDE_DIR "${CONDA_PREFIX}/include")
  set(CUDAToolkit_LIBRARY_DIR "${CONDA_PREFIX}/lib")

  list(PREPEND PATH "${CONDA_PREFIX}/bin")
endif()

#### TORCH LIB  #####
if (NOT TARGET torch_library)
  execute_process(
    COMMAND python -c "import torch; import os; print(os.path.dirname(torch.__file__), end='')"
    OUTPUT_VARIABLE TorchPath
  )
  list(APPEND CMAKE_PREFIX_PATH ${TorchPath})
  find_package(Torch REQUIRED)
endif()

add_library(liars_dice_lib liars_dice.cc subgame_solving.cc real_net.cc recursive_solving.cc stats.cc)
target_link_libraries(liars_dice_lib torch torch_python)
set_target_properties(liars_dice_lib PROPERTIES CXX_STANDARD 17)

add_executable(recursive_eval recursive_eval.cc)
target_link_libraries(recursive_eval liars_dice_lib)

# add_subdirectory(
#   ${CMAKE_CURRENT_SOURCE_DIR}/../../third_party/pybind11 third_party/pybind11
# )

add_library(_rela
  rela/types.cc
)
target_include_directories(_rela PUBLIC ${CMAKE_CURRENT_SOURCE_DIR})
target_include_directories(_rela PUBLIC ${PYTHON_INCLUDE_DIRS})
target_include_directories(_rela PUBLIC $ENV{CONDA_PREFIX}/include)
target_link_libraries(_rela torch)

# python lib
pybind11_add_module(rela rela/pybind.cc)
target_link_libraries(rela PUBLIC _rela liars_dice_lib ${PYTHON_LIBRARIES})

add_executable(gen_benchmark gen_benchmark.cc)
target_link_libraries(gen_benchmark liars_dice_lib _rela ${PYTHON_LIBRARIES})

#################
# Tests
# include(GoogleTest)
# enable_testing()

# add_executable(liar_game_test liars_dice_test.cc)
# target_link_libraries(liar_game_test liars_dice_lib gtest_main)
# add_test(NAME liar_game COMMAND liar_game_test)

# add_executable(liar_tree_test tree_test.cc)
# target_link_libraries(liar_tree_test liars_dice_lib gtest_main)
# add_test(NAME liar_tree COMMAND liar_tree_test)

# add_executable(liar_recursive_solving_test recursive_solving_test.cc)
# target_link_libraries(liar_recursive_solving_test liars_dice_lib gtest_main)
# add_test(NAME liar_recursive_solving COMMAND liar_recursive_solving_test)

# add_executable(liar_subgame_solving_test subgame_solving_test.cc)
# target_link_libraries(liar_subgame_solving_test liars_dice_lib gtest_main)
# add_test(NAME liar_subgame_solving COMMAND liar_subgame_solving_test)
