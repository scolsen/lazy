# Copyright 2020 Google LLC
#
# Licensed under the Apache License, Version 2.0 (the "License");
# you may not use this file except in compliance with the License.
# You may obtain a copy of the License at
#
#  https://www.apache.org/licenses/LICENSE-2.0
#
#  Unless required by applicable law or agreed to in writing, software
#  distributed under the License is distributed on an "AS IS" BASIS,
#  WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
#  See the License for the specific language governing permissions and
#  limitations under the License.

cmake_minimum_required(VERSION 3.16)
project(lazy_carp)
set(CMAKE_CXX_STANDARD 14)
include_directories($ENV{CARP_DIR}/core)

# Generate c code from carp source files.
add_custom_target(gen_c
        COMMAND carp -b genc.carp
        WORKING_DIRECTORY ${PROJECT_SOURCE_DIR}
        COMMAND mv ${PROJECT_SOURCE_DIR}/c/main.c ${PROJECT_SOURCE_DIR}/c/lazy.c)

# Generate docs from carp source files.
add_custom_target(gen_docs
        COMMAND carp -x gendocs.carp
        WORKING_DIRECTORY ${PROJECT_SOURCE_DIR})

# Run all tests under test/
add_custom_target(test
        COMMAND ./test-all.sh
        WORKING_DIRECTORY ${PROJECT_SOURCE_DIR})

add_library(lazy_carp ${PROJECT_SOURCE_DIR}/c/lazy.c)
add_dependencies(lazy_carp gen_c gen_docs)
