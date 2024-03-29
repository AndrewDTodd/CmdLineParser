#CmdLineParser tests target section
################################################################################################################################################################
#installation and setup of the gTest suite
#build a tests executable for the execution of the projects tests
include(FetchContent)
FetchContent_Declare(
	googletest
	DOWNLOAD_EXTRACT_TIMESTAMP true
	URL https://github.com/google/googletest/archive/refs/tags/v1.14.0.zip
)

#For Windows: Prevent overriding the parent project's compiler/linker settings
set(gtest_force_shared_crt ON CACHE BOOL "" FORCE)
#new format for including googletest subdirectory. To prevent googletest items being added to install
FetchContent_GetProperties(googletest)
if(NOT googletest_POPULATED)
	FetchContent_Populate(googletest)
	add_subdirectory(${googletest_SOURCE_DIR} ${googletest_BINARY_DIR} EXCLUDE_FROM_ALL)
endif()

enable_testing()
include(GoogleTest)

#create a set for build all tests
set(TEST_TARGETS)

#create a set for the Flag aggregate types unit tests
set(FLAG_AGGREGATE_TYPES_TARGETS)
#get a list of the .cpp files in the subdirectory for the unit tests
file(GLOB FLAG_AGGREGATE_TYPES_SOURCES "${CMAKE_CURRENT_SOURCE_DIR}/unit/Flag/flag_aggregate/*.cpp")

#add each source file as a test target
foreach(TEST_SOURCE ${FLAG_AGGREGATE_TYPES_SOURCES})
	get_filename_component(TEST_NAME ${TEST_SOURCE} NAME_WE)
	add_executable(${TEST_NAME} EXCLUDE_FROM_ALL ${TEST_SOURCE})
	target_link_libraries(${TEST_NAME} PRIVATE GTest::gtest_main CmdLineParser)
	target_include_directories(${TEST_NAME} PRIVATE ${googletest_SOURCE_DIR}/googletest/include)
	target_include_directories(${TEST_NAME} PRIVATE ${CmdLineParser_SOURCE_DIR}/include)
	list(APPEND FLAG_AGGREGATE_TYPES_TARGETS ${TEST_NAME})
	list(APPEND TEST_TARGETS ${TEST_NAME})
	gtest_discover_tests(${TEST_NAME})
	set_target_properties(${TEST_NAME} PROPERTIES FOLDER Tests/flag_aggregates/)
endforeach()

#create a custom target for the flag_aggregate types tests
add_custom_target(flag_aggregate_tests DEPENDS ${FLAG_AGGREGATE_TYPES_TARGETS})

#create a custom target for all tests
add_custom_target(build_all_tests DEPENDS ${TEST_TARGETS})
################################################################################################################################################################

#add tests to be discoverable by ctest *Note this is only necessary when not using gtest_discover. The tests are automatically added by gtest
################################################################################################################################################################
#add_test(NAME FlagArgument_UnitTests COMMAND FlagArgument_UnitTests)