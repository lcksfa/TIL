# 使用catch2用于TDD

## 安装catch2

直接使用vcpkg安装即可

## 建立工程

vscode需要安装cmake相关的插件

1. 新建一个CMakeLists.txt文件	

   ```cmake
   cmake_minimum_required(VERSION 3.13.0)
   project(cmakedemo1 VERSION 0.1.0)
   
   include(CTest)
   enable_testing()
   
   add_executable(cmakedemo1 main.cpp)
   
   
   set(CPACK_PROJECT_NAME ${PROJECT_NAME})
   set(CPACK_PROJECT_VERSION ${PROJECT_VERSION})
   include(CPack)
   add_test(test_all cmakedemo1)
   ```

2. 新建一个main文件

   ```c++
   #define CATCH_CONFIG_MAIN  // This tells Catch to provide a main() - only do this in one cpp file
   #include "catch2\catch.hpp"
    
   unsigned int Factorial(unsigned int number)
   {
      return number <= 1 ? number : Factorial(number - 1)*number;
   }  
      
   TEST_CASE("Factorials are computed", "[factorial]")
    {    
      REQUIRE(Factorial(1) == 1);   
      REQUIRE(Factorial(2) == 2);    
      REQUIRE(Factorial(3) == 6);   
      REQUIRE(Factorial(10) == 3628800); 
   }
   
   ```

   3. 因为我的vcpkg使用的是vs2015的x86的安装环境,所以cmake的环境也要与之相同

   ![1564222128519](assets/1564222128519.png)

   如此，直接点击build可以构建,点击test可以进行单元测试;

