# Day 18 of Advent of Code 2022

Todays puzzle does not look so hard. for each cube just add 6 for
visible sides and substract 2 for each combination of sides that
are touching. The first question is whether we are going to store
the all the cubes with their coordinates or mark their location in
a Boolean area. Lets first read all the data and find the maximum
value. (I started just before 9 in the morning.)
```c
void read_input()
{
    FILE *f = fopen("input/day18.txt", "r");
    if (f == 0)
        return;
    char buffer[40];
    while (fgets(buffer, 40, f))
    {
        const char *s = buffer;
        int x = read_int(&s);
        s++;
        int y = read_int(&s);
        s++;
        int z = read_int(&s);
        process(x, y, z);
    }
    fclose(f);
}

void process(int x, int y, int z)
{
    printf("%d,%d,%d\n", x, y, z);
    find_range(x);
    find_range(y);
    find_range(z);
}

int min = 1000;
int max = -1000;
void find_range(int v)
{
    if (v < min) min = v;
    if (v > max) max = v;
}

int main(int argc, char *argv[])
{
    read_input();
    printf("%d - %d\n", min, max);
}
```

So the range is from 0 to 21 (including). Lets try with a Boolean
array in three dimesions:
```c
#define SIZE 22
bool samples[SIZE][SIZE][SIZE];

void init_sampes()
{
    for (int x = 0; x < SIZE; x++)
        for (int y = 0; y < SIZE; y++)
            for (int z = 0; z < SIZE; z++)
                samples[x][y][z] = FALSE;
}

num_t open_sides = 0;
void process(int x, int y, int z)
{
    if (samples[x][y][z])
        printf("more than one at %d, %d, %d\n", x, y, z);
    samples[x][y][z] = TRUE;
    open_sides += 6;
}

void count_touching()
{
    for (int x = 0; x < SIZE-1; x++)
        for (int y = 0; y < SIZE; y++)
            for (int z = 0; z < SIZE; z++)
                if (samples[x][y][z] && samples[x+1][y][z])
                    open_sides -= 2;
    for (int x = 0; x < SIZE; x++)
        for (int y = 0; y < SIZE-1; y++)
            for (int z = 0; z < SIZE; z++)
                if (samples[x][y][z] && samples[x][y+1][z])
                    open_sides -= 2;
    for (int x = 0; x < SIZE; x++)
        for (int y = 0; y < SIZE; y++)
            for (int z = 0; z < SIZE-1; z++)
                if (samples[x][y][z] && samples[x][y][z+1])
                    open_sides -= 2;
}

int main(int argc, char *argv[])
{
    init_sampes();
    read_input();
    count_touching();
    printf("%lld\n", open_sides);
}
```

That found me the solution for the first part.

### Second part

The second part is kind of what I had expected. You can simply
solve this with a trickle algorithm. Lets add a array to represent
what locations can be reached with the water:
```c
bool water[SIZE][SIZE][SIZE];

void answer2()
{
    for (int x = 0; x < SIZE; x++)
        for (int y = 0; y < SIZE; y++)
            for (int z = 0; z < SIZE; z++)
                water[x][y][z] = FALSE;
    for (int go = 1; go--; )
    {
        // Some debugging code:
        int nr = 0;
        for (int x = 0; x < SIZE; x++)
            for (int y = 0; y < SIZE; y++)
                for (int z = 0; z < SIZE; z++)
                    if (water[x][y][z])
                        nr++;
        printf("Pass %d\n", nr);
        for (int x = 0; x < SIZE; x++)
            for (int y = 0; y < SIZE; y++)
                for (int z = 0; z < SIZE; z++)
                    if (!water[x][y][z] && !samples[x][y][z]
                        && (   x == 0 || water[x-1][y][z] || x == SIZE-1 || water[x+1][y][z]
                            || y == 0 || water[x][y-1][z] || y == SIZE-1 || water[x][y+1][z]
                            || z == 0 || water[x][y][z-1] || z == SIZE-1 || water[x][y][z+1]))
                    {
                        water[x][y][z] = TRUE;
                        go =  1;
                    }
    }
    
    int nr_touching = 0;
    for (int x = 0; x < SIZE; x++)
        for (int y = 0; y < SIZE; y++)
            for (int z = 0; z < SIZE; z++)
                if (samples[x][y][z])
                {
                    if (x == 0      || water[x-1][y][z]) nr_touching++;
                    if (x == SIZE-1 || water[x+1][y][z]) nr_touching++;
                    if (y == 0      || water[x][y-1][z]) nr_touching++;
                    if (y == SIZE-1 || water[x][y+1][z]) nr_touching++;
                    if (z == 0      || water[x][y][z-1]) nr_touching++;
                    if (z == SIZE-1 || water[x][y][z+1]) nr_touching++;
                }
    printf("%d\n", nr_touching);
}

int main(int argc, char *argv[])
{
    ...
    answer2();
}
```
After some debugging and some modification, I found the correct answer
with the above code.

### Some standard definitions

```c
#include "stdlib.h"
typedef int bool;
typedef int long long num_t;
#define FALSE 0
#define TRUE 1

int read_int(const char **ref_s)
{
    int v = 0;
    int s = 1;
    if (**ref_s == '-')
    {
        s = -1;
        (*ref_s)++;
    } 
    for (; '0' <= **ref_s && **ref_s <= '9'; (*ref_s)++)
        v = 10 * v + **ref_s - '0';
    return s * v;
}
```


The command I use to process this markdown file, is:
```
./IParse/software/MarkDownC Day18.md >p.c; gcc -Wall -g p.c -o p && ./p
```
