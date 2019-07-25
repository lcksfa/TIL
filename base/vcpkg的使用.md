# Windows10上使用vcpkg

:arrow_double_up:  

- mkdir 05-vcpkg-install

- cd .\05-vcpkg-install\

- git clone https://github.com/Microsoft/vcpkg

- cd .\vcpkg\

- .\bootstrap-vcpkg.bat

- ./vcpkg integrate install  （这里是全局集成）`.\vcpkg integrate remove`反集成

提示

`All MSBuild C++ projects can now #include any installed libraries.
Linking will be handled automatically.
Installing new libraries will make them instantly available.

CMake projects should use: "-DCMAKE_TOOLCHAIN_FILE=F:/05-vcpkg-install/vcpkg/scripts/buildsystems/vcpkg.cmake"

`

- 安装gtest  ` .\vcpkg install gtest `

  cmake下使用的说明

  `

  The package gtest is compatible with built-in CMake targets:

      enable_testing()
      find_package(GTest MODULE REQUIRED)
      target_link_libraries(main PRIVATE GTest::GTest GTest::Main)
      add_test(AllTestsInMain main)
  `

- ```
  .\vcpkg integrate powershell
  ```

在vs2015中使用 gtest 报错的解决方法 ：

https://stackoverflow.com/questions/41315739/vcpkg-does-not-work-for-google-test

**The required per project configuration:**
In: `Configuration Properties` > `Linker` > `Input` > `Additional Dependencies`
For release-builds:

```cpp
$(VcpkgRoot)lib\manual-link\gtest_main.lib
```

For debug-builds:

```cpp
$(VcpkgRoot)debug\lib\manual-link\gtest_maind.lib
```

If you want to create your own custom main(), replace `gtest_main.lib` with `gtest.lib`.
If you want to use gmock, you can replace it with `gmock_main.lib` or `gmock.lib`.

