# JSONCONS

jsoncons is a C++, header-only library for constructing [JSON](http://www.json.org) and JSON-like
data formats such as [CBOR](http://cbor.io/). It supports 

- Parsing JSON-like text or binary data into an unpacked representation of variant type
  that defines an interface for accessing and modifying that data.

- Serializing the unpacked representation into different JSON-like text or binary data.

- Converting from JSON-like text or binary data to C++ objects and back via [json_type_traits](https://github.com/danielaparker/jsoncons/blob/master/doc/ref/json_type_traits.md).

- Streaming JSON read and write events, somewhat analogously to SAX (push parsing) and StAX (pull parsing) in the XML world. 

Compared to other JSON libraries, jsoncons has been designed to handle very large JSON texts. At its heart are
SAX style parsers and serializers. Its [json parser](https://github.com/danielaparker/jsoncons/blob/master/doc/ref/json_parser.md) is an 
incremental parser that can be fed its input in chunks, and does not require an entire file to be loaded in memory at one time. 
Its unpacked in-memory representation of JSON is more compact than most, and can be made more compact still with a user-supplied
allocator. It also supports memory efficient parsing of very large JSON texts with a [pull parser](https://github.com/danielaparker/jsoncons/blob/master/doc/ref/json_stream_reader.md),
built on top of its incremental parser.  

The jsoncons [data model](doc/ref/data-model.md) extends the familiar JSON types - nulls,
booleans, numbers, strings, arrays, objects - to accomodate byte 
strings, date-time values, epoch time values, big numbers, and 
decimal fractions. This allows it to preserve these type semantics 
when parsing JSON-like data formats such as CBOR that support them. 

Planned new features are listed on the [roadmap](doc/Roadmap.md)

jsoncons is distributed under the [Boost Software License](http://www.boost.org/users/license.html).

## Supported compilers

jsoncons uses some features that are new to C++ 11, including [move semantics](http://thbecker.net/articles/rvalue_references/section_02.html) and the [AllocatorAwareContainer](http://en.cppreference.com/w/cpp/concept/AllocatorAwareContainer) concept. It is tested in continuous integration on [AppVeyor](https://ci.appveyor.com/project/danielaparker/jsoncons) and [Travis](https://travis-ci.org/danielaparker/jsoncons).

| Compiler                | Version          |Architecture | Operating System | Notes |
|-------------------------|------------------|-------------|------------------|-------|
| Microsoft Visual Studio | vs2015 and above | x86,x64     | Windows 10       |       |
| g++                     | 4.8 and above    | x64         | Ubuntu           |`std::regex` isn't fully implemented in 4.8, so `jsoncons::jsonpath` regular expression filters aren't supported in 4.8 |
| clang                   | 3.8 and above    | x64         | Ubuntu           |       |
| clang xcode             | 6.4 and above    | x64         | OSX              |       |

It is also cross compiled for ARMv8-A architecture on Travis using clang and executed using the emulator qemu. 

[UndefinedBehaviorSanitizer (UBSan)](http://clang.llvm.org/docs/UndefinedBehaviorSanitizer.html) is enabled for selected gcc and clang builds.

## Get jsoncons

Download the [latest release](https://github.com/danielaparker/jsoncons/releases) and unpack the zip file. Copy the directory `include/jsoncons` to your `include` directory. If you wish to use extensions, copy `include/jsoncons_ext` as well. 

Or, download the latest code on [master](https://github.com/danielaparker/jsoncons/archive/master.zip).

## How to use it

- [Quick guide](http://danielaparker.github.io/jsoncons)
- [Examples](doc/Examples.md)
- [Reference](doc/Home.md)

As the `jsoncons` library has evolved, names have sometimes changed. To ease transition, jsoncons deprecates the old names but continues to support many of them. See the [deprecated list](doc/ref/deprecated.md) for the status of old names. The deprecated names can be suppressed by defining macro `JSONCONS_NO_DEPRECATED`, which is recommended for new code.

## Benchmarks

[json_benchmarks](https://github.com/danielaparker/json_benchmarks) provides some measurements about how `jsoncons` compares to other `json` libraries.

- [JSONTestSuite and JSON_checker test suites](https://danielaparker.github.io/json_benchmarks/) 

- [Performance benchmarks with text and integers](https://github.com/danielaparker/json_benchmarks/blob/master/report/performance.md)

- [Performance benchmarks with text and doubles](https://github.com/danielaparker/json_benchmarks/blob/master/report/performance_fp.md)

## Extensions

- [jsonpointer](doc/ref/jsonpointer/jsonpointer.md) implements the IETF standard [JavaScript Object Notation (JSON) Pointer](https://tools.ietf.org/html/rfc6901)
- [jsonpatch](doc/ref/jsonpatch/jsonpatch.md) implements the IETF standard [JavaScript Object Notation (JSON) Patch](https://tools.ietf.org/html/rfc6902)
- [jsonpath](doc/ref/jsonpath/jsonpath.md) implements [Stefan Goessner's JSONPath](http://goessner.net/articles/JsonPath/).  It also supports search and replace using JSONPath expressions.
- [cbor](doc/ref/cbor/cbor.md) implements decode from and encode to the IETF standard [Concise Binary Object Representation (CBOR)](http://cbor.io/). It also supports a set of operations for iterating over and accessing the nested data items of a packed CBOR value.
- [msgpack](doc/ref/msgpack/msgpack.md) implements decode from and encode to the [MessagePack](http://msgpack.org/index.html) binary serialization format.
- [csv](doc/ref/csv/csv.md) implements reading (writing) JSON values from (to) CSV files

### A simple example

```c++
#include <iostream>
#include <fstream>
#include <jsoncons/json.hpp>

// For convenience
using jsoncons::json;

int main()
{
    json color_spaces = json::array();
    color_spaces.push_back("sRGB");
    color_spaces.push_back("AdobeRGB");
    color_spaces.push_back("ProPhoto RGB");

    json image_sizing; // empty object
    image_sizing["Resize To Fit"] = true; // a boolean 
    image_sizing["Resize Unit"] = "pixels"; // a string
    image_sizing["Resize What"] = "long_edge"; // a string
    image_sizing["Dimension 1"] = 9.84; // a double
    
    json export_settings;

    // create "File Format Options" as an object and put "Color Spaces" in it
    export_settings["File Format Options"]["Color Spaces"] = std::move(color_spaces); 

    export_settings["Image Sizing"] = std::move(image_sizing);

    // Write to stream
    std::ofstream os("export_settings.json");
    os << export_settings;

    // Read from stream
    std::ifstream is("export_settings.json");
    json j = json::parse(is);

    // Pretty print
    std::cout << "(1)\n" << pretty_print(j) << "\n\n";

    // Does object member exist?
    std::cout << "(2) " << std::boolalpha << j.contains("Image Sizing") << "\n\n";

    // Get reference to object member
    const json& val = j["Image Sizing"];

    // Access member as double
    std::cout << "(3) " << "Dimension 1 = " << val["Dimension 1"].as<double>() << "\n\n";

    // Try access member with default
    std::cout << "(4) " << "Dimension 2 = " << val.get_with_default("Dimension 2",0.0) << "\n";
}
```
Output:
```json
(1)
{
    "File Format Options": {
        "Color Spaces": ["sRGB","AdobeRGB","ProPhoto RGB"],
        "Image Formats": ["JPEG","PSD","TIFF","DNG"]
    },
    "File Settings": {
        "Color Space": "sRGB",
        "Image Format": "JPEG",
        "Limit File Size": true,
        "Limit File Size To": 10000
    },
    "Image Sizing": {
        "Dimension 1": 9.84,
        "Resize To Fit": true,
        "Resize Unit": "pixels",
        "Resize What": "long_edge"
    }
}

(2) true

(3) Dimension 1 = 9.8

(4) Dimension 2 = 0
```

## About jsoncons::basic_json

The jsoncons library provides a `basic_json` class template, which is the generalization of a `json` value for different 
character types, different policies for ordering name-value pairs, etc. A `basic_json` provides an unpacked representation 
of JSON-like string or binary data formats, and defines an interface for accessing and modifying that data.

```c++
typedef basic_json<char,
                   ImplementationPolicy = sorted_policy,
                   Allocator = std::allocator<char>> json;
```
The library includes four instantiations of `basic_json`:

- [json](doc/ref/json.md) constructs a utf8 character json value that sorts name-value members alphabetically

- [ojson](doc/ref/ojson.md) constructs a utf8 character json value that preserves the original name-value insertion order

- [wjson](doc/ref/wjson.md) constructs a wide character json value that sorts name-value members alphabetically

- [wojson](doc/ref/wojson.md) constructs a wide character json value that preserves the original name-value insertion order

## More examples

[Playing around with CBOR, JSON, and CSV](#E1)

[Convert JSON text to C++ objects, and back](#E2)

[Pull parser example](#E3)

[Dump json content into a larger document](#E4)

<div id="E1"/>

### Playing around with CBOR, JSON, and CSV

```c++
#include <jsoncons/json.hpp>
#include <jsoncons_ext/cbor/cbor_serializer.hpp>
#include <jsoncons_ext/cbor/cbor_view.hpp>
#include <jsoncons_ext/jsonpointer/jsonpointer.hpp>
#include <jsoncons_ext/csv/csv_serializer.hpp>

// For convenience
using namespace jsoncons;    

int main()
{
        // Construct some CBOR using the streaming API
        std::vector<uint8_t> b;
        cbor::cbor_bytes_serializer writer(b);
        writer.begin_array(); // indefinite length outer array
        writer.begin_array(3); // a fixed length array
        writer.string_value("foo");
        writer.byte_string_value({'b','a','r'});
        writer.bignum_value("-18446744073709551617");
        writer.end_array();
        writer.end_array();
        writer.flush();

        // Print bytes
        std::cout << "(1)\n";
        for (auto c : b)
        {
            std::cout << std::hex << std::setprecision(2) << std::setw(2)
                      << std::setfill('0') << static_cast<int>(c);
        }
        std::cout << "\n\n";
/*
        9f -- Start indefinte length array
          83 -- Array of length 3
            63 -- String value of length 3
              666f6f -- "foo" 
            43 -- Byte string value of length 3
              626172 -- 'b''a''r'
            c3 -- Tag 3 (negative bignum)
              49 -- Byte string value of length 9
                010000000000000000 -- Bytes content
          ff -- "break" 
*/
        cbor::cbor_view bv = b; // a non-owning view of the CBOR bytes

        // Loop over the rows
        std::cout << "(2)\n";
        for (cbor::cbor_view row : bv.array_range())
        {
            std::cout << row << "\n";
        }
        std::cout << "\n";

        // Get element at position 0/2 using jsonpointer (must be by value)
        cbor::cbor_view v = jsonpointer::get(bv, "/0/2");
        std::cout << "(3) " << v.as<std::string>() << "\n\n";

        // Print JSON representation with default options
        std::cout << "(4)\n";
        std::cout << pretty_print(bv) << "\n\n";

        // Print JSON representation with different options
        json_serializing_options options;
        options.byte_string_format(byte_string_chars_format::base64)
               .bignum_format(bignum_chars_format::base64url);
        std::cout << "(5)\n";
        std::cout << pretty_print(bv, options) << "\n\n";

        // Unpack bytes into a json variant value, and add some more elements
        json j = cbor::decode_cbor<json>(bv);

        json another_array = json::array(); 
        another_array.emplace_back(byte_string({'q','u','x'}));
        another_array.emplace_back("273.15", semantic_tag_type::decimal);
        another_array.emplace(another_array.array_range().begin(),"baz"); // place at front

        j.push_back(std::move(another_array));
        std::cout << "(6)\n";
        std::cout << pretty_print(j) << "\n\n";

        // Get element at position /1/2 using jsonpointer (can be by reference)
        json& ref = jsonpointer::get(j, "/1/2");
        std::cout << "(7) " << ref.as<std::string>() << "\n\n";

        // If code compiled with GCC and std=gnu++11 (rather than std=c++11)
        __int128 i = j[1][2].as<__int128>();

        // Repack bytes
        std::vector<uint8_t> b2;
        cbor::encode_cbor(j, b2);

        // Print the repacked bytes
        std::cout << "(8)\n";
        for (auto c : b2)
        {
            std::cout << std::hex << std::setprecision(2) << std::setw(2)
                      << std::setfill('0') << static_cast<int>(c);
        }
        std::cout << "\n\n";
/*
        82 -- Array of length 2
          83 -- Array of length 3
            63 -- String value of length 3
              666f6f -- "foo" 
            43 -- Byte string value of length 3
              626172 -- 'b''a''r'
            c3 -- Tag 3 (negative bignum)
            49 -- Byte string value of length 9
              010000000000000000 -- Bytes content
          83 -- Another array of length 3
          63 -- String value of length 3
            62617a -- "baz" 
          43 -- Byte string value of length 3
            717578 -- 'q''u''x'
          c4 - Tag 4 (decimal fraction)
            82 - Array of length 2
              21 -- -2
              19 6ab3 -- 27315
*/
        std::cout << "(9)\n";
        cbor::cbor_view bv2 = b2;
        std::cout << pretty_print(bv2) << "\n\n";

        // Serialize to CSV
        csv::csv_serializing_options csv_options;
        csv_options.column_names("Column 1,Column 2,Column 3");

        std::string csv_j;
        csv::encode_csv(j, csv_j, csv_options);
        std::cout << "(10)\n";
        std::cout << csv_j << "\n\n";

        std::string csv_bv2;
        csv::encode_csv(bv2, csv_bv2, csv_options);
        std::cout << "(11)\n";
        std::cout << csv_bv2 << "\n\n";
}

```
Output:
```
(1)
9f8363666f6f43626172c349010000000000000000ff

(2)
["foo","YmFy","-18446744073709551617"]

(3) -18446744073709551617

(4)
[
    ["foo","YmFy","-18446744073709551617"]
]

(5)
[
    ["foo","YmFy","~AQAAAAAAAAAA"]
]

(6)
[
    ["foo","YmFy","-18446744073709551617"],
    ["baz","cXV4","273.15"]
]

(7) 273.15

(8)
828363666f6f43626172c349010000000000000000836362617a43717578c48221196ab3

(9)
[
    ["foo","YmFy","-18446744073709551617"],
    ["baz","cXV4","273.15"]
]

(10)
Column 1,Column 2,Column 3
foo,YmFy,-18446744073709551617
baz,cXV4,273.15


(11)
Column 1,Column 2,Column 3
foo,YmFy,-18446744073709551617
baz,cXV4,273.15
```

### Convert `json` values to standard library types and back

```c++
std::vector<int> v{1, 2, 3, 4};
json j(v);
std::cout << "(1) "<< j << std::endl;
std::deque<int> d = j.as<std::deque<int>>();

std::map<std::string,int> m{{"one",1},{"two",2},{"three",3}};
json j(m);
std::cout << "(2) " << j << std::endl;
std::unordered_map<std::string,int> um = j.as<std::unordered_map<std::string,int>>();
```
Output:
```
(1) [1,2,3,4]

(2) {"one":1,"three":3,"two":2}
```

See [json_type_traits](doc/ref/json_type_traits.md)

### Convert `json` values to user defined types and back (also standard library containers of user defined types)

```c++
    struct book
    {
        std::string author;
        std::string title;
        double price;
    };

    namespace jsoncons
    {
        template<class Json>
        struct json_type_traits<Json, book>
        {
            // Implement static functions is, as and to_json 
        };
    }        

    book book1{"Haruki Murakami", "Kafka on the Shore", 25.17};
    book book2{"Charles Bukowski", "Women: A Novel", 12.0};

    std::vector<book> v{book1, book2};

    json j = v;

    std::list<book> l = j.as<std::list<book>>();
```

See [Type Extensibility](doc/Tutorials/Type%20Extensibility.md) for details.

<div id="E2"/>

### Convert JSON text to C++ objects, and back

The functions `decode_json` and `encode_json` convert JSON 
formatted strings to C++ objects and back. `decode_json` 
and `encode_json` will work for all C++ classes that have 
[json_type_traits](https://github.com/danielaparker/jsoncons/blob/master/doc/ref/json_type_traits.md) 
defined.

```c++
#include <iostream>
#include <map>
#include <tuple>
#include <jsoncons/json.hpp>

using namespace jsoncons;

int main()
{
    typedef std::map<std::string,std::tuple<std::string,std::string,double>> employee_collection;

    employee_collection employees = 
    { 
        {"John Smith",{"Hourly","Software Engineer",10000}},
        {"Jane Doe",{"Commission","Sales",20000}}
    };

    std::string s;
    jsoncons::encode_json(employees, s, jsoncons::indenting::indent);
    std::cout << "(1)\n" << s << std::endl;
    auto employees2 = jsoncons::decode_json<employee_collection>(s);

    std::cout << "\n(2)\n";
    for (const auto& pair : employees2)
    {
        std::cout << pair.first << ": " << std::get<1>(pair.second) << std::endl;
    }
}
```
Output:
```
(1)
{
    "Jane Doe": ["Commission","Sales",20000.0],
    "John Smith": ["Hourly","Software Engineer",10000.0]
}

(2)
Jane Doe: Sales
John Smith: Software Engineer
```

`decode_json` and `encode_json` are supported for many standard library types, and for  
[user defined types](doc/Tutorials/Type%20Extensibility.md)

See [decode_json](doc/ref/decode_json.md) and [encode_json](doc/ref/encode_json.md) 

<div id="E3"/>

### Pull parser example

A typical pull parsing application will repeatedly process the `current()` 
event and call `next()` to advance to the next event, until `done()` 
returns `true`.

The example JSON text, `book_catalog.json`, is used by the examples below.

```json
[ 
  { 
      "author" : "Haruki Murakami",
      "title" : "Hard-Boiled Wonderland and the End of the World",
      "isbn" : "0679743464",
      "publisher" : "Vintage",
      "date" : "1993-03-02",
      "price": 18.90
  },
  { 
      "author" : "Graham Greene",
      "title" : "The Comedians",
      "isbn" : "0099478374",
      "publisher" : "Vintage Classics",
      "date" : "2005-09-21",
      "price": 15.74
  }
]
```

#### Reading the JSON stream
```c++
std::ifstream is("book_catalog.json");

json_stream_reader reader(is);

for (; !reader.done(); reader.next())
{
    const auto& event = reader.current();
    switch (event.event_type())
    {
        case stream_event_type::begin_array:
            std::cout << "begin_array\n";
            break;
        case stream_event_type::end_array:
            std::cout << "end_array\n";
            break;
        case stream_event_type::begin_object:
            std::cout << "begin_object\n";
            break;
        case stream_event_type::end_object:
            std::cout << "end_object\n";
            break;
        case stream_event_type::name:
            // If underlying type is string, can return as string_view
            std::cout << "name: " << event.as<jsoncons::string_view>() << "\n";
            break;
        case stream_event_type::string_value:
            std::cout << "string_value: " << event.as<jsoncons::string_view>() << "\n";
            break;
        case stream_event_type::null_value:
            std::cout << "null_value: " << event.as<std::string>() << "\n";
            break;
        case stream_event_type::bool_value:
            std::cout << "bool_value: " << event.as<std::string>() << "\n";
            break;
        case stream_event_type::int64_value:
            std::cout << "int64_value: " << event.as<std::string>() << "\n";
            break;
        case stream_event_type::uint64_value:
            std::cout << "uint64_value: " << event.as<std::string>() << "\n";
            break;
        case stream_event_type::double_value:
            // Return as string, could also use event.as<double>()
            std::cout << "double_value: " << event.as<std::string>() << "\n";
            break;
        default:
            std::cout << "Unhandled event type\n";
            break;
    }
}
```
Output:
```
begin_array
begin_object
name: author
string_value: Haruki Murakami
name: title
string_value: Hard-Boiled Wonderland and the End of the World
name: isbn
string_value: 0679743464
name: publisher
string_value: Vintage
name: date
string_value: 1993-03-02
name: price
double_value: 18.90
end_object
begin_object
name: author
string_value: Graham Greene
name: title
string_value: The Comedians
name: isbn
string_value: 0099478374
name: publisher
string_value: Vintage Classics
name: date
string_value: 2005-09-21
name: price
double_value: 15.74
end_object
end_array
```

#### Implementing a stream_filter

```c++
// A stream filter to filter out all events except name 
// and restrict name to "author"

class author_filter : public stream_filter
{
    bool accept_next_ = false;
public:
    bool accept(const stream_event& event, const serializing_context&) override
    {
        if (event.event_type()  == stream_event_type::name &&
            event.as<jsoncons::string_view>() == "author")
        {
            accept_next_ = true;
            return false;
        }
        else if (accept_next_)
        {
            accept_next_ = false;
            return true;
        }
        else
        {
            accept_next_ = false;
            return false;
        }
    }
};
```

#### Filtering the JSON stream

```c++
std::ifstream is("book_catalog.json");

author_filter filter;
json_stream_reader reader(is, filter);

for (; !reader.done(); reader.next())
{
    const auto& event = reader.current();
    switch (event.event_type())
    {
        case stream_event_type::string_value:
            std::cout << event.as<jsoncons::string_view>() << "\n";
            break;
    }
}
```
Output:
```
Haruki Murakami
Graham Greene
```

See [json_stream_reader](doc/ref/json_stream_reader.md) 

<div id="E4"/>

### Dump json content into a larger document

```c++
#include <jsoncons/json.hpp>

using namespace jsoncons;

int main()
{
    const json some_books = json::parse(R"(
    [
        {
            "title" : "Kafka on the Shore",
            "author" : "Haruki Murakami",
            "price" : 25.17
        },
        {
            "title" : "Women: A Novel",
            "author" : "Charles Bukowski",
            "price" : 12.00
        }
    ]
    )");

    const json more_books = json::parse(R"(
    [
        {
            "title" : "A Wild Sheep Chase: A Novel",
            "author" : "Haruki Murakami",
            "price" : 9.01
        },
        {
            "title" : "Cutter's Way",
            "author" : "Ivan Passer",
            "price" : 8.00
        }
    ]
    )");

    json_serializer serializer(std::cout, jsoncons::indenting::indent); // pretty print
    serializer.begin_array();
    for (const auto& book : some_books.array_range())
    {
        book.dump(serializer);
    }
    for (const auto& book : more_books.array_range())
    {
        book.dump(serializer);
    }
    serializer.end_array();
    serializer.flush();
}
```
Output:
```json
[
    {
        "author": "Haruki Murakami",
        "price": 25.17,
        "title": "Kafka on the Shore"
    },
    {
        "author": "Charles Bukowski",
        "price": 12.0,
        "title": "Women: A Novel"
    },
    {
        "author": "Haruki Murakami",
        "price": 9.01,
        "title": "A Wild Sheep Chase: A Novel"
    },
    {
        "author": "Ivan Passer",
        "price": 8.0,
        "title": "Cutter's Way"
    }
]
```

## Building the test suite and examples with CMake

[CMake](https://cmake.org/) is a cross-platform build tool that generates makefiles and solutions for the compiler environment of your choice. On Windows you can download a [Windows Installer package](https://cmake.org/download/). On Linux it is usually available as a package, e.g., on Ubuntu,
```
sudo apt-get install cmake
```
Once cmake is installed, you can build the tests:
```
mkdir build
cd build
cmake ../ -DBUILD_TESTS=ON
cmake --build . --target test_jsoncons --config Release
```
Run from the jsoncons tests directory:

On Windows:
```
..\build\tests\Release\test_jsoncons
```

On UNIX:
```
../build/tests/Release/test_jsoncons
```

## Acknowledgements

Special debt owed to the excellent MIT licensed [tinycbor](https://github.com/intel/tinycbor), which this library draws on for platform dependent binary configuration.

A _big_ thanks to Milo Yip, author of [RapidJSON](http://rapidjson.org/), for raising the quality of JSON libraries across the board, by publishing [the benchmarks](https://github.com/miloyip/nativejson-benchmark), and contacting this project (among others) to share the results.

Special thanks to our [contributors](https://github.com/danielaparker/jsoncons/blob/master/acknowledgements.txt)

