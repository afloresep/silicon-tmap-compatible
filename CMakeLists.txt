# message( FATAL_ERROR "some error")
cmake_minimum_required(VERSION 3.10)
set(CMAKE_CXX_STANDARD 17)
set(CMAKE_CXX_STANDARD_REQUIRED ON)
set(CMAKE_CXX_EXTENSIONS OFF)

# Set minimum macOS deployment target for arm64 compatibility
if(APPLE)
  set(CMAKE_OSX_DEPLOYMENT_TARGET "14.0" CACHE STRING "Minimum macOS version" FORCE)
endif()

if(MSVC)
    set(CMAKE_GENERATOR_TOOLSET "ClangCL")
endif(MSVC)

if(DEFINED ENV{ARCHFLAGS})
  # Parse the cibuildwheel ARCHFLAGS env variable as something we can understand on
  #  OSX, initially.
  string(REPLACE "-arch " "" NEW_ARCHFLAGS $ENV{ARCHFLAGS})
  string(REPLACE " " ";" NEW_ARCHFLAGS "${NEW_ARCHFLAGS}")

  # The following, CMAKE_OSX_ARCHITECTURES, is ignored on non-osx platforms
  set(CMAKE_OSX_ARCHITECTURES "${NEW_ARCHFLAGS}" CACHE STRING "Target architecture" FORCE)
endif(DEFINED ENV{ARCHFLAGS})

project(_tmap)

# Put LIBOGDF on the include- and lib-path
include_directories($ENV{LIBOGDF_INSTALL_PATH}/include)
link_directories($ENV{LIBOGDF_INSTALL_PATH}/lib)
# Also check for arch-specific lib directory (e.g., lib/x86_64-linux-gnu on Debian/Ubuntu)
link_directories($ENV{LIBOGDF_INSTALL_PATH}/lib/${CMAKE_LIBRARY_ARCHITECTURE})

# Find OGDF and COIN libraries explicitly
if(APPLE)
  find_library(OGDF_LIBRARY NAMES OGDF libOGDF PATHS $ENV{LIBOGDF_INSTALL_PATH}/lib NO_DEFAULT_PATH)
  find_library(COIN_LIBRARY NAMES COIN libCOIN PATHS $ENV{LIBOGDF_INSTALL_PATH}/lib NO_DEFAULT_PATH)
  if(NOT OGDF_LIBRARY)
    message(FATAL_ERROR "OGDF library not found in $ENV{LIBOGDF_INSTALL_PATH}/lib")
  endif()
  if(NOT COIN_LIBRARY)
    message(FATAL_ERROR "COIN library not found in $ENV{LIBOGDF_INSTALL_PATH}/lib")
  endif()
  message(STATUS "Found OGDF: ${OGDF_LIBRARY}")
  message(STATUS "Found COIN: ${COIN_LIBRARY}")
endif()

if(MSVC)
    set(CMAKE_CXX_FLAGS_DEBUG "${CMAKE_CXX_FLAGS_DEBUG} /O2")
    set(CMAKE_CXX_FLAGS_RELEASE "${CMAKE_CXX_FLAGS_RELEASE} /O2")
    # set(CMAKE_CXX_FLAGS_DEBUG "${CMAKE_CXX_FLAGS_DEBUG} /permissive /std:c++17")
    # set(CMAKE_CXX_FLAGS_RELEASE "${CMAKE_CXX_FLAGS_RELEASE} /permissive /std:c++17 /wd4068 /wd4305 /wd4267 /wd4244")
    
    if(CMAKE_C_COMPILER_ID MATCHES "Clang")
      set(OpenMP_C_FLAGS "-Wno-unused-command-line-argument")
      set(OpenMP_C_LIB_NAMES "libomp" "libgomp" "libiomp5")
      set(OpenMP_libomp_LIBRARY ${OpenMP_C_LIB_NAMES})
      set(OpenMP_libgomp_LIBRARY ${OpenMP_C_LIB_NAMES})
      set(OpenMP_libiomp5_LIBRARY ${OpenMP_C_LIB_NAMES})
    endif()

    if(CMAKE_CXX_COMPILER_ID MATCHES "Clang")
      set(OpenMP_CXX_FLAGS "-Wno-unused-command-line-argument")
      set(OpenMP_CXX_LIB_NAMES "libomp" "libgomp" "libiomp5")
      set(OpenMP_libomp_LIBRARY ${OpenMP_CXX_LIB_NAMES})
      set(OpenMP_libgomp_LIBRARY ${OpenMP_CXX_LIB_NAMES})
      set(OpenMP_libiomp5_LIBRARY ${OpenMP_CXX_LIB_NAMES})
    endif()
else()
  # Set O2 optimisations, the more stable option
  set(CMAKE_CXX_FLAGS_DEBUG "${CMAKE_CXX_FLAGS_DEBUG} -O2")
  set(CMAKE_CXX_FLAGS_RELEASE "${CMAKE_CXX_FLAGS_RELEASE} -O2")

  if(APPLE)
    # macOS with Apple Clang needs special OpenMP setup
    # libomp is installed via Homebrew
    execute_process(
      COMMAND brew --prefix libomp
      OUTPUT_VARIABLE LIBOMP_PREFIX
      OUTPUT_STRIP_TRAILING_WHITESPACE
      ERROR_QUIET
    )
    if(LIBOMP_PREFIX)
      set(OpenMP_C_FLAGS "-Xpreprocessor -fopenmp")
      set(OpenMP_CXX_FLAGS "-Xpreprocessor -fopenmp")
      set(OpenMP_C_LIB_NAMES "omp")
      set(OpenMP_CXX_LIB_NAMES "omp")
      set(OpenMP_omp_LIBRARY "${LIBOMP_PREFIX}/lib/libomp.dylib")
      include_directories("${LIBOMP_PREFIX}/include")
      link_directories("${LIBOMP_PREFIX}/lib")
    endif()
  elseif(CMAKE_C_COMPILER_ID MATCHES "Clang")
    set(OpenMP_C "${CMAKE_C_COMPILER}")
    set(OpenMP_C_FLAGS "-fopenmp")
    set(OpenMP_C_LIB_NAMES "omp")
  endif()

  if(NOT APPLE AND CMAKE_CXX_COMPILER_ID MATCHES "Clang")
    set(OpenMP_CXX "${CMAKE_CXX_COMPILER}")
    set(OpenMP_CXX_FLAGS "-fopenmp")
    set(OpenMP_CXX_LIB_NAMES "omp")
  elseif(NOT APPLE AND NOT CMAKE_CXX_COMPILER_ID MATCHES "Clang")
    set(CMAKE_CXX_FLAGS_DEBUG "${CMAKE_CXX_FLAGS_DEBUG} -fopenmp")
    set(CMAKE_CXX_FLAGS_RELEASE "${CMAKE_CXX_FLAGS_RELEASE} -fopenmp")
  endif()
endif(MSVC)

find_package(OpenMP REQUIRED)

set(THREADS_PREFER_PTHREAD_FLAG ON)

find_package(Threads REQUIRED)

SET(SOURCE_DIR "src/_tmap")
# Tell cmake that headers are in alse in source_dir
include_directories(${SOURCE_DIR})
SET(SOURCES "${SOURCE_DIR}/layout.cc" "${SOURCE_DIR}/lshforest.cc" "${SOURCE_DIR}/minhash.cc" "${SOURCE_DIR}/analyse.cc")

# Generate python module
add_subdirectory(pybind11)

pybind11_add_module(_tmap ${SOURCES} "${SOURCE_DIR}/bindings.cc")

# Set RPATH for finding shared libraries at runtime
if(APPLE)
  set_target_properties(_tmap PROPERTIES
    BUILD_WITH_INSTALL_RPATH TRUE
    INSTALL_RPATH "@loader_path"
  )
endif()

if(MSVC)
  target_link_libraries(_tmap PRIVATE OGDF.lib COIN.lib)
elseif(APPLE)
  target_link_libraries(_tmap PRIVATE ${OGDF_LIBRARY} ${COIN_LIBRARY})
else()
  target_link_libraries(_tmap PRIVATE OGDF COIN)
endif()

target_link_libraries(_tmap PRIVATE Threads::Threads)
target_link_libraries(_tmap PRIVATE ${OpenMP_CXX_FLAGS})
