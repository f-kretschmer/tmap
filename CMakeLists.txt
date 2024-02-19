# message( FATAL_ERROR "some error")
cmake_minimum_required(VERSION 3.12)
set(CMAKE_CXX_STANDARD 17)
set(CMAKE_CXX_STANDARD_REQUIRED ON)
set(CMAKE_CXX_EXTENSIONS OFF)

if(MSVC)
    set(CMAKE_GENERATOR_TOOLSET "ClangCL")
endif(MSVC)

if(DEFINED ENV{ARCHFLAGS})
  # Parse the cibuildwheel ARCHFLAGS env variable as something we can understand on
  #  OSX, initially.
  string(REPLACE "-arch " "" NEW_ARCHFLAGS $ENV{ARCHFLAGS})
  string(REPLACE " " ";" NEW_ARCHFLAGS "${NEW_ARCHFLAGS}")

  # The following, CMAKE_OSX_ARCHITECTURES, is ignored on non-osx platforms
  set(CMAKE_OSX_ARCHITECTURES "${NEW_ARCHFLAGS}")
endif(DEFINED ENV{ARCHFLAGS})

project(_tmap)

# Put LIBOGDF on the include- and lib-path
include_directories($ENV{LIBOGDF_INSTALL_PATH}/include)
link_directories($ENV{LIBOGDF_INSTALL_PATH}/lib)

if(MSVC)
    set(CMAKE_CXX_FLAGS_DEBUG "${CMAKE_CXX_FLAGS_DEBUG} /O2")
    set(CMAKE_CXX_FLAGS_RELEASE "${CMAKE_CXX_FLAGS_RELEASE} /O2")
    # set(CMAKE_CXX_FLAGS_DEBUG "${CMAKE_CXX_FLAGS_DEBUG} /permissive /std:c++17")
    # set(CMAKE_CXX_FLAGS_RELEASE "${CMAKE_CXX_FLAGS_RELEASE} /permissive /std:c++17 /wd4068 /wd4305 /wd4267 /wd4244")
else()
  # Set O2 optimisations, the more stable option
  set(CMAKE_CXX_FLAGS_DEBUG "${CMAKE_CXX_FLAGS_DEBUG} -O2")
  set(CMAKE_CXX_FLAGS_RELEASE "${CMAKE_CXX_FLAGS_RELEASE} -O2")

endif(MSVC)

find_package(OpenMP REQUIRED)

set(THREADS_PREFER_PTHREAD_FLAG ON)

find_package(Threads REQUIRED)

SET(SOURCE_DIR "src/_tmap")
# Tell cmake that headers are in alse in source_dir
include_directories(${SOURCE_DIR})
SET(SOURCES "${SOURCE_DIR}/layout.cc" "${SOURCE_DIR}/lshforest.cc" "${SOURCE_DIR}/minhash.cc" "${SOURCE_DIR}/analyse.cc")

if(CMAKE_HOST_WIN32)
  add_library(tmap STATIC ${SOURCES})
else()
  add_library(tmap SHARED ${SOURCES})
endif(CMAKE_HOST_WIN32)

if(MSVC)
  target_link_libraries(tmap PRIVATE OGDF.lib COIN.lib)
else()
  target_link_libraries(tmap PRIVATE OGDF COIN)
endif(MSVC)

target_link_libraries(tmap PRIVATE Threads::Threads)
target_link_libraries(tmap PRIVATE OpenMP::OpenMP_CXX)
