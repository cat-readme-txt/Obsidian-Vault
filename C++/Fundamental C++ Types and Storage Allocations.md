|Category|Type|Typical bits|Typical bytes|Typical range / precision|Important notes|
|---|---|--:|--:|---|---|
|Boolean|`bool`|8*|1|`false`, `true`|Logically only needs 1 bit, but typically occupies 1 byte|
|Character|`char`|8|1|implementation-dependent signedness|Always exactly `1` C++ byte|
|Character|`signed char`|8|1|−128 to 127|Small integer type; explicitly signed|
|Character|`unsigned char`|8|1|0 to 255|Often used to inspect raw bytes|
|Character|`char8_t`|8|1|UTF-8 code unit|C++20; distinct type|
|Character|`char16_t`|16|2|UTF-16 code unit|At least 16 bits|
|Character|`char32_t`|32|4|UTF-32 code unit|At least 32 bits|
|Character|`wchar_t`|16 or 32|2 or 4|implementation-dependent|Usually 16-bit Windows, 32-bit Linux/macOS|
|Integer|`short`|16|2|−32,768 to 32,767|At least 16 bits|
|Integer|`unsigned short`|16|2|0 to 65,535|No negative values|
|Integer|`int`|32|4|−2,147,483,648 to 2,147,483,647|At least 16 bits; usually 32|
|Integer|`unsigned int`|32|4|0 to 4,294,967,295|Arithmetic wraps modulo $2^N$|
|Integer|`long`|32 or 64|4 or 8|platform-dependent|32-bit Windows; usually 64-bit Linux/macOS|
|Integer|`unsigned long`|32 or 64|4 or 8|$0$ to $2^N-1$|Same width as `long`|
|Integer|`long long`|64|8|about ±9.22 × 10¹⁸|At least 64 bits|
|Integer|`unsigned long long`|64|8|0 to about 1.84 × 10¹⁹|Usually 64 bits|
|Floating point|`float`|32|4|~7 decimal digits|Usually IEEE-754 binary32|
|Floating point|`double`|64|8|~15–16 decimal digits|Usually IEEE-754 binary64|
|Floating point|`long double`|64, 80, or 128|8, 12/16, or 16|implementation-dependent|Varies substantially by platform|
|Special|`void`|—|—|no values|Cannot create a normal `void` object|
|Null pointer|`std::nullptr_t`|typically 64|typically 8|null pointer value|Type of `nullptr`|