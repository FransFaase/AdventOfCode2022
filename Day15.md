# Day 15 of Advent of Code 2022

I started reading the puzzle description at 8:11 in the morning.
I think, I am first going to read my input file. I noticed that
it has 24 lines.

```c
typedef long long num_t;
typedef struct pairs pairs_t;
struct pairs
{
    num_t x_s;
    num_t y_s;
    num_t x_b;
    num_t y_b;
    num_t d;
};
#define NR 23
pairs_t pairs[NR];

void read_input()
{
    FILE *f = fopen("input/day15.txt", "r");
    if (f == 0)
        return;
    char buffer[200];
    for (int i = 0; fgets(buffer, 200, f); i++)
    {
        // Sensor at x=, y=: closest beacon is at x=, y=
        const char *s = buffer + strlen("Sensor at x=");
        pairs[i].x_s = read_int(&s);
        s += 4;
        pairs[i].y_s = read_int(&s);
        s += strlen(": closest beacon is at x=");
        pairs[i].x_b = read_int(&s);
        s += 4;
        pairs[i].y_b = read_int(&s);
        pairs[i].d = abs_num(pairs[i].x_s - pairs[i].x_b) + abs(pairs[i].y_s - pairs[i].y_b);
    }
    
    fclose(f);
}

int main(int argc, char *argv[])
{
    read_input();
    for (int p = 0; p < NR; p++)
        printf("%lld, %lld  %lld, %lld  %lld\n", pairs[p].x_s, pairs[p].y_s, pairs[p].x_b, pairs[p].y_b, pairs[p].d);
}
```
Some small mistakes. There are only 23 lines the input. Got it working at 8:35.

I noticed that the `abs` function works on the `int` type. Lets implement our own and use it in the above
code, just to be sure.

```c
num_t abs_num(num_t v) { return v < 0 ? -v : v; }
```

Now lets look at the formulea to determin which parts for each.
```c
#define Y 2000000

void step1()
{
    num_t y = Y;
    for (int p = 0; p < NR; p++)
    {
        num_t dist_y = abs_num(pairs[p].y_s - y);
        printf("dist_y = %lld ", dist_y);
        num_t dist_x = pairs[p].d - dist_y;
        printf("dist_x = %lld ", dist_x);
        num_t min_x = pairs[p].x_s - dist_x;
        num_t max_x = pairs[p].x_s + dist_x;
        printf("%lld   %lld:  %lld  %lld:  %lld\n",
            pairs[p].d,
            min_x, abs_num(pairs[p].x_s - min_x) + abs(pairs[p].y_s - y),
            max_x, abs_num(pairs[p].x_s - max_x) + abs(pairs[p].y_s - y));
    }
}

int main(int argc, char *argv[])
{
    read_input();
    step1();
}
```

It looks like some sensors are too far from the 2000000 line. Lets ignore those.
```c
void step2()
{
    num_t y = Y;
    for (int p = 0; p < NR; p++)
    {
        num_t dist_x = pairs[p].d - abs_num(pairs[p].y_s - y);
        num_t min_x = pairs[p].x_s - dist_x;
        num_t max_x = pairs[p].x_s + dist_x;
        if (abs_num(pairs[p].x_s - min_x) + abs(pairs[p].y_s - y) <= pairs[p].d)
        {
            printf("%2d:  %8lld  %8lld = %8lld\n", p, min_x, max_x, max_x - min_x + 1);
        }
    }
}

int main(int argc, char *argv[])
{
    read_input();
    step2();
}
```
It looks like some of these ranges are overlapping. Lets trim the pair wise overlapping ranges.
Seems like a good idea to store these with the `pair_t`.
```c
struct pairs
{
    ...
    bool in_range;
    num_t min_x;
    num_t max_x;
};

void step3()
{
    num_t y = Y;
    for (int p = 0; p < NR; p++)
    {
        num_t dist_x = pairs[p].d - abs_num(pairs[p].y_s - y);
        pairs[p].min_x = pairs[p].x_s - dist_x;
        pairs[p].max_x = pairs[p].x_s + dist_x;
        pairs[p].in_range = abs_num(pairs[p].x_s - pairs[p].min_x) + abs(pairs[p].y_s - y) <= pairs[p].d; 
        if (pairs[p].in_range)
        {
            printf("%2d:  %8lld  %8lld = %8lld\n", p, pairs[p].min_x, pairs[p].max_x, pairs[p].max_x - pairs[p].min_x + 1);
        }
    }
    printf("\n");
    
    // Trim
    for (int i = 0; i < NR-1; i++)
        for (int j = i+1; j < NR; j++)
            if (pairs[i].in_range && pairs[j].in_range && pairs[i].min_x < pairs[j].max_x && pairs[j].min_x < pairs[i].max_x)
            {
                printf("%2d %2d overlap: ", i, j);
                if (pairs[i].min_x <= pairs[j].min_x && pairs[j].max_x <= pairs[i].max_x)
                {
                    printf("%2d inside %2d\n", j, i);
                    pairs[j].in_range = FALSE;
                }
                else if (pairs[j].min_x <= pairs[i].min_x && pairs[i].max_x <= pairs[j].max_x)
                {
                    printf("%2d inside %2d\n", i, j);
                    pairs[i].in_range = FALSE;
                }
                else if (pairs[j].min_x <= pairs[i].min_x)
                {
                    printf("%2d left of %2d\n", j, i);
                    pairs[i].min_x = pairs[j].max_x + 1;
                }
                else if (pairs[i].min_x <= pairs[j].min_x)
                {
                    printf("%2d left of %2d\n", i, j);
                    pairs[j].min_x = pairs[i].max_x + 1;
                }
                else
                    printf("???\n");
            }

    num_t answer = 0;            
    for (int p = 0; p < NR; p++)
    {
        if (pairs[p].in_range)
        {
            printf("%2d:  %8lld  %8lld = %8lld\n", p, pairs[p].min_x, pairs[p].max_x, pairs[p].max_x - pairs[p].min_x + 1);
            answer += pairs[p].max_x - pairs[p].min_x + 1;
        }
        if (pairs[p].y_b == y)
            answer--;
    }
    printf("%lld\n", answer);
}

int main(int argc, char *argv[])
{
    read_input();
    step3();
}
```

The program did not return the correct answer. Lets try it on the example input:
```c
#define NR 14
#define Y 10
```

There was a small bug in the condition for when to pairs possibly overlapped. From the example input,
it is also the case that possible beacons should be excluded. And then it returned the correct answer
for the example input. Now lets try for my input again.
```c
#define NR 24
#define Y 2000000
```
And it is still not correct. In my input there are two sensors that connect to the beacon
at the 2000000. I added one two the answer returned by the program and got the right answer. The
correct code for `step3` is:
```c
void step3()
{
    num_t y = Y;
    for (int p = 0; p < NR; p++)
    {
        num_t dist_x = pairs[p].d - abs_num(pairs[p].y_s - y);
        pairs[p].min_x = pairs[p].x_s - dist_x;
        pairs[p].max_x = pairs[p].x_s + dist_x;
        pairs[p].in_range = abs_num(pairs[p].x_s - pairs[p].min_x) + abs(pairs[p].y_s - y) <= pairs[p].d; 
        if (pairs[p].in_range)
        {
            printf("%2d:  %8lld  %8lld = %8lld\n", p, pairs[p].min_x, pairs[p].max_x, pairs[p].max_x - pairs[p].min_x + 1);
        }
    }
    printf("\n");
    
    // Trim
    for (int i = 0; i < NR-1; i++)
        for (int j = i+1; j < NR; j++)
            if (pairs[i].in_range && pairs[j].in_range && pairs[i].min_x < pairs[j].max_x && pairs[j].min_x < pairs[i].max_x)
            {
                printf("%2d %2d overlap: ", i, j);
                if (pairs[i].min_x <= pairs[j].min_x && pairs[j].max_x <= pairs[i].max_x)
                {
                    printf("%2d inside %2d\n", j, i);
                    pairs[j].in_range = FALSE;
                }
                else if (pairs[j].min_x <= pairs[i].min_x && pairs[i].max_x <= pairs[j].max_x)
                {
                    printf("%2d inside %2d\n", i, j);
                    pairs[i].in_range = FALSE;
                }
                else if (pairs[j].min_x <= pairs[i].min_x)
                {
                    printf("%2d left of %2d\n", j, i);
                    pairs[i].min_x = pairs[j].max_x + 1;
                }
                else if (pairs[i].min_x <= pairs[j].min_x)
                {
                    printf("%2d left of %2d\n", i, j);
                    pairs[j].min_x = pairs[i].max_x + 1;
                }
                else
                    printf("???\n");
            }

    num_t answer = 0;            
    for (int p = 0; p < NR; p++)
    {
        if (pairs[p].in_range)
        {
            printf("%2d:  %8lld  %8lld = %8lld\n", p, pairs[p].min_x, pairs[p].max_x, pairs[p].max_x - pairs[p].min_x + 1);
            answer += pairs[p].max_x - pairs[p].min_x + 1;
        }
    }
    
    // correct for beacons on the line
    for (int i = 0; i < NR; i++)
    {
        if (pairs[i].y_b == y)
        {
            bool earlier = FALSE;
            for (int j = 0; j < i && !earlier; j++)
                earlier = pairs[i].x_b == pairs[j].x_b;
            if (!earlier)
                answer--;
        }
    }
     
    printf("%lld\n", answer);
}
```

### Second part

So, you have to find a location in a very large search space. I am going to leave this for the
evening and ponder about it during the day.

I continued at 22:22. I did think about it for some time. I think
I will go for a kind of brute force approach, scanning all lines
from 0 to 4000000.
```c
void answer2()
{
    for (num_t y = 0; y <= 4000000; y++)
    {
        for (int p = 0; p < NR; p++)
        {
            num_t dist_x = pairs[p].d - abs_num(pairs[p].y_s - y);
            pairs[p].min_x = pairs[p].x_s - dist_x;
            pairs[p].max_x = pairs[p].x_s + dist_x;
            pairs[p].in_range = abs_num(pairs[p].x_s - pairs[p].min_x) + abs(pairs[p].y_s - y) <= pairs[p].d; 
        }
        
        num_t x = 0;
        for (bool go = TRUE; go;)
        {
            go = FALSE;
            for (int p = 0; p < NR; p++)
                if (pairs[p].in_range && pairs[p].min_x <= x && x <= pairs[p].max_x)
                {
                    x = pairs[p].max_x + 1;
                    go = TRUE;
                    if (x > 4000000)
                    {
                        go = FALSE;
                        break;
                    }
                }
        }
        if (x <= 4000000)
        {
            printf("%lld %lld %lld\n", x, y, x * 4000000 + y);
        }
    }
}
                
int main(int argc, char *argv[])
{
    read_input();
    answer2();
}
```

The program ran for less than a second before finding the solution. Submitted
the answer at 22:33:34 (CET).


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
../IParse/software/MarkDownC Day15.md >day15.c; gcc -Wall -g day15.c -o day15; ./day15
```
