#
# Nicholas Yue nicholas_yue@users.sourceforge.net
#
# Note:
# (1) A dummy file include/config.h is required (remance from configure)
# (2) Default installation directory is /usr/local, override with -DCMAKE_INSTALL_PREFIX="" during cmake
#     invocation
# (3) Do the usual "make clean all" to build the library
# (5) Need to include changes in include/log4cpp/Portability.hh for OSX to build
cmake_minimum_required( VERSION 3.0)
project ( Log4cpp )

set(CMAKE_VERSION_MAJOR 1)
set(CMAKE_VERSION_MINOR 1)
set(CMAKE_PATCH_VERSION 4)

if( NOT DEFINED CMAKE_DEBUG_POSTFIX)
  set (CMAKE_DEBUG_POSTFIX "D" CACHE STRING "Suffix to be appeneded before the extension to a debug output")
endif()

set(SOURCES
  src/Appender.cpp
  src/AppenderSkeleton.cpp
  src/AppendersFactory.cpp
  src/BufferingAppender.cpp
  src/FactoryParams.cpp
  src/LayoutsFactory.cpp
  src/LevelEvaluator.cpp
  src/Localtime.cpp
  src/PassThroughLayout.cpp
  src/TriggeringEventEvaluatorFactory.cpp
  src/LayoutAppender.cpp
  src/FileAppender.cpp
  src/DailyRollingFileAppender.cpp
  src/RollingFileAppender.cpp
  src/FixedContextCategory.cpp
  src/IdsaAppender.cpp
  src/OstreamAppender.cpp
  src/StringQueueAppender.cpp
  src/SyslogAppender.cpp
  src/RemoteSyslogAppender.cpp
  src/SimpleLayout.cpp
  src/BasicLayout.cpp
  src/PatternLayout.cpp
  src/Category.cpp
  src/CategoryStream.cpp
  src/HierarchyMaintainer.cpp
  src/Configurator.cpp
  src/BasicConfigurator.cpp
  src/SimpleConfigurator.cpp
  src/PropertyConfigurator.cpp
  src/PropertyConfiguratorImpl.cpp
  src/LoggingEvent.cpp
  src/Priority.cpp
  src/NDC.cpp
  src/Filter.cpp
  src/TimeStamp.cpp
  src/StringUtil.cpp
  src/Properties.cpp
  src/Win32DebugAppender.cpp
  src/NTEventLogAppender.cpp
  src/DllMain.cpp
  src/DummyThreads.cpp
  src/MSThreads.cpp
  src/OmniThreads.cpp
  src/PThreads.cpp
  src/PortabilityImpl.cpp
  src/AbortAppender.cpp
)

option(LOG4CPP_BUILD_SHARED "Toggle building of static or shared library variant" ON)
if (LOG4CPP_BUILD_SHARED)
	add_library ( Log4cpp  SHARED ${SOURCES} )
	target_compile_definitions(Log4cpp PUBLIC  LOG4CPP_HAS_DLL )
else()
	add_library ( Log4cpp  STATIC ${SOURCES} )
endif()

if (WIN32)
  target_compile_definitions ( Log4cpp  PRIVATE -D_CRT_SECURE_NO_WARNINGS )
  target_link_libraries (Log4cpp ws2_32  )
  set_target_properties(Log4cpp PROPERTIES   DEFINE_SYMBOL LOG4CPP_BUILD_DLL )
elseif (APPLE)
  target_compile_definitions ( Log4cpp PRIVATE -DNDEBUG -DLOG4CPP_HAVE_SSTREAM )
else ()
  target_compile_definitions ( Log4cpp PRIVATE -DLOG4CPP_HAVE_UNISTD_H -DNDEBUG  -DLOG4CPP_HAVE_SSTREAM )
  target_compile_options( Log4cpp PRIVATE -pthread )
endif ()

#set_target_properties(Log4cpp PROPERTIES CXX_STANDARD 98 )
target_include_directories (Log4cpp PRIVATE ${CMAKE_CURRENT_SOURCE_DIR}
                                    PUBLIC $<BUILD_INTERFACE:${CMAKE_CURRENT_SOURCE_DIR}/include>
                                    INTERFACE $<INSTALL_INTERFACE:include>
  )

install (
  DIRECTORY include/log4cpp
  DESTINATION include
  PATTERN "config.h" EXCLUDE
  PATTERN ".svn" EXCLUDE
  PATTERN "*.am" EXCLUDE
  PATTERN "*.in" EXCLUDE
  )

install (
  TARGETS Log4cpp EXPORT Log4cpp
  LIBRARY DESTINATION lib
  ARCHIVE DESTINATION lib
  RUNTIME DESTINATION bin
  )

install(EXPORT Log4cpp
  NAMESPACE Log4cpp::
  FILE Log4cpp.cmake
  DESTINATION lib/cmake/Log4cpp
)

include(CMakePackageConfigHelpers)
# generate the config file that is includes the exports
configure_package_config_file(${PROJECT_SOURCE_DIR}/cmake/log4cppConfig.cmake.in
  "${CMAKE_CURRENT_BINARY_DIR}/Log4cppConfig.cmake"
  INSTALL_DESTINATION "lib/cmake/Log4cpp"
  NO_SET_AND_CHECK_MACRO
  NO_CHECK_REQUIRED_COMPONENTS_MACRO
)

# generate the version file for the config file
write_basic_package_version_file(
  "${CMAKE_CURRENT_BINARY_DIR}/Log4cppConfigVersion.cmake"
  VERSION "${CMAKE_VERSION_MAJOR}.${CMAKE_VERSION_MINOR}"
  COMPATIBILITY AnyNewerVersion
)

# install the configuration file
install(FILES
  ${CMAKE_CURRENT_BINARY_DIR}/Log4cppConfig.cmake
  DESTINATION lib/cmake/Log4cpp
)
