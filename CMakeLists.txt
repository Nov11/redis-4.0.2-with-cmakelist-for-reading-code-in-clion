# vim: set ai expandtab:

cmake_minimum_required(VERSION 3.2 FATAL_ERROR)

# The following must be set BEFORE doing project() or eanble_language().
# ::-------------------------------------------------------------------------::
message("j")
if (NOT CMAKE_BUILD_TYPE)
    message(STATUS "No build type defined; defaulting to 'Debug'")
    set(CMAKE_BUILD_TYPE "Debug" CACHE STRING
            "The type of build. Possible values are: Debug, Release, RelWithDebInfo and MinSizeRel.")
endif()
# ::-------------------------------------------------------------------------::

project(redis VERSION 3.2)

message(STATUS "Host is: ${CMAKE_HOST_SYSTEM}.  Build target is: ${CMAKE_SYSTEM}")
get_filename_component(REDIS_ROOT "${CMAKE_CURRENT_SOURCE_DIR}" ABSOLUTE)
message(STATUS "Project root directory is: ${REDIS_ROOT}")

# Just for debugging when handling a new platform.
if (false)
    message("C++ compiler supports these language features:")
    foreach(i ${CMAKE_CXX_COMPILE_FEATURES})
        message("  ${i}")
    endforeach()
endif()

add_executable(redis
        src/adlist.c
        src/adlist.h
        src/ae.c
        src/ae.h
        src/anet.c
        src/anet.h
        src/aof.c
        src/asciilogo.h
        src/bio.c
        src/bio.h
        src/bitops.c
        src/blocked.c
        src/cluster.c
        src/cluster.h
        src/config.c
        src/config.h
        src/crc16.c
        src/crc64.c
        src/crc64.h
        src/db.c
        src/debug.c
        src/dict.c
        src/dict.h
        src/endianconv.c
        src/endianconv.h
        src/fmacros.h
        src/geo.c
        src/geo.h
        src/help.h
        src/hyperloglog.c
        src/intset.c
        src/intset.h
        src/latency.c
        src/latency.h
        src/lzf.h
        src/lzfP.h
        src/lzf_c.c
        src/lzf_d.c
        src/memtest.c
        src/multi.c
        src/networking.c
        src/notify.c
        src/object.c
        src/pqsort.c
        src/pqsort.h
        src/pubsub.c
        src/quicklist.c
        src/quicklist.h
        src/rand.c
        src/rand.h
        src/rdb.c
        src/rdb.h
        src/redis-check-rdb.c
        src/redisassert.h
        src/release.c
        src/release.h
        src/replication.c
        src/rio.c
        src/rio.h
        src/scripting.c
        src/sds.c
        src/sds.h
        src/sdsalloc.h
        src/sentinel.c
        src/server.c
        src/server.h
        src/setproctitle.c
        src/sha1.c
        src/sha1.h
        src/slowlog.c
        src/slowlog.h
        src/solarisfixes.h
        src/sort.c
        src/sparkline.c
        src/sparkline.h
        src/syncio.c
        src/t_hash.c
        src/t_list.c
        src/t_set.c
        src/t_string.c
        src/t_zset.c
        src/testhelp.h
        src/util.c
        src/util.h
        src/version.h
        src/ziplist.c
        src/ziplist.h
        src/zipmap.c
        src/zipmap.h
        src/zmalloc.c
        src/zmalloc.h
        )

set_property(TARGET redis PROPERTY C_STANDARD 99)
set_property(TARGET redis PROPERTY CXX_STANDARD 11)
set_property(TARGET redis PROPERTY CXX_STANDARD_REQUIRED ON)


if (${CMAKE_SYSTEM_NAME} MATCHES "Linux")
    #    FINAL_CFLAGS+= -DUSE_JEMALLOC -I../deps/jemalloc/include
    #DEPENDENCY_TARGETS+= jemalloc
    #FINAL_CFLAGS+= -DUSE_JEMALLOC -I../deps/jemalloc/include
    #FINAL_LIBS+= ../deps/jemalloc/lib/libjemalloc.a -ldl
endif()

# XXX: Is this required for OSX?
#set(THREADS_PREFER_PTHREAD_FLAG ON)
#find_package(Threads REQUIRED)

target_include_directories(redis
        PRIVATE ${REDIS_ROOT}/deps/hiredis
        PRIVATE ${REDIS_ROOT}/deps/linenoise
        PRIVATE ${REDIS_ROOT}/deps/lua/src
        PRIVATE ${REDIS_ROOT}/deps/geohash-int
        )

# now create an imported static target
add_library(luaa STATIC IMPORTED)
# Import target "scintilla" for configuration ""
set_property(TARGET luaa APPEND PROPERTY IMPORTED_CONFIGURATIONS NOCONFIG)
set_target_properties(luaa PROPERTIES IMPORTED_LOCATION_NOCONFIG "${REDIS_ROOT}/deps/lua/src/liblua.a")

target_link_libraries(redis
        #PRIVATE Threads::Threads
        PRIVATE m
        PRIVATE luaa
        PRIVATE ${REDIS_ROOT}/deps/lua/src/liblua.a
        PRIVATE ${REDIS_ROOT}/deps/linenoise/linenoise.o
        PRIVATE ${REDIS_ROOT}/deps/hiredis/libhiredis.a
        #        PRIVATE ${REDIS_ROOT}/deps/geohash-int/geohash.o
        #        PRIVATE ${REDIS_ROOT}/deps/geohash-int/geohash_helper.o
        )

# PRIVATE adds the property just to this target.
# INTERFACE adds the property to targets that link to this target.
# PUBLIC acts as both PRIVATE and INTERFACE because it adds the property to both this target and to targets that link to this target.
target_compile_definitions(redis
        PRIVATE REDIS_STATIC=
        )

target_compile_options(redis
        PRIVATE -Wall -W -pedantic
        )

#add_custom_target(
#   extern_lib
#   COMMAND make
#   WORKING_DIRECTORY "${REDIS_ROOT}/deps/lua
#)
#add_executable(myexecutable myexcutable.c)
#target_link_libraries(myexecutable full_path_to_generated_library)
#add_dependencies(myexecutable extern_lib)

message(${PROJECT_SOURCE_DIR})
message(${PROJECT_BINARY_DIR})


add_custom_target(redis2 COMMAND make -C ${PROJECT_SOURCE_DIR}
        CLION_EXE_DIR=${PROJECT_BINARY_DIR} && cp ${PROJECT_SOURCE_DIR}/src/redis-server ./
        )