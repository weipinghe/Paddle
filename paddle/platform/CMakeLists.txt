cc_library(cpu_info SRCS cpu_info.cc DEPS gflags glog)
cc_test(cpu_info_test SRCS cpu_info_test.cc DEPS cpu_info)

nv_library(gpu_info SRCS gpu_info.cc DEPS gflags)

cc_library(place SRCS place.cc)
cc_test(place_test SRCS place_test.cc DEPS place glog gflags)

add_subdirectory(dynload)

IF(WITH_GPU)
    set(GPU_CTX_DEPS dynload_cuda dynamic_loader)
ELSE()
    set(GPU_CTX_DEPS)
ENDIF()

cc_library(device_context SRCS device_context.cc DEPS place eigen3 ${GPU_CTX_DEPS})
nv_test(device_context_test SRCS device_context_test.cc DEPS device_context gpu_info)
