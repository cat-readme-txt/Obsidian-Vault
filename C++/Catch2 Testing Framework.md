```cpp
#include <catch2/catch_test_macros.hpp> // Required for Catch2 v3

// Optional: the second parameter tags
TEST_CASE("Short descriptive name of the test", "[tag1][tag2]") {
	// Optional: setup local test variables
	int a = 5;
	int b = 10;
	
	// Evaluate standard C++ boolean expressions
	REQUIRE(a + b == 15);
}
```