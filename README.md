# ZPacker is a lightweight data serialization & deserialization toolkit
It designed for scenarios that data must be persistent but modern data interchange types are not present

It is lightweight, no third-party dependencies and header only, can easily to integrate with existing projects that written in C++

It served for my private project "Zeus DLP System" for almost a year, it now comes public for guys who needs it

## Application Scenario
- It can be used for data exchange in some inter-process communication such as PIPE and ALPC
- It can be used for windows driver to load & save some configuration data in the registry or on the disk
- Used in some embeded systems

## Features
- no reflection
- easy to integrate with other system software
- support crc8/16/32 checksums(optional)
- support to pack the serialized data into custom data format and unpack it smoothly
- support all c++ STL sequence and association containers, std::tuple, std::variant, std::array(serialization only), std::forward_list(serialization only) and customized types

## Examples
All the examples are placed in example.cpp, here we demonstrate some basic usages:

- serialize a basic STL container
```C++
    std::list<int> list1 = {1, 2, 3, 4};

    std::vector<uint8_t> buffer;

    // initialize a bytes writer
    zeus::bytes_writer writer{buffer};

    // serialize `list1` to `buffer`
    writer << list1;

    // initialize a bytes reader
    zeus::bytes_reader reader{buffer};

    // deserialize a `std::vector<int>` from `buffer`
    // since `std::list` and `std::vector` are all sequence container, so the data type is compatible
    // the serializer will do necessary runtime checks for us, even we can deserialize out a `std::vector<uint8_t>` from `buffer` with no runtime error
    // BUT we CANNOT deserialize a `std::vector<uint64_t>`, since it will cause undefined error
    // the serializer have done all error-checking jobs for us
    auto object = reader.read<std::vector<int>>();

    // print the contents in `object`
    std::for_each(object.begin(), object.end(), [](const auto &v)
                  { printf("value = %d\n", v); });
```
- serialize and pack the output data
```C++
    std::unordered_map<std::string, uint32_t> map1{{"Jacky", 68}, {"Element", 97}, {"Bob", 45}};

    // call the `serialize` function to do the serialization and packing job
    auto data1 = zeus::serialize(map1);

    // call the `deserialize` function to unpack the binary data and deserialize into a new `object` of type `std::map<std::string, uint32_t>`
    auto object = zeus::deserialize<std::map<std::string, uint32_t>>(data1);

    // print the contents in `object`
    std::for_each(object.begin(), object.end(), [](const auto &v)
                  { printf("name: %s, score: %d\n", v.first.c_str(), v.second); });
```
- serialize custom type instance, user must implement `serialize` or `deserialize` method
```C++
    // custom type that is not trivially copyable
    // for any custom types that is not trivially copyable, it must implement `serialize` or `deserialize` method
    struct CustomType
    {
        uint32_t id{};
        std::string name{};
        uint32_t salary{};
        std::list<std::string> friends{};

        CustomType() : id(0), name("jacky"), salary(3267), friends{ "Bob", "Element" } {}

        // user defined `serialize` method for static binding
        // `_REQUIRE_WRITER` is a macro defined in zpacker.hpp, used by SFINAE
        template <class _Writer, _REQUIRE_WRITER(_Writer, CustomType)>
        void serialize(_Writer &writer) const
        {
            writer << id << name << friends;
        }

        // user defined `deserialize` method for static binding
        // `_REQUIRE_READER` is a macro defined in zpacker.hpp, used by SFINAE
        template <class _Reader, _REQUIRE_READER(_Reader, CustomType)>
        static CustomType deserialize(_Reader &reader)
        {
            CustomType self{};

            reader >> self.id >> self.name >> self.friends;

            return self;
        }
    };

    CustomType custom{};

    auto data = zeus::serialize(custom);

    auto object = zeus::deserialize<decltype(custom)>(data);

    // verify....
    // ...
```

# License
This is licensed under the MIT License