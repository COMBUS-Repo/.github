
# COMBUS C++ Standard

## C++ Version and Compilation

* The following guide has been created to provide a useful navigational tool for developing concise, readable, reproducible and - most importantly - safe and bug-prone code. Almost all issues of COMBUS model outputs encountered over the last 5 years of C++ (and VBA, python, bash, etc.) development can be *directly* attributed to simple bugs, derived from notation and obscure data type declarations. Instances where incorrect COMBUS model outputs were caused by mathematical design faults have been the exception to this rule. 
An effort should be made by COMBUS programmers to follow the following paradigms, even if implementation is notably more time consuming

*  COMBUS C++ programming is done through Visual Studio 2022 (pending further releases). All newly made projects should be created via the COMBUS C++ template (preloaded on all current COMBUS development computers). For new systems, locate the template found in "**R:/COMBUS C++ Project.zip**" and copy into the Visual Studio template folder

* The COMBUS C++ template is setup to use the C++20 standard with the following noteworthy compilation options:
	* -Werror enabled for release mode, disabled for debug mode
	* .clang-tidy formatting file using the modified Google C++ standard
	* clang-warnings enabled
	* Automatic inclusion of the COMBUS C++ library folder containing relevant .h files (located in **"R:/Libraries"**) 

* **-Werror should be on at all times**

* Google ABSL libraries are temporarily disabled pending the release of an ABSL libary compatible with C++17/20 

* Much of this standard is derived from the Google C++ style guide; some rules deemed critical are copied verbatum.  If a programming style paradigm is absent from this guide, search in https://google.github.io/styleguide/cppguide.html


## Storing Data

### Local Variables
Place function's variables in the narrowest scope possible. Avoid as much as possible initialisations separate from declarations:
```cpp
int i;
i = f(); //BAD

int j = f(); //GOOD

auto v = std::vector<int>();
v.push_back(1);
v.push_back(2); //BAD

auto v2 = std::vector<int>({1,2}); //GOOD and prefer list initialisation
```

A notable exception to this is where a constructor is invoked repeatedly each time it enters scope where it is more efficient to separate the declaration. In these cases consider wrapping the code in a scope brace `{...}`
```cpp
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
```cpp
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

```cpp
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


## Naming Convention
Equal in importance only to comments, the consistency and clarity of naming conventions across classes, files and projects is the fundamental backbone of well written code. There are 2 pillars of naming conventions which should be adhered to at all times:
* **Consistency**. Is the naming style common across all code, or is it eclectic and constantly changing? Choice of naming style is largely arbitrary - stick to the same style (this style) irrespective of individual preference
* **Clarity**. All code should be self documenting by the word content of the names themselves. Variable, class, and function names should leave no uncertainty as to their meaning and intent to the next contributor

Of particular note is that, with the existence of thousands of silicon valley coding standards, there is no such thing as a 'best' standard, only consistent standards. If a style is already clear, a better coding style is not one which boast a better use of capitals or underscores for greater coherence over their neighbours, but one which is absolutely reproducible and consistent

After writing a function, declaring a class, or naming a variable, ask oneself if the name adheres to the style convention and, equally importantly, conveys the necessary meaning of the object. If not: rename immediately.

### General Naming Rules
* **Favour intuitive naming over file size.** The meaning of a variable's name is not worth sacrificing for a few bytes of character space. HDDs are cheap (and more importantly we programmers don't pay for it): avoid short and unclear names
```cpp
std::ifstream f("accounts.csv"); //BAD. f is obviously myfile here,
								 //what if we refer to it later?

std::ifstream accounts_f("accounts.csv"); //Better - obviously the accounts file

//AN EXAMPLE FROM COMBUS CODE:
//This function is very obscure in name - someone unsure of areaperils would not 
//understand this. There is no directive or verb to the name and could genuinely perform several different functions
double latitude_apid(long long qk) {
...
}

//Better! In the previous function it is unclear what is being converted in what
//direction. Less ambiguity here
double apid_to_latitude(long long qk) {
...
}

//Even BETTER! No ambiguity here, the function does exactly what it says: gets
//the latitude from an areaperil
double get_areaperil_latitude(long long qk) {
...
}
```

### Specific Notation
Hungarian notation and other hyper-pedantic coding styles have been relegated into obscurity by abstract virtual programming and a general dislike for pedantry. There are, however, a few notable specific notation paradigms which occur frequently enough to elicit common naming features. These and other paradigms are covered below.

### General Variable Conventions
Variables names should contain no uppercases and be entirely lowercase (except in very specific circumstances where the capitals provide direct meaning). Names constituted by multiple words should have these words separated by an underscore.
```cpp
int num_world_tiles{50}; //Good
std::string worldLocation; //Bad
```

### Functions
Functions should contain no uppercases and be entirely lowercase (except in very specific circumstances where the capitals provide direct meaning). As with variables, an underscore differentiates multiple words in a name. Note this constrasts the google c++ standard which uses capitals to differentiate words. Prefer the trailing `auto` return type syntax for all functions. While potentially unfamiliar with new (or very old) C++ users, it's often easier to determine return types for complex templated functions
```cpp
auto foo(int x) -> int{...} //Use this
int foo(int x){...} //Rather than this

//often easier to read when the return type is on the right
template<typename T, typename U>
auto add(T& a, U& b) -> decltype(auto){
	 return a + b; 
}
```

### File Variables
Variable names for `FILE*, std::ifstream, std::ofstream,etc..` should all be succeded with `_f`
```cpp
std::ifstream accounts_f {"accounts.csv"}; //good
std::ifstream accounts {"accounts.csv"}; //bad
``` 

### Constants Constant variables should be preceded with a `k_`
```cpp
const int k_num_week_days { 7 };
```

### Classes and Structs
Class and struct names should start with a capital and contain no underscores. For a class name containing several words, each new world should start with a capital letter. Class names should be clearly identify use and scope; avoid ambiguous names which do not convey a recognisable use and purpose. Minimise the use of abbreviations that would be unknown to someone outside of the immediate project.

There are a number of important paradigms to follow when programming classes, some of these are best considered as good C++ practise transcending a specific coding style.
* Avoid virtual methods in class constructors
* Follow the rule of zero/three/five. First consider the rule of three which states that if any of a classes destructor, copy constructor or copy assignment are required, *all* three need to be defined. If a move operator is required, use the rule of five. This rule states that if any of a classes destructor, copy constructor, copy assignment, move constructor or move assignment are required, *all* five must be defined. 

```cpp
class MyClass{
public:
	//MyClass destructor and move constructor is defined
	//the rule of 5 is then followed
	~MyClass();
	MyClass(const MyClass& a){}
	MyClass(MyClass&& a){}
	auto operator=(const MyClass& a) -> MyClass&{}
	auto operator=(MyClass&& a) noexcept -> MyClass&{}
}
```

* If the copy/move constructors/operators are undesired, ensure to explicitly delete them. When relevant for simple structures, use the default keyword
```cpp
class NoCopyClassOnlyMove{
public:
	~MyClass() = default
	MyClass(const MyClass& a) = delete;
	auto operator=(const MyClass& a) -> MyClass& = delete;
}
```

### Class and Struct Members
Members of classes or structs should be succeded with an underscore `_`. The descriptiveness of a variable's name should be proportional to the variable's scope of visibility. As an example, calling a variable `n` or `i` is often acceptable in a 5 line for loop - within the scope of a class it is obscure.
```cpp
struct FootprintRecord{
	int event_id_;
	unsigned long long apid_;
}; 

class Animal{
	...
private:
	std::string animal_name_;
	//int n_; //BAD. n is obscure!
	int age_; //BETTER, age is more obvious than n_
};
```

### OOP Rules
General OOP rules should apply to all classes. The scope for these rules likely go beyond this guide; however, the SOLID principles are reiterated here for posterity:
* **S**ingle responsibility principle - there should never be more than one reason for a class to change, that is, each class should only have one responsibility. Eg: Prefer a *FootprintWriter* and *FootprintReader* class to a joint FootprintFile class which reads and writes
* **O**pen-closed principle - classes should be open to extension but closed to modification
* **L**iskov substitution principle - functions that use pointers or references to base classes must be able to use objects of derived classes without knowing it
* **I**nterface segregation principle - clients should not be forced to rely on interfaces they do not use
* **D**ependency inversion principle - depend upon abstractions not concretions

### Macros
Using macros in modern C++ is equivalent to using chisel and stone to write a quarterly report - its relevance has long past and there are unquestionably superior alternatives. There are few, if any, good reasons to use `#define` macros in C++, even fewer for COMBUS C++ projects. If macros are deemed necessary, name them in all caps and wrap any complex expressions in brackets to avoid unwanted arithmetic expansions 

## Comments
Comments, given sensibly written code containing consistent naming conventions, keep code readible, and provide longevity for developers and users alike long after the project is written. **REMEMBER**: the best code is self documenting! Comments should augment the meaning of classes, variables and functions not possible to be conveyed through naming conventions alone.

### Comment Style
Prefer `//` code blocks over `/**/`. Standard `//` blocks are far more common

### Class Comments
Start each class with a comment block explaining the purpose and scope of the code. If a file contains code for multiple abstractions, ie. a virtual class and its implementations, a block should be provided at the top of each interface. Note: prefer splitting all abstractions and implementations into their own files, combine only for very small classes. Class comments should also provide a simple example on how the class should be used, as well as describing any necessary assumptions.

### Class Function Comments
If the use of a function is not obvious (ie. get, set, etc...) place a comment above the function describing the operation. Almost every function should have a comment immediately preceding it descibing use and providing an example. A function comment should provide the following (if relevant): 
* What the inputs and outputs are. If function argument names are provided in `backticks`, then code-indexing tools may be able to present the documentation better.
* For class member functions: whether the object remembers reference or pointer arguments beyond the duration of the method call. This is quite common for pointer/reference arguments to constructors.
* For each pointer argument, whether it is allowed to be null and what happens if it is.
* For each output or input/output argument, what happens to any state that argument is in. (E.g. is the state appended to or overwritten?).
* If there are any performance implications of how a function is used.

### Class Variable Comments
In general, a variable name should provide enough information about its purpose. In cases where this is not immediately obvious, provide comments describing its use. 
In particular, add comments to describe the existence and meaning of sentinel values, such as nullptr or -1, when they are not obvious. For example:
```cpp
private:
 // Used to bounds-check table accesses. -1 means
 // that we don't yet know how many entries the table has.
 int num_total_entries_;
 ```

 ### Global Variables
 All global variables should have a comment describing their use and reason for being global

 ## General Formatting
 A .clang-format file is provided which manages formatting automatically. In visual studio, periodically use hotkeys `Cntrl-k, Cntrl-d` to perform an auto clang format. 
