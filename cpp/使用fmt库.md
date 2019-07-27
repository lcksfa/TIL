# 使用可能成为c++标准库的 fmt格式字符串库

## 先看看文档

[api使用手册](https://fmt.dev/latest/api.html)

## 基础使用

1. 类似printf的打印,sprintf的格式化

   ```c++
   fmt::print("Don't {}\n", "panic");
   fmt::print("I'd rather be {1} than {0}.\n", "right", "happy");
   fmt::printf("Elapsed time: %.2f seconds", 1.23);
   std::string message = fmt::sprintf("The answer is %d", 42);
   ```
   
2. 打印到文件

   ```c++
   fmt::fprintf(stderr, "Don't %s!", "panic");
   ```

   ```c++
   FILE *fp = nullptr;
   fopen_s(&fp, "test.txt", "w+");
   if (!fp) {
       fmt::fprintf(stderr, "text.txt open failed!");
   }
   fmt::fprintf(fp, "Hello %s", "World.");
   ```

3. 写入数据

   ```c++
   std::vector<wchar_t> result;
   fmt::format_to(back_inserter(result), L"{}", 42);
   ```

4. 无内存分配创建字符串

   ```c++
   fmt::basic_memory_buffer<char, 100> buffer;
   fmt::format_to(buffer, "{}", 52);
   auto result2 = fmt::to_string(buffer);
   ```

5. 用户自定义语法(初阶)

   ```c++
   // USER-DEFINED LITERALS
   using namespace fmt::literals;
   std::wstring message = L"The answer is {}"_format(42);
   EXPECT_EQ(message, L"The answer is 42");
   
   auto udlstr = fmt::format("The answer is {the_answer}", "the_answer"_a = 42);
   // equl to format(L"The answer is {the_answer}", arg("the_answer", 42));
   EXPECT_EQ(udlstr, "The answer is 42");
   
   auto uslstr2 = fmt::format("My name is {my_name},i am {old} years old", "my_name"_a = "lcksfa","old"_a = 29);
   EXPECT_EQ(uslstr2, "My name is lcksfa,i am 29 years old");
   
   std::string message2 = "{0}{1}{0}"_format("abra", "cad");
   EXPECT_EQ(message2, "abracadabra");
   ```

6. 用户自定义语法(高阶)

7. 

8. 

9. 

10. 

11. 

## source 

    [so上的问答](https://stackoverflow.com/questions/tagged/fmt)

