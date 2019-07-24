# 使用fmt库


## date的格式化
```c++
#include "gtest\gtest.h"
#include "fmt\time.h"
#include <time.h>
#include <string>

TEST(TESTDEMO1, T1) { EXPECT_EQ(1, 1); }

TEST(TESTFMT,FMT1){
	std::time_t t = std::time(nullptr);
	struct tm newtime;
	localtime_s(&newtime,&t);
	std::string g = fmt::format("The date is {:%Y-%m-%d}.", newtime);
	EXPECT_EQ(g,"The date is 2019-07-24.");
}

int main(int argc, char **argv) {
	::testing::InitGoogleTest(&argc, argv);
	return RUN_ALL_TESTS();
}
```

