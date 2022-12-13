# C++

- reverse compatible
- flexible but chaotic syntax
  - you can program in c way, c++ way and even assembly way
- not beginner friendly



# L1 Introduction

## Hello World!

- main function: things in `.cpp` that would be executed by default

- `#include <iostream>`: 

  - import a library, yet `<lib-name>` only works for standard library
  - another way: `#include "iostream.h"`   a more common way

- `using namespace std`: another way for import library

- `<<`: stream operators

- `return 0`: main function must return int, so we often return 0;

  

  ```c++
  #include <iostream>
  
  using namespace std;
  
  int main() {
      string str = "Hello World!";
      cout << str << endl;
      return 0;
  }
  
  ```

  



# L2 Stream

- string in c++ is essentially char array

  ```c++
  #include <iostream>
  
  using namespace std;
  
  int main() {
      string str = "Hello World!";
      cout << str[0] << endl;
      str[0] = 'h';        // 'h'是一个char, 注意使用'', 而不是""
      cout << str << endl;
      return 0;
  }
  
  ```

  

## StringStream

- a character buffer that automatically interacts with the external source
- convert variables to a string form that can be written in the buffer
- can obstain string from the buffer



- provide a unified interface to access console



### outputstream

```c++
#include <iostream>
#include <sstream>

using namespace std;

int main() {
    ostringstream oss("Green Tea");		// intial string
    cout << oss.str() << endl;		// "Green Tea"
    oss << 16.9;
    cout << oss.str() << endl;		// "16.9n Tea"

    
    ostringstream oss2("Green Tea", ostringstream::ate);    // mode: ate: at the end
    cout << oss2.str() << endl;		// "Green Tea"
    oss2 << "16.9";
    cout << oss2.str() << endl;		// "Green Tea16.9"

    return 0;
}

```



### inputstream

```c++
#include <iostream>
#include <sstream>

using namespace std;

int main() {
    
    // automatic conversion 1

    istringstream iss("16.9 Ounces");
    double amount;
    string unit;
    iss >> amount;      // iss read input string, until reach whitespace, then give amount 16.9
    iss >> unit;        // read next, give unit "Ounces"
    cout << amount/2 << endl;  // 8.45
    cout << unit << endl;      // Ounces


    // automatic conversion 2
    istringstream iss1("16.9 Ounces");
    int amount1;
    string unit1;
    iss1 >> amount1;      // iss1 read input string, until reach ., then give amount1 16
    iss1 >> unit1;        // read next, give unit ".9"
    cout << amount1/2 << endl;  // 8
    cout << unit1 << endl;      // .9
    
    // iss1 >> amount1 >> unit1
    // this is the same as line 23 ~ 24
    // `iss1 >> amount1` actually returns iss1

    return 0;
}

```



- How inputstream define separating characters? Type matters!

<img src="C:\Users\Jon\AppData\Roaming\Typora\typora-user-images\image-20221212020833261.png" alt="image-20221212020833261" style="zoom: 50%;" />



- if the `iss` was `>>` to a incompatible type, it will do nothing. like:

  ```c++
  #include <iostream>
  #include <sstream>
  
  using namespace std;
  
  int stringToInteger(const string& s) {
      istringstream iss(s);
      int out;          // out is initialized to be 0
      iss >> out;    // "a" can't be cast to int, so do nothing
      return out;    // out is still 0
  }
  
  int main() {
      cout << stringToInteger("a") << endl;	  // result: 0
      return 0;
  }
  
  
  
  ```

  

- positioning functions

  <img src="C:\Users\Jon\AppData\Roaming\Typora\typora-user-images\image-20221212021531445.png" alt="image-20221212021531445" style="zoom:50%;" />



### state bit

- there are 4 bits indicating the state of the stream

  <img src="C:\Users\Jon\AppData\Roaming\Typora\typora-user-images\image-20221212022429128.png" alt="image-20221212022429128" style="zoom:50%;" />

  

  

- bad state

  ```c++
  #include <iostream>
  #include <sstream>
  
  using namespace std;
  
  int stringToInteger(const string& s) {
      istringstream iss(s);
      int out;
      string random;		// initialized to be ""
      iss >> out;            // bad state
      iss >> random;     // do noting
      cout << random << endl;   //  cout ""
      return out;
  }
  
  int main() {
      cout << stringToInteger("th.i") << endl;     // print "" + \n + 0
      return 0;
  }
  ```



- use state bit for checking

  ```c++
  #include <iostream>
  #include <sstream>
  
  using namespace std;
  
  int stringToInteger(const string& s) {
      istringstream iss(s);
      int out;
      string random="initial";
      iss >> out;
      if (iss.fail()) throw domain_error("no value int at beginning!");
      if (!iss.eof()) throw domain_error("more than a single character");
      return out;
  }
  
  void printStateBit() {
  
  }
  
  
  int main() {
      cout << stringToInteger("1") << endl;
      return 0;
  }
  
  ```

  

- for operation `iss >> var`, if it successed, it can be regarded as `true`, otherwise it can be regarded as `false`



## IO Stream

- similar to stringstream, but there is an additional external source: the console
- 4 standard iostreams:
  - cin
  - `cout`: buffered
  - `cerr`: error stream, unbuffered
  - `clog`: error stream, buffered

- use `cout << flush` to push characters in the buffer to console
  - `endl`: `flush ` + `\n`



### cin

- Why `cin` is a nightmare
  - behaviors:
    - when EOF happens in cin, it will pause the whole program and wait for users to input another string
    - upon user press `Enter` and input the string, cin will read the string, and stop at the position of (or before) `\n` (input by `Enter`)
    - For next input, cin will automatically skip the whitespace (`\n`) and read next string
  - `cin` read in a full line, but only extract the first whitespace-separated token
  - therefore, trash (inapporpriate whitespace) in the buffer will make cin not pause the program and prompt the user for input
  - and all future `cin` fails too
  - How to solve it: **`getline`**
    - `cin >> variable` treats whitespace as separator
    - `getline(cin, variable)` only treats `\n` as separator, and it also consumes the `\n` after reading the line



## Manipulators

- special keywords that change the behavior of the stream

<img src="C:\Users\Jon\AppData\Roaming\Typora\typora-user-images\image-20221212030441164.png" alt="image-20221212030441164" style="zoom:50%;" />
