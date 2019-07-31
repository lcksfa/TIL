The package cpprestsdk:x86-windows provides CMake targets:

    find_package(cpprestsdk CONFIG REQUIRED)
    # Note: 1 target(s) were omitted.
    target_link_libraries(main PRIVATE cpprestsdk::cpprest cpprestsdk::cpprestsdk_zlib_internal cpprestsdk::cpprestsdk_boost_internal cpprestsdk::cpprestsdk_openssl_internal)
ok,编译好份时间