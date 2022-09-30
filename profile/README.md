
# COMBUS C++ Standard

## C++ Version and Compilation

* The following guide has been created to provide a useful navigational tool for developing concise, readable, reproducible and - most importantly - safe and bug-prone code. Almost all issues of model output encountered over the last 5 years of COMBUS C++ development can be *directly* attributed to simple bugs, derived from notation and obscure data type declarations. Instances where incorrect COMBUS model outputs were caused by mathematical design faults have been the exception to this rule. 
An effort should be made by COMBUS programmers to follow the following paradigms, even if implementation is notably more time consuming

*  COMBUS C++ programming is done through Visual Studio 2022 (pending further releases). All newly made projects should be created via the COMBUS C++ template (preloaded on all current COMBUS development computers). For new systems, located the template found in "**R:/COMBUS C++ Project.zip**" and copy into the Visual Studio template folder

* The COMBUS C++ template is setup to use the C++20 standard with the following noteworthy compilation options:
	* -Werror enabled for release mode, disabled for debug mode
	* .clang-tidy formatting file using the modified Google C++ standard
	* clang-warnings enabled
	* Automatic inclusion of the COMBUS C++ library folder containing relevant .h files (located in **"R:/Libraries"**) 

* **-Werror should be on at all times**

* Much of this standard is derived from the Google C++ style guide; some rules deemed critical are copied verbatum.  If a programming style paradigm is absent from this guide, search in https://google.github.io/styleguide/cppguide.html


## Storing Data

### Local Variables

Place function's variables in the narrowest scope possible. Avoid as much as possible initialisations separate from declarations:
```c++
int i;
i = f(); //BAD

int j = f(); //GOOD

auto v = std::vector<int>();
v.push_back(1);
v.push_back(2); //BAD

auto v2 = std::vector<int>({1,2}); //GOOD and prefer list initialisation
```

A notable exception to this is where a constructor is invoked repeatedly each time it enters scope where it is more efficient to separate the declaration. In these cases consider wrapping the code in a scope brace `{...}`
```c++
// Inefficient implementation:
for (int i = 0; i < 1000000; ++i) {
  Foo f;  // My ctor and dtor get called 1000000 times each.
  f.DoSomething(i);
}

//More efficient implementation
{
	Foo f;  // My ctor and dtor get called once each.
	for (int i = 0; i < 1000000; ++i) {
	f.DoSomething(i);
	}
	
}//f destructor called
```


### Fundamental Types

* All fundamental types - `int, long long, float, double,...` should be explicitly initialised and *not* infered through the `auto` keyword. Initialisation should be performed via list initialisation *or* a static cast using `auto`. ie. it should be obvious by immediate observation what the precision is of all fundamental variables
```c++
auto i = 1;   //BAD! Not explicitly stating what i is
auto j = 2.0; //BAD! Not explicitly stating the precision of the floating point

auto k = static_cast<int>(1); //Good! obvious i is an int (but somewhat lengthy)
int l{1};                     //Even Better! obvious and concise

int m = 1;  //Fine, BUT be careful when the RHS is a complex equation which might
			//not be able to cast to an int without losing data. ALWAYS prefer
			//brace initialisation here
int n = 10000000 * 1000000; //TERRIBLE, causes overflow!
int n { 10000000 * 1000000 }; //ERROR braces pick up overflow, GOOD!
```

* Unless *specifically* required for size, avoid the use of unsigned values, especially when used in `for` loops

### Strings

Avoid the use of std::string whenever using read-only strings - prefer std::string_view for space and computational efficiency

```c++
auto print_string(std::string b) -> void {
	std::cout << b;
}
auto print_stringV(std::string_view b) -> void {
	std::cout << b;
}
std::string a{"My String"};
print_string(a); //BAD! 2 slow copies of a char* occur here

std::string_view b{"My String View"};
print_stringV(b)     ; //Good! a single char* is constructed and 'viewed' through b       
```

### Containers
Avoid the use of C-style arrays via `malloc` or `new` unless where explicitly required. Prefer standard C++ containers such as vectors or arrays. For constant or preinitialised data, use `std::array`; for dynamic data use `std::vector`

Use `std::map` and `std::set` only where required *and* where order matters. Always prefer `std::unordered_map` and `std::unordered_set` if order is irrelevant.



