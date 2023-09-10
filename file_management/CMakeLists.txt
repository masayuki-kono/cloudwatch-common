cmake_minimum_required(VERSION 2.8.3)
project(file_management)
set(FILE_MANAGEMENT_VERSION 1.0.0)

if(CMAKE_COMPILER_IS_GNUCXX OR CMAKE_CXX_COMPILER_ID MATCHES "Clang")
  add_compile_options(-Wall -Wextra -Wpedantic)
endif()

set(SERVICE_COMPONENTS core)

find_package(dataflow_lite REQUIRED)
find_package(aws_common REQUIRED)
find_package(AWSSDK REQUIRED COMPONENTS ${SERVICE_COMPONENTS})

set(ALL_SRC_FILES
    src/file_upload/file_manager_strategy.cpp
    ) 

add_library(${PROJECT_NAME} SHARED ${ALL_SRC_FILES})
target_link_libraries(${PROJECT_NAME} ${AWSSDK_LINK_LIBRARIES} ${dataflow_lite_LIBRARIES} stdc++fs)
target_include_directories(${PROJECT_NAME}
        PRIVATE ${AWSSDK_INCLUDE_DIRS} include ${aws_common_INCLUDE_DIRS} ${dataflow_lite_INCLUDE_DIRS})
target_include_directories(${PROJECT_NAME} PUBLIC
        $<BUILD_INTERFACE:${CMAKE_CURRENT_SOURCE_DIR}/include>
        $<INSTALL_INTERFACE:include>)

#############
## Install ##
#############
install(TARGETS ${PROJECT_NAME}
        EXPORT "${PROJECT_NAME}-targets"
        ARCHIVE DESTINATION lib
        LIBRARY DESTINATION lib
        RUNTIME DESTINATION bin)

install(DIRECTORY include/
        DESTINATION include)

export(TARGETS ${PROJECT_NAME}
        FILE "${PROJECT_BINARY_DIR}/${PROJECT_NAME}-targets.cmake")
export(PACKAGE ${PROJECT_NAME})

CONFIGURE_FILE(${PROJECT_NAME}Config.cmake.in
               ${PROJECT_BINARY_DIR}/${PROJECT_NAME}Config.cmake @ONLY)
CONFIGURE_FILE(${PROJECT_NAME}ConfigVersion.cmake.in
               ${PROJECT_BINARY_DIR}/${PROJECT_NAME}ConfigVersion.cmake @ONLY)

INSTALL(FILES ${PROJECT_BINARY_DIR}/${PROJECT_NAME}Config.cmake DESTINATION share/${PROJECT_NAME}/cmake)
INSTALL(FILES ${PROJECT_BINARY_DIR}/${PROJECT_NAME}ConfigVersion.cmake DESTINATION share/${PROJECT_NAME}/cmake)
INSTALL(EXPORT ${PROJECT_NAME}-targets DESTINATION share/${PROJECT_NAME}/cmake)


#############
## Testing ##
#############

enable_testing()
find_common_test_packages()
if(GMOCK_LIBRARY)

  macro(link_test_target target_name)
    if(TARGET ${target_name})
      target_include_directories("${target_name}" PRIVATE ${HEADERS_FOR_TESTS})
      target_link_libraries("${target_name}" ${LIBS_FOR_TESTS})
    endif()
  endmacro()

  add_common_gtest(test_common_utilities
          test/file_management/file_manager_strategy_test.cpp
          test/file_management/file_upload_streamer_test.cpp
          test/file_management/token_test.cpp
          )

  set(LIBS_FOR_TESTS
          ${GTEST_LIBRARIES}
          pthread
          ${GMOCK_LIBRARY}
          ${PROJECT_NAME}
          )

  set(HEADERS_FOR_TESTS
          PRIVATE include
          PRIVATE ${aws_common_INCLUDE_DIRS}
          PUBLIC ${PROJECT_SOURCE_DIR}/test/include
          )

  link_test_target(test_common_utilities)

endif()
