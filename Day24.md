# Day 24 of Advent of Code 2022

Lets first try to solve the example problem. I think I am going to use
bit operations. Lets first read the input file.

```c

#define W 6
#define H 4

unsigned char map[H][W];
#define BN 1
#define BE 2
#define BS 4
#define BW 8

#define INPUT_FN "day24.txt"

void read_map()
{
    FILE *f = fopen(INPUT_FN, "r");
    if (f == 0)
        return;
    
    for (int i = 0; i < H; i++)
        for (int j = 0; j < W; j++)
            map[i][j] = 0;
            
    char buffer[W+5];
    fgets(buffer, W+5, f);
    for (int i = 0; i < H; i++)
    {
        fgets(buffer, W+5, f);
        for (int j = 0; j < W; j++)
            if (buffer[j+1] == '^') map[i][j] = BN; else
            if (buffer[j+1] == '>') map[i][j] = BE; else
            if (buffer[j+1] == 'v') map[i][j] = BS; else
            if (buffer[j+1] == '<') map[i][j] = BW;
    }
    fclose(f);
}

//                  ^ 0101010101010101
//                  > 0011001100110011
//                  v 0000111100001111
//                  < 0000000011111111
const char *decode = ".^>2v223<2232334";

void print_map()
{
    for (int i = 0; i < W+2; i++)
        printf("#");
    printf("\n");
    for (int i = 0; i < H; i++)
    {
        printf("#");
        for (int j = 0; j < W; j++)
            printf("%c", decode[map[i][j]]);
        printf("#\n");
    }
    for (int i = 0; i < W+2; i++)
        printf("#");
    printf("\n\n");
}


int main(int argc, char *argv[])
{
    read_map();
    print_map();
}
```

Now lets implement the function for one step:
```c
void one_step()
{
    for (int i = 0; i < H; i++)
        for (int j = 0; j < W; j++)
            map[i][j] |= (  (map[(i+H-1)%H][j        ] & BS)
                          | (map[i        ][(j+1)%W  ] & BW)
                          | (map[(i+1)%H  ][j        ] & BN)
                          | (map[i        ][(j+W-1)%W] & BE)
                         ) << 4;
    for (int i = 0; i < H; i++)
        for (int j = 0; j < W; j++)
            map[i][j] = map[i][j] >> 4;
}
                     
int main(int argc, char *argv[])
{
    ...
    for (int s = 0; s < 18; s++)
    {
        one_step();
        print_map();
    }
}
```
Okay, that looks like it is working.

The idea it to use a Boolean array to mark all locations
that can be reached from the start at each step.

```c
bool cur[H][W];

void one_step()
{
    ...
    bool next[H][W];
    for (int i = 0; i < H; i++)
        for (int j = 0; j < W; j++)
            next[i][j] =    map[i][j] == 0
                         && (   cur[i][j]
                             || (i == 0 && j == 0)
                             || (i > 0   && cur[i-1][j])
                             || (i < H-1 && cur[i+1][j])
                             || (j > 0   && cur[i][j-1])
                             || (j < W-1 && cur[i][j+1]));
    for (int i = 0; i < H; i++)
        for (int j = 0; j < W; j++)
            cur[i][j] = next[i][j];
}

void print_map()
{
    for (int i = 0; i < W+2; i++)
        printf("#");
    printf("\n");
    for (int i = 0; i < H; i++)
    {
        printf("#");
        for (int j = 0; j < W; j++)
            if (cur[i][j])
                printf("E");
            else
                printf("%c", decode[map[i][j]]);
        printf("#\n");
    }
    for (int i = 0; i < W+2; i++)
        printf("#");
    printf("\n\n");
}

int main(int argc, char *argv[])
{
    read_map();
    print_map();
    for (int i = 0; i < H; i++)
        for (int j = 0; j < W; j++)
            cur[i][j] = 0;
    for (int s = 0; s < 18; s++)
    {
        one_step();
        print_map();
    }
}
```

That looks good. Now lets find the count:
```c
int main(int argc, char *argv[])
{
    read_map();
    print_map();
    for (int i = 0; i < H; i++)
        for (int j = 0; j < W; j++)
            cur[i][j] = 0;
    int count = 1;
    while (!cur[H-1][W-1])
    {
        one_step();
        print_map();
        count++;
    }
    printf("%d\n", count);
}
```

Let see what it does on my puzzle input;
```c
#define H 25
#define W 120
#define INPUT_FN "input/day24.txt"
```

That did return the correct answer.

### Second part

This does not look so much complicated as the first part.
I gave it a first shot, but that did not work out. I have
to fix somethings.

```c

int enter_at_i;
int enter_at_j;

int minutes = 0;

void next_minute()
{
    minutes++;
    for (int i = 0; i < H; i++)
        for (int j = 0; j < W; j++)
            map[i][j] |= (  (map[(i+H-1)%H][j        ] & BS)
                          | (map[i        ][(j+1)%W  ] & BW)
                          | (map[(i+1)%H  ][j        ] & BN)
                          | (map[i        ][(j+W-1)%W] & BE)
                         ) << 4;
    for (int i = 0; i < H; i++)
        for (int j = 0; j < W; j++)
            map[i][j] = map[i][j] >> 4;
}

void one_step()
{
    bool next[H][W];
    for (int i = 0; i < H; i++)
        for (int j = 0; j < W; j++)
            next[i][j] =    map[i][j] == 0
                         && (   cur[i][j]
                              || (i == enter_at_i && j == enter_at_j)
                             || (i > 0   && cur[i-1][j])
                             || (i < H-1 && cur[i+1][j])
                             || (j > 0   && cur[i][j-1])
                             || (j < W-1 && cur[i][j+1]));
    for (int i = 0; i < H; i++)
        for (int j = 0; j < W; j++)
            cur[i][j] = next[i][j];
}

void walk(int fi, int fj, int ti, int tj)
{
    for (int i = 0; i < H; i++)
        for (int j = 0; j < W; j++)
            cur[i][j] = FALSE;
    enter_at_i = fi;
    enter_at_j = fj;
    while (!cur[ti][tj])
    {
        next_minute();
        one_step();
        //print_map();
    }
    next_minute();
    print_map();
}    

int main(int argc, char *argv[])
{
    read_map();
    walk(0, 0, H-1, W-1);
    printf("%d\n", minutes);
    //minutes = 0;
    walk(H-1, W-1, 0, 0);
    printf("%d\n", minutes);
    //minutes = 0;
    walk(0, 0, H-1, W-1);
    printf("%d\n", minutes);
}

```


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
../IParse/software/MarkDownC Day24.md >p.c; gcc -Wall -g p.c -o p && ./p
```
