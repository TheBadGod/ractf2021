# Lego Car Generator

A very basic LCG with a seed of 32 bits, so just
try all of the possible seeds and print all the strings
which only readable ascii chars (Spoiler: There arent' many)

```cpp
#include <iostream>
#include <fstream>
#include <istream>
#include <cstring>

char buf[100];

int main() {
    std::ifstream file { "secret" };
    file >> buf;
    int len = strlen(buf);
    std::cout << len << std::endl;
    char out[100];
    out[len] = 0;
    for(unsigned int i = 0; i < 0xFFFFFFFF; i++) {
        bool corr = true;
        if(i%100000000 == 0) std::cout << "progress: " << i << std::endl;
        unsigned int s = i;
        
        int j = 0;
        while(j < len && corr) {
            s = s * 0x17433a5b + 0xb7e184a3;
            unsigned int x = s;
            for(int k = 0; k < 4 && j < len; k++) {
                out[j] = buf[j] ^ ((x>>24)&0xFF);
                if(out[j] < 0x0) {
                    corr = false;
                    break;
                }
                j++;
                x = (x >> 24) | (x << 8);
            }
        }

        if(corr) std::cout << out << std::endl;
    }
}
```

Compiled with O3 this gives us the flag (and a bit more) in a few seconds: 
`ractf{CL04K_3NGa6ed}sikeyouthoughttheflagwasthislong`
