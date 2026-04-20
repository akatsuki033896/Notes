```sh
apt update
apt install -y libtinyxml2-dev
g++ test.cpp -ltinyxml2
```

```cpp
#include <tinyxml2.h>
```

```cmake
find_package(tinyxml2 REQUIRED)
target_link_libraries(your_target tinyxml2)
```