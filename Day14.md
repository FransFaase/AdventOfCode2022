# Day 14 of Advent of Code 2022

I started writing this as 7:28 in the morning. Lets read todays
puzzle. At 7:35, I finished reading the puzzle. I think that I
am going to read the input to figure out the area in which there
are rocks. 

```c
void read_input()
{
    FILE *f = fopen("input/day14.txt", "r");
    if (f == 0)
        return;
    char buffer[2000];
    while (fgets(buffer, 2000, f))
    {
        start_line();
        for (const char *s = buffer; ; s += 4)
        {
            int x = read_int(&s);
            s++;
            int y = read_int(&s);
            process_point(x, y);
            if (strncmp(s, " -> ", 4) != 0)
                break;
        }
    }
    
    fclose(f);
}

void start_line()
{
}

int min_x = 10000;
int max_x = -10000;
int min_y = 10000;
int max_y = -10000;


void process_point(int x, int y)
{
    if (x < min_x) min_x = x;
    if (x > max_x) max_x = x;
    if (y < min_y) min_y = y;
    if (y > max_y) max_y = y;
}

int main(int argc, char* argv[])
{
    read_input();
    printf("x %d-%d y %d-%d\n", min_x, max_x, min_y, max_y);
}
```

At 7:47, the above program printed:
```
x 483-534 y 13-168
```

Lets define a two dimensional array and draw the rocks:
```c

//int cave[200][200];

void init_cave()
{
    for (int x = 0; x < 200; x++)
        for (int y = 0; y < 200; y++)
            cave[x][y] = '.';
}

void print_cave()
{
    for (int y = 0; y < 200; y++)
    {
        for (int x = 0; x < 200; x++)
            printf("%c", cave[x][y]);
        printf("\n");
    }
}

bool prev = FALSE;
int prev_x;
int prev_y;

void start_line()
{
    prev = FALSE;
}

void process_point(int x, int y)
{
    x -= 400;
    if (prev)
    {
        for (;;)
        {
            cave[prev_x][prev_y] = '#';
            if (prev_x == x && prev_y == y)
                break;
            if (prev_x < x) prev_x++;
            if (prev_x > x) prev_x--;
            if (prev_y < y) prev_y++;
            if (prev_y > y) prev_y--;
        }
    }
    else
    {
        prev_x = x;
        prev_y = y;
        prev = TRUE;
    }
}

```

At 7:58, lets test this with the example input:

```c
int main(int argc, char *argv[])
{
    init_cave();
    start_line();
    process_point(498,4);
    process_point(498,6);
    process_point(496,6);
    start_line();
    process_point(503,4);
    process_point(502,4);
    process_point(502,9);
    process_point(494,9);
    print_cave();
}
```
I used a define for the translation of x. That lead to some syntax problem with MarkDownC.
So, I rewrote the above ocde a bit. At 8:06, lets try to run it. In the print out the
x and y axes where swapped. Fixed this in `print_cave`. I stopped at 8:11.

I continued at 8:28. Lets implement the function of dropping a stone:
```c
bool drop()
{
    int x = 500 - 400;
    int y = 0;
    while (y < 200)
    {
        if (cave[x][y+1] == '.')
            y++;
        else if (cave[x-1][y+1] == '.')
        {
            y++;
            x--;
        }
        else if (cave[x+1][y+1] == '.')
        {
            y++;
            x++;
        }
        else
        {
            cave[x][y] = 'o';
            break;
        }
    }
    return y < 200;
}

int main(int argc, char *argv[])
{
    ...
    int c = 0;
    while (drop())
        c++;
    print_cave();
    printf("%d\n", c);
}
```

At 8:37, it looks like it is working for the example input. Now lets try it on my
input file:
```c
int main(int argc, char *argv[])
{
    init_cave();
    read_input();
    int c = 0;
    while (drop())
        c++;
    print_cave();
    printf("%d\n", c);
}
```
At 8:40, that resulted in a correct answer.

### Second part

After having read the above, I realize that I need to make some modifications
to the program at various places. I think the array the represent the cave,
needs to be a lot wider, and also the stop condition for the drop needs to
be different. I am going to stop a bit now, at 8:44.

At 8:47, I continued.
```c
char cave[500][200];

void init_cave()
{
    for (int x = 0; x < 500; x++)
        for (int y = 0; y < 200; y++)
            cave[x][y] = '.';
}


void print_cave()
{
    for (int y = 0; y < 200; y++)
    {
        for (int x = 0; x < 500; x++)
            printf("%c", cave[x][y]);
        printf("\n");
    }
}

void process_point(int x, int y)
{
    x -= 250;
    if (prev)
    {
        for (;;)
        {
            cave[prev_x][prev_y] = '#';
            if (prev_x == x && prev_y == y)
                break;
            if (prev_x < x) prev_x++;
            if (prev_x > x) prev_x--;
            if (prev_y < y) prev_y++;
            if (prev_y > y) prev_y--;
        }
    }
    else
    {
        prev_x = x;
        prev_y = y;
        prev = TRUE;
    }
    if (y > max_y)
        max_y = y;
}

bool drop()
{
    int x = 500 - 250;
    int y = 0;
    while (y < 200)
    {
        if (cave[x][y+1] == '.')
            y++;
        else if (cave[x-1][y+1] == '.')
        {
            y++;
            x--;
        }
        else if (cave[x+1][y+1] == '.')
        {
            y++;
            x++;
        }
        else
        {
            cave[x][y] = 'o';
            break;
        }
    }
    return y < 200;
}

int main(int argc, char *argv[])
{
    init_cave();
    read_input();
    for (int x = 0; x < 500; x++)
        cave[x][max_y + 2] = '#';
    long c = 0;
    while (cave[500 - 250][0] == '.')
    {
        drop();
        c++;
    }
    print_cave();
    printf("%ld\n", c);
}
```

I notice that MarkDownC does not yet support redefinitions of variables.
So, I have to temporary remove the first declaration of `cave`.

It would have been much easier if I would have used defines for the
'magic' constants. Than I would not have had to repeat some of the
code above and also got the second answer a bit faster, because I at
first forgot to replace the one of the appearances of the constant.

### Some standard definitions

```c
#include "stdlib.h"
typedef int bool;
typedef int long long integer_t;
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
../IParse/software/MarkDownC Day14.md >day14.c; gcc -Wall -g day14.c -o day14; ./day14
```
