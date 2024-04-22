cmake_minimum_required(VERSION 3.0)
project(libsvm)

# Set compiler flags
set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -Wall -Wconversion -O3 -fPIC")
set(SHVER 3)

# Check operating system
if (APPLE)
    set(SHARED_LIB_FLAG "-dynamiclib -Wl,-install_name,libsvm.so.${SHVER}")
else ()
    set(SHARED_LIB_FLAG "-shared -Wl,-soname,libsvm.so.${SHVER}")
endif ()

# Uncomment the following lines to enable parallelization with OpenMP
# find_package(OpenMP)
# if(OpenMP_CXX_FOUND)
#     set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} ${OpenMP_CXX_FLAGS}")
#     set(SHARED_LIB_FLAG "${SHARED_LIB_FLAG} ${OpenMP_CXX_FLAGS}")
# endif()

# Include directories
include_directories(${CMAKE_CURRENT_SOURCE_DIR})

# Define executables and their sources
add_executable(svm-predict svm-predict.c svm.o)
add_executable(svm-train svm-train.c svm.o)
add_executable(svm-scale svm-scale.c)

# Link libraries
target_link_libraries(svm-predict m)
target_link_libraries(svm-train m)

# Create shared library
add_library(libsvm SHARED svm.cpp svm.h)
set_target_properties(libsvm PROPERTIES OUTPUT_NAME "svm")
set_target_properties(libsvm PROPERTIES VERSION ${SHVER})
set_target_properties(libsvm PROPERTIES SOVERSION ${SHVER})

# Clean target
add_custom_target(clean-all
        COMMAND ${CMAKE_BUILD_TOOL} clean
        COMMAND rm -f *~ libsvm.so.${SHVER}
        WORKING_DIRECTORY ${CMAKE_CURRENT_SOURCE_DIR}
)

# Install targets
install(TARGETS svm-predict svm-train svm-scale libsvm
        RUNTIME DESTINATION bin
        LIBRARY DESTINATION lib
        ARCHIVE DESTINATION lib
)
