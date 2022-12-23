# Day 22 of Advent of Code 2022

I started around 20:30. Lets first write a function to read the example map:

```c
#define W 16
#define H 12

char map[H][W];

void read_map(FILE *f)
{
    for (int i = 0; i < H; i++)
    {
        char buffer[W+5];
        fgets(buffer, W+5, f);
        for (int j = 0; j < W; j++)
            map[i][j] = buffer[j];
    }
}

void print_map()
{
    for (int i = 0; i < H; i++)
    {
        for (int j = 0; j < W; j++)
            printf("%c", map[i][j]);
        printf("\n");
    }
    printf("\n");
}

#define INPUT_FN "day22.txt"

int main(int argc, char *argv[])
{
    FILE *f = fopen(INPUT_FN, "r");
    if (f == 0)
        return 0;
    
    read_map(f);
    fclose(f);
    print_map();    
}
```

It looks like the lines in the input are just as long as they do
need to be and are not padded with spaces. Let fix this:
```c
void read_map(FILE *f)
{
    for (int i = 0; i < H; i++)
    {
        char buffer[W+5];
        fgets(buffer, W+5, f);
        const char *s = buffer;
        for (int j = 0; j < W; j++)
            if (*s < ' ')
                map[i][j] = ' ';
            else
                map[i][j] = *s++;
    }
}
```

Lets see if we can implement the walk. For the direction, left
use the encoding that also needs to be used for the answer.
```c
void process_walk(const char *w)
{
    int dir = 0;
    int i = 0;
    int j = 0;
    while (map[i][j] == ' ')
        j++;
    for (;;)
    {
        int steps = read_int(&w);
        for (int s = 0; s < steps; s++)
        {
            map[i][j] = ">v<^"[dir];
            int dir_i = dir == 1 ? 1 : dir == 3 ? H-1 : 0;
            int dir_j = dir == 0 ? 1 : dir == 2 ? W-1 : 0;
            int next_i = i;
            int next_j = j;
            for (;;)
            {
                next_i = (next_i + dir_i) % H;
                next_j = (next_j + dir_j) % W;
                if (map[next_i][next_j] != ' ')
                    break;
            }
            if (map[next_i][next_j] == '#')
                break;
            i = next_i;
            j = next_j;
        }
        //printf("%d %d\n", steps, dir);
        //print_map();
        if (*w == 'R')
            dir = (dir + 1) % 4;
        else if (*w == 'L')
            dir = (dir + 3) % 4;
        else
            break;
        w++;
    }
    printf("%d %d %d : %d\n", i+1, j+1, dir, 1000 * (i+1) + 4 * (j+1) + dir); 
}

int main(int argc, char *argv[])
{
    FILE *f = fopen(INPUT_FN, "r");
    if (f == 0)
        return 0;
    
    read_map(f);
    char walk[6000];
    fgets(walk, 6000, f);
    fgets(walk, 6000, f);
    fclose(f);
    process_walk(walk);
}
```
After some small fixes, I managed it to produce the same output as the example
output. I removed the debug statements and introduced the define 'INPUT_FN' in
the code above. Now lets make some adaptations for the puzzle input:
```c
#define W 150
#define H 200
#define INPUT_FN "input/day22.txt"
```

That did find the right answer.

### Second part

Okay, that is interesting my, map looks like:
```
 12
 3
45
6
```

I think, I am just going to adapt the code for my map, instead of writing some code
to generate the mapping. (That is an interesting puzzle as well.) I did take a
piece of paper and cut it according to my map, to see which side match by writing
some letters on it. Then I folded out, and figured out that 'boxes' had to be mapped
on which box for each direction. I first started writing out the separated case,
but then decided to use a function (`side`) for all the case, being affraid of
making a mistake with substracting and adding numbers.

```c
void process_walk(const char *w)
{
    int dir = 0;
    int i = 0;
    int j = 0;
    while (map[i][j] == ' ')
        j++;
    for (;;)
    {
        int steps = read_int(&w);
        for (int s = 0; s < steps; s++)
        {
            map[i][j] = ">v<^"[dir];
            int next_i = i;
            int next_j = j;
            int next_dir = dir;
            int dir_i = next_dir == 1 ? 1 : next_dir == 3 ? H-1 : 0;
            int dir_j = next_dir == 0 ? 1 : next_dir == 2 ? W-1 : 0;
            next_i = (next_i + dir_i) % H;
            next_j = (next_j + dir_j) % W;
            if (next_dir == 0)
            {
                if (   side(0, 0, 2, 1, 2, &next_i, &next_j, &next_dir)
                    || side(1, 2, 0, 2, 3, &next_i, &next_j, &next_dir)
                    || side(2, 2, 0, 2, 2, &next_i, &next_j, &next_dir)
                    || side(3, 1, 2, 1, 3, &next_i, &next_j, &next_dir));
            }
            else if (next_dir == 1)
            {
                if (   side(0, 0, 0, 2, 0, &next_i, &next_j, &next_dir)
                    || side(3, 1, 3, 0, 1, &next_i, &next_j, &next_dir)
                    || side(1, 2, 1, 1, 1, &next_i, &next_j, &next_dir));
            }
            else if (next_dir == 2)
            {
                if (   side(0, 0, 2, 0, 2, &next_i, &next_j, &next_dir)
                    || side(1, 0, 2, 0, 3, &next_i, &next_j, &next_dir)
                    || side(2, 2, 0, 1, 2, &next_i, &next_j, &next_dir)
                    || side(3, 2, 0, 1, 3, &next_i, &next_j, &next_dir));
            }
            else if (next_dir == 3)
            {
                if (   side(1, 0, 1, 1, 1, &next_i, &next_j, &next_dir)
                    || side(3, 1, 3, 0, 1, &next_i, &next_j, &next_dir)
                    || side(3, 2, 3, 0, 0, &next_i, &next_j, &next_dir));
            }
            printf("%4d %4d %d ", next_i, next_j, next_dir);
            if (next_i < 0 || next_i >= H || next_j < 0 || next_j >= W)
            {
                printf("ERROR\n");
                return;
            }
            printf("%c\n", map[next_i][next_j]);
            if (map[next_i][next_j] == '#')
                break;
            i = next_i;
            j = next_j;
            dir = next_dir;
        }
        //printf("%d %d\n", steps, dir);
        //print_map();
        if (*w == 'R')
            dir = (dir + 1) % 4;
        else if (*w == 'L')
            dir = (dir + 3) % 4;
        else
            break;
        w++;
    }
    print_map();
    printf("%d %d %d : %d\n", i+1, j+1, dir, 1000 * (i+1) + 4 * (j+1) + dir); 
}

bool side(int fi, int fj, int ti, int tj, int dir, int *next_i, int *next_j, int *next_dir)
{
    int i = *next_i - 50 * fi;
    int j = *next_j - 50 * fj;
    if (0 <= i && i < 50 && 0 <= j && j < 50)
    {
        for (int d = 0; d < dir; d++)
        {
            int n_i = j;
            int n_j = 49 - i;
            i = n_i;
            j = n_j;
        }
        *next_i = i + 50 * ti;
        *next_j = j + 50 * tj;
        *next_dir = (*next_dir + dir) % 4;
        return TRUE;
    }
    return FALSE;
}
```

After fixing a small mistake, it did return the correct answer.

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
../IParse/software/MarkDownC Day22.md >p.c; gcc -Wall -g p.c -o p && ./p
```
