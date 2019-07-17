# 使用cquery

[源码](https://github.com/cquery-project/cquery/wiki/Building-cquery)

windows 下：

```
git clone --recursive https://github.com/cquery-project/cquery.git
cd cquery
git submodule update --init
mkdir build && cd build
cmake .. -DCMAKE_BUILD_TYPE=Release -DCMAKE_INSTALL_PREFIX=release -DCMAKE_EXPORT_COMPILE_COMMANDS=YES
cmake --build . --config release
cmake --build . --target install --config release
```

这里注意要编译为release。
TabNine::