function(op_library TARGET)
    # op_library is a function to create op library. The interface is same as
    # cc_library. But it handle split GPU/CPU code and link some common library
    # for ops.
    set(cc_srcs)
    set(cu_srcs)
    set(op_common_deps operator op_registry)
    set(options "")
    set(oneValueArgs "")
    set(multiValueArgs SRCS DEPS)
    cmake_parse_arguments(op_library "${options}" "${oneValueArgs}"
            "${multiValueArgs}" ${ARGN})

    foreach(src ${op_library_SRCS})
        if (${src} MATCHES ".*\\.cu$")
            list(APPEND cu_srcs ${src})
        elseif(${src} MATCHES ".*\\.cc$")
            list(APPEND cc_srcs ${src})
        else()
            message(FATAL_ERROR "${TARGET} Source file ${src} should only be .cc or .cu")
        endif()
    endforeach()

    list(LENGTH cc_srcs cc_srcs_len)
    if (${cc_srcs_len} EQUAL 0)
        message(FATAL_ERROR "The op library ${TARGET} should contains at least one .cc file")
    endif()

    list(LENGTH cu_srcs cu_srcs_len)
    if (${cu_srcs_len} EQUAL 0)
        message(WARNING "The op library ${TARGET} not support GPU!")
    endif()

    if (WITH_GPU)
        nv_library(${TARGET} SRCS ${cc_srcs} ${cu_srcs} DEPS ${op_library_DEPS}
                ${op_common_deps})
    else()
        cc_library(${TARGET} SRCS ${cc_srcs} DEPS ${op_library_DEPS}
                ${op_common_deps})
    endif()
endfunction()

op_library(add_op SRCS add_op.cc add_op.cu)
cc_test(add_op_test SRCS add_op_test.cc DEPS add_op)
