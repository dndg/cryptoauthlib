if(CONFIG_CRYPTOAUTHLIB)

    # set(ATCACERT_COMPCERT_EN "")
    # set(ATCACERT_FULLSTOREDCERT_EN "")
    # set(HAS_MALLOC "ON")
    # set(HAS_FREE "ON")
    set(ATCA_USE_ATCAB_FUNCTIONS "ON")

    if(CONFIG_CRYPTOAUTHLIB_HAL_I2C)
        set(ATCA_HAL_I2C "ON")
    endif()

    if(CONFIG_CRYPTOAUTHLIB_NO_POLL)
        set(ATCA_NO_POLL "ON")
    endif()

    if(CONFIG_CRYPTOAUTHLIB_NO_HEAP)
        set(ATCA_NO_HEAP "ON")
    else()
        #set(ATCA_PLATFORM_MALLOC "malloc")
        #set(ATCA_PLATFORM_FREE "free")
    endif()

    if(CONFIG_CRYPTOAUTHLIB_POST_DELAY_MSEC)
        set(ATCA_POST_DELAY_MSEC "${CONFIG_CRYPTOAUTHLIB_POST_DELAY_MSEC}")
    else()
        set(ATCA_POST_DELAY_MSEC "25")
    endif()

    if(CONFIG_CRYPTOAUTHLIB_ATECC608A_SUPPORT)
        set(ATCA_ATECC608_SUPPORT "ON")
        set(ATCA_ATECC608A_SUPPORT "ON")
    endif()

    if(CONFIG_CRYPTOAUTHLIB_ATECC608B_SUPPORT)
        set(ATCA_ATECC608_SUPPORT "ON")
    endif()

    configure_file(
        "${CMAKE_CURRENT_LIST_DIR}/../lib/atca_config.h.in"
        "${CMAKE_CURRENT_BINARY_DIR}/include/atca_config.h")
    file(COPY
        "${CMAKE_CURRENT_BINARY_DIR}/include/atca_config.h"
        DESTINATION "${CMAKE_CURRENT_BINARY_DIR}/include/cryptoauthlib")

    file(GLOB LIB_SRC "${CMAKE_CURRENT_LIST_DIR}/../lib/*.c")
    file(GLOB ATCACERT_SRC "${CMAKE_CURRENT_LIST_DIR}/../lib/atcacert/*.c")
    file(GLOB CALIB_SRC "${CMAKE_CURRENT_LIST_DIR}/../lib/calib/*.c")
    file(GLOB CRYPTO_SRC "${CMAKE_CURRENT_LIST_DIR}/../lib/crypto/*.c")
    file(GLOB CRYPTO_HASHES_SRC "${CMAKE_CURRENT_LIST_DIR}/../lib/crypto/hashes/*.c")

    include_directories("${CMAKE_CURRENT_LIST_DIR}/../lib")
    include_directories("${CMAKE_CURRENT_LIST_DIR}/../lib/atcacert")
    include_directories("${CMAKE_CURRENT_LIST_DIR}/../lib/calib")
    include_directories("${CMAKE_CURRENT_LIST_DIR}/../lib/crypto")
    include_directories("${CMAKE_CURRENT_LIST_DIR}/../lib/crypto/hashes")
    include_directories("${CMAKE_CURRENT_BINARY_DIR}/include")

    # Copy header files and add some prefixes to make sure everything
    # is aligned to Zephyr external modules policy.

    function(replace_include_prefix FILE PREFIX)
        file(READ ${FILE} SRC)
        string(REPLACE "#include \"${PREFIX}" "#include \"cryptoauthlib/${PREFIX}" DST "${SRC}")
        file(WRITE ${FILE} "${DST}")
    endfunction()

    file(GLOB LIB_H "${CMAKE_CURRENT_LIST_DIR}/../lib/*.h")
    file(GLOB ATCACERT_H "${CMAKE_CURRENT_LIST_DIR}/../lib/atcacert/*.h")
    file(GLOB CALIB_H "${CMAKE_CURRENT_LIST_DIR}/../lib/calib/*.h")
    file(GLOB CRYPTO_H "${CMAKE_CURRENT_LIST_DIR}/../lib/crypto/*.h")
    file(GLOB CRYPTO_HASHES_H "${CMAKE_CURRENT_LIST_DIR}/../lib/crypto/hashes/*.h")
    file(GLOB HAL_H "${CMAKE_CURRENT_LIST_DIR}/../lib/hal/atca_hal.h")

    file(COPY ${LIB_H} DESTINATION "${CMAKE_CURRENT_BINARY_DIR}/include/cryptoauthlib")
    file(COPY ${ATCACERT_H} DESTINATION "${CMAKE_CURRENT_BINARY_DIR}/include/cryptoauthlib/atcacert")
    file(COPY ${CALIB_H} DESTINATION "${CMAKE_CURRENT_BINARY_DIR}/include/cryptoauthlib/calib")
    file(COPY ${CRYPTO_H} DESTINATION "${CMAKE_CURRENT_BINARY_DIR}/include/cryptoauthlib/crypto")
    file(COPY ${CRYPTO_HASHES_H} DESTINATION "${CMAKE_CURRENT_BINARY_DIR}/include/cryptoauthlib/crypto/hashes")
    file(COPY ${HAL_H} DESTINATION "${CMAKE_CURRENT_BINARY_DIR}/include/cryptoauthlib/hal")

    replace_include_prefix("${CMAKE_CURRENT_BINARY_DIR}/include/cryptoauthlib/atca_helpers.h" "")
    replace_include_prefix("${CMAKE_CURRENT_BINARY_DIR}/include/cryptoauthlib/atca_basic.h" "")
    replace_include_prefix("${CMAKE_CURRENT_BINARY_DIR}/include/cryptoauthlib/hal/atca_hal.h" "")
    replace_include_prefix("${CMAKE_CURRENT_BINARY_DIR}/include/cryptoauthlib/calib/calib_config_check.h" "")
    replace_include_prefix("${CMAKE_CURRENT_BINARY_DIR}/include/cryptoauthlib/calib/calib_device.h" "atca")
    replace_include_prefix("${CMAKE_CURRENT_BINARY_DIR}/include/cryptoauthlib/calib/calib_execution.h" "atca")
    replace_include_prefix("${CMAKE_CURRENT_BINARY_DIR}/include/cryptoauthlib/calib/calib_packet.h" "atca")
    replace_include_prefix("${CMAKE_CURRENT_BINARY_DIR}/include/cryptoauthlib/crypto/atca_crypto_sw.h" "")
    replace_include_prefix("${CMAKE_CURRENT_BINARY_DIR}/include/cryptoauthlib/crypto/crypto_sw_config_check.h" "atca")
    replace_include_prefix("${CMAKE_CURRENT_BINARY_DIR}/include/cryptoauthlib/crypto/crypto_hw_config_check.h" "")
    replace_include_prefix("${CMAKE_CURRENT_BINARY_DIR}/include/cryptoauthlib/crypto/atca_crypto_hw_aes.h" "cryptoauthlib")

    zephyr_include_directories("${CMAKE_CURRENT_BINARY_DIR}/include")

    zephyr_library()
    zephyr_library_sources(
        "${LIB_SRC}"
        "${CALIB_SRC}"
        "${CRYPTO_SRC}"
        "${CRYPTO_HASHES_SRC}"
        "${CMAKE_CURRENT_LIST_DIR}/../lib/hal/atca_hal.c"
        "${CMAKE_CURRENT_LIST_DIR}/../third_party/hal/zephyr/hal_zephyr.c"
    )
    zephyr_library_sources_ifdef(
        CONFIG_CRYPTOAUTHLIB_HAL_I2C
        "${CMAKE_CURRENT_LIST_DIR}/../third_party/hal/zephyr/hal_zephyr_i2c.c")

endif()
