cmake_minimum_required(VERSION 3.22)

project(DiscordSDK)


set(CMAKE_CXX_STANDARD 20)
set(CMAKE_CXX_STANDARD_REQUIRED ON)

set(DISCORD_SDK_VERSION "3.2.1")
set(DISCORD_SDK_LANG "cpp")
set(DISCORD_SDK_ARCH "x86_64")

set(DISCORD_ZIP_NAME "discord_game_sdk.zip")
set(DISCORD_EXTRACT_FOLDER_NAME "discordsdk")
set(DISCORD_EXTRACT_FOLDER ${CMAKE_CURRENT_BINARY_DIR}/${DISCORD_EXTRACT_FOLDER_NAME})
set(DISCORD_LANG_FOLDER ${DISCORD_EXTRACT_FOLDER}/${DISCORD_SDK_LANG})
set(DISCORD_LIB_FOLDER ${DISCORD_EXTRACT_FOLDER}/lib/${DISCORD_SDK_ARCH})
set(DISCORD_SDK_DOWNLOAD_PATH "https://dl-game-sdk.discordapp.net/${DISCORD_SDK_VERSION}/discord_game_sdk.zip")

if(NOT EXISTS ${CMAKE_CURRENT_BINARY_DIR}/${DISCORD_ZIP_NAME})
    message(STATUS "Downloading DiscordSDK...")
    file(DOWNLOAD ${DISCORD_SDK_DOWNLOAD_PATH} ${DISCORD_ZIP_NAME})
    message(STATUS "Finish downloading DiscordSDK!")
else()
    message(STATUS "DiscordSDk has been already downloaded!")
endif()

file(ARCHIVE_EXTRACT INPUT ${DISCORD_ZIP_NAME} DESTINATION ${DISCORD_EXTRACT_FOLDER_NAME})
file(GLOB DISCORD_SDK_SOURCES ${DISCORD_LANG_FOLDER}/*.cpp ${DISCORD_LANG_FOLDER}/*.h ${DISCORD_LANG_FOLDER}/*.cs)

add_library(discordsdklib SHARED IMPORTED)
if(WIN32 OR MINGW OR MSVC)
    configure_file(${DISCORD_LIB_FOLDER}/discord_game_sdk.dll ${CMAKE_CURRENT_BINARY_DIR}/discord_game_sdk.dll COPYONLY)
    configure_file(${DISCORD_LIB_FOLDER}/discord_game_sdk.lib ${CMAKE_LIBRARY_OUTPUT_DIRECTORY}/discord_game_sdk.lib COPYONLY)
    set_target_properties(discordsdklib PROPERTIES IMPORTED_LOCATION ${CMAKE_CURRENT_BINARY_DIR}/discord_game_sdk.dll IMPORTED_IMPLIB ${CMAKE_LIBRARY_OUTPUT_DIRECTORY}/discord_game_sdk.lib)
elseif(UNIX)
    configure_file(${DISCORD_LIB_FOLDER}/discord_game_sdk.so ${CMAKE_LIBRARY_OUTPUT_DIRECTORY}/discord_game_sdk.so COPYONLY)
    set_target_properties(discordsdklib PROPERTIES IMPORTED_LOCATION ${DISCORD_LIB_FOLDER}/discord_game_sdk.so)
endif()
add_library(discordsdk SHARED ${DISCORD_SDK_SOURCES})
target_include_directories(discordsdk PUBLIC ${DISCORD_LIB_FOLDER})
target_link_libraries(discordsdk PRIVATE discordsdklib)
