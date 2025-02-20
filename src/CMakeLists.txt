# SPDX-License-Identifier: MIT

include(GNUInstallDirs)
include(CMakePackageConfigHelpers)

add_subdirectory(common)

# initialize KEM|SIG_OBJS for --warn-uninitialized
set(KEM_OBJS "")
set(SIG_OBJS "")

if(${OQS_ENABLE_KEM_BIKE})
    add_subdirectory(kem/bike)
    set(KEM_OBJS ${KEM_OBJS} ${BIKE_OBJS})
endif()
if(${OQS_ENABLE_KEM_FRODOKEM})
    add_subdirectory(kem/frodokem)
    set(KEM_OBJS ${KEM_OBJS} ${FRODOKEM_OBJS})
endif()
if(${OQS_ENABLE_KEM_SIKE} OR ${OQS_ENABLE_KEM_SIDH})
    add_subdirectory(kem/sike)
    set(KEM_OBJS ${KEM_OBJS} $<TARGET_OBJECTS:sike>)
endif()
if(${OQS_ENABLE_SIG_PICNIC})
    add_subdirectory(sig/picnic)
    set(SIG_OBJS ${SIG_OBJS} $<TARGET_OBJECTS:picnic>)
endif()
##### OQS_COPY_FROM_UPSTREAM_FRAGMENT_ADD_ALG_OBJECTS_START
if(OQS_ENABLE_KEM_CLASSIC_MCELIECE)
    add_subdirectory(kem/classic_mceliece)
    set(KEM_OBJS ${KEM_OBJS} ${CLASSIC_MCELIECE_OBJS})
endif()
if(OQS_ENABLE_KEM_HQC)
    add_subdirectory(kem/hqc)
    set(KEM_OBJS ${KEM_OBJS} ${HQC_OBJS})
endif()
if(OQS_ENABLE_KEM_KYBER)
    add_subdirectory(kem/kyber)
    set(KEM_OBJS ${KEM_OBJS} ${KYBER_OBJS})
endif()
if(OQS_ENABLE_KEM_NTRU)
    add_subdirectory(kem/ntru)
    set(KEM_OBJS ${KEM_OBJS} ${NTRU_OBJS})
endif()
if(OQS_ENABLE_KEM_NTRUPRIME)
    add_subdirectory(kem/ntruprime)
    set(KEM_OBJS ${KEM_OBJS} ${NTRUPRIME_OBJS})
endif()
if(OQS_ENABLE_KEM_SABER)
    add_subdirectory(kem/saber)
    set(KEM_OBJS ${KEM_OBJS} ${SABER_OBJS})
endif()
if(OQS_ENABLE_SIG_DILITHIUM)
    add_subdirectory(sig/dilithium)
    set(SIG_OBJS ${SIG_OBJS} ${DILITHIUM_OBJS})
endif()
if(OQS_ENABLE_SIG_FALCON)
    add_subdirectory(sig/falcon)
    set(SIG_OBJS ${SIG_OBJS} ${FALCON_OBJS})
endif()
if(OQS_ENABLE_SIG_RAINBOW)
    add_subdirectory(sig/rainbow)
    set(SIG_OBJS ${SIG_OBJS} ${RAINBOW_OBJS})
endif()
if(OQS_ENABLE_SIG_SPHINCS)
    add_subdirectory(sig/sphincs)
    set(SIG_OBJS ${SIG_OBJS} ${SPHINCS_OBJS})
endif()
##### OQS_COPY_FROM_UPSTREAM_FRAGMENT_ADD_ALG_OBJECTS_END

add_library(oqs kem/kem.c
                ${KEM_OBJS}
                sig/sig.c
                ${SIG_OBJS}
                ${COMMON_OBJS})
set(COMMON_OBJS ${COMMON_OBJS} PARENT_SCOPE)
if(DEFINED SANITIZER_LD_FLAGS)
    target_link_libraries(oqs PUBLIC ${SANITIZER_LD_FLAGS})
endif()
if(${OQS_USE_OPENSSL})
    target_link_libraries(oqs PRIVATE ${OPENSSL_CRYPTO_LIBRARY})
endif()

target_include_directories(oqs
                           PUBLIC
                           "$<BUILD_INTERFACE:${CMAKE_CURRENT_SOURCE_DIR}>"
                           "$<INSTALL_INTERFACE:${CMAKE_INSTALL_INCLUDEDIR}>"
)
set_target_properties(oqs
    PROPERTIES
    ARCHIVE_OUTPUT_DIRECTORY "${CMAKE_BINARY_DIR}/lib"
    LIBRARY_OUTPUT_DIRECTORY "${CMAKE_BINARY_DIR}/lib"
    VERSION ${OQS_VERSION_TEXT}
    SOVERSION 0
    # For Windows DLLs
    RUNTIME_OUTPUT_DIRECTORY "${CMAKE_BINARY_DIR}/bin")

configure_package_config_file(${CMAKE_CURRENT_SOURCE_DIR}/Config.cmake.in
  "${CMAKE_CURRENT_BINARY_DIR}/liboqsConfig.cmake"
  INSTALL_DESTINATION ${CMAKE_INSTALL_LIBDIR}/cmake/liboqs
)

# generate the version file for the config file
write_basic_package_version_file(
  "${CMAKE_CURRENT_BINARY_DIR}/liboqsConfigVersion.cmake"
  VERSION "${OQS_VERSION_TEXT}"
  COMPATIBILITY AnyNewerVersion
)

install(FILES
          "${CMAKE_CURRENT_BINARY_DIR}/liboqsConfig.cmake"
          "${CMAKE_CURRENT_BINARY_DIR}/liboqsConfigVersion.cmake"
        DESTINATION "${CMAKE_INSTALL_LIBDIR}/cmake/liboqs"
)

install(TARGETS oqs
        EXPORT liboqsTargets
        LIBRARY DESTINATION ${CMAKE_INSTALL_LIBDIR}
        ARCHIVE DESTINATION ${CMAKE_INSTALL_LIBDIR}
        RUNTIME DESTINATION ${CMAKE_INSTALL_BINDIR}
        INCLUDES DESTINATION ${CMAKE_INSTALL_INCLUDEDIR}/oqs
)

install(EXPORT liboqsTargets
        FILE liboqsTargets.cmake
        NAMESPACE OQS::
        DESTINATION ${CMAKE_INSTALL_LIBDIR}/cmake/liboqs
)

install(FILES ${PUBLIC_HEADERS}
        DESTINATION ${CMAKE_INSTALL_INCLUDEDIR}/oqs)

export(EXPORT liboqsTargets
       FILE "${CMAKE_CURRENT_BINARY_DIR}/cmake/liboqsTargets.cmake"
       NAMESPACE OQS::
)

