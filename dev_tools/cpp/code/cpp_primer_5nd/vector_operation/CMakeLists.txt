cmake_minimum_required( VERSION 2.8 )
project( vector_operation )

set( CMAKE_BUILD_TYPE "Release" )
set( CMAKE_CXX_FLAGS "-std=c++11 -O3" )

add_executable( vector_operation vector_operation.cpp  )
