
# COMBUS C++ Standard

## C++ Version and Compilation


## Storing Data

### Fundamental Types

* All fundamental types - `int, long long, float, double,...` should be explicitly initialised and not infered through the auto type (unless statically cast). This should be performed either via list initialisation OR a static cast. For example:
```c++
auto i = 1;   //BAD! Not explicitly stating what i is
auto j = 2.0; //BAD! Not explicitly stating the precision of the floating point

auto i = static_cast<int>(1); //Good! obvious i is an int (but somewhat lengthy)
int i{1};                     //Even Better! obvious and concise

int i = 1;  //BAD! Why? What happens if the rvalue is a complicated expression which 
			//cannot be cast to an int without loss of data? See list initialisation
```

* Unless *specifically* required for size, avoid the use of unsigned values, especially when used in `for` loops

*  x

### Strings

* Avoid the use of std::string wherever possible - prefer std::string_view for space and computational efficiency

* 
