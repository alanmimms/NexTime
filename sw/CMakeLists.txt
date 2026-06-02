# SPDX-License-Identifier: Apache-2.0
# WSPR-ease Zephyr Application

cmake_minimum_required(VERSION 3.20.0)

# Include home WiFi credentials if they exist but keep
# them out of git repo.
set(SECRETS_FILE "$ENV{HOME}/.zephyr_secrets/home-wifi.conf")

if(EXISTS ${SECRETS_FILE})
    message(STATUS "Injecting Wi-Fi credentials from ${SECRETS_FILE}")
    list(APPEND EXTRA_CONF_FILE ${SECRETS_FILE})
else()
    message(WARNING "Secrets file not found at ${SECRETS_FILE}. Wi-Fi connection will fail.")
endif()

find_package(Zephyr REQUIRED HINTS $ENV{ZEPHYR_BASE})
project(wspr-ease)

# Include directories
target_include_directories(app PRIVATE
  ${CMAKE_CURRENT_SOURCE_DIR}/src

  # For regs.hpp
  ${CMAKE_CURRENT_SOURCE_DIR}/../FPGA
)

# Calculate paths dynamically to remain generalized
set(toolsDirPath "${CMAKE_CURRENT_SOURCE_DIR}/../tools")
set(flashScriptPath "${toolsDirPath}/remote-flash.sh")
set(binFilePath "${CMAKE_BINARY_DIR}/zephyr/zephyr.bin")

# Define the custom target that depends on the final zephyr binary
add_custom_target(remoteFlash
  COMMAND bash ${flashScriptPath} ${binFilePath}
  DEPENDS zephyr_final
)

# Source files
target_sources(app PRIVATE
  src/main.cpp
  src/wifiManager.cpp
  src/webserver.cpp
  src/captiveDNS.cpp
  src/gnss.cpp
  src/fpga.cpp
  src/shellcmds.cpp
  src/filesystem.cpp
  src/logmanager.cpp
)
