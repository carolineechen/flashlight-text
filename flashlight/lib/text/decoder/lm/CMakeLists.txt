cmake_minimum_required(VERSION 3.10)

set(FL_TEXT_KENLM_MAX_ORDER 6 CACHE STRING "Maximum ngram order for KenLM")

target_sources(
  flashlight-text
  PRIVATE
  ${CMAKE_CURRENT_LIST_DIR}/ConvLM.cpp
  ${CMAKE_CURRENT_LIST_DIR}/ZeroLM.cpp
  )

# ------------------------- KenLM-specific -------------------------

if (FL_TEXT_USE_KENLM)
  find_package(kenlm)
  if (NOT kenlm_FOUND)
    if (FL_TEXT_BUILD_STANDALONE)
      message(STATUS "KenLM not found - will download and build from source")
      include(${CMAKE_MODULE_PATH}/BuildKenlm.cmake)
    else()
      message(FATAL_ERROR "KenLM not found but FL_TEXT_USE_KENLM enabled. "
        "Install KenLM or set the KENLM_ROOT environment variable.")
    endif()
  endif()
  if (FL_TEXT_BUILD_STANDALONE)
    # Install the module so downstream projects can find the lib
    install(
      FILES ${CMAKE_MODULE_PATH}/Findkenlm.cmake
      DESTINATION ${FL_INSTALL_CMAKE_DIR}
      )
  endif()

  target_sources(
    flashlight-text
    PRIVATE
    ${CMAKE_CURRENT_LIST_DIR}/KenLM.cpp
    )

  target_link_libraries(
    flashlight-text
    PRIVATE
    kenlm::kenlm
    kenlm::kenlm_util
    )

  target_compile_definitions(
    flashlight-text
    PRIVATE
    KENLM_MAX_ORDER=${FL_TEXT_KENLM_MAX_ORDER}
    )
endif()
