# Day 8 of Advent of Code 2022

Looks like it is needed to read the whole input in memory.
It looks my puzzle input is 99 by 99 large.
So lets first write a function for doing that:
```c
#define SIZE 99

char height[SIZE][SIZE];

void read_input()
{
    FILE *f = fopen("input/day08.txt", "r");
    if (f == 0)
        return;
    int r = 0;
    char buffer[200];
    while (fgets(buffer, 200, f))
    {
    	if (r >= SIZE)
        {
            printf("Error: %d rows\n", r);
            continue;
        }
        int len = strlen(buffer);
        while (buffer[len-1] < ' ')
            buffer[--len] = '\0';
        if (len != SIZE)
        {
            printf("Error: %d cols\n", len);
            continue;
        }
        for (int c = 0; c < SIZE; c++)
            height[r][c] = buffer[c];
        r++;
    }
    if (r != SIZE)
        printf("Error: %d rows\n", r);
}

int main(int argc, char *argv[])
{
    read_input();
}
```

That looks like it is working. We have to find the number
of visible trees. A tree is visible when it in at least
one direction only trees that are lower.
```c
void puzzle1()
{
    long nr_visible = 0;
    for (int r = 0; r < SIZE; r++)
        for (int c = 0; c < SIZE; c++)
            if (   all_lower(r, c, -1, 0)
                || all_lower(r, c, 0, -1)
                || all_lower(r, c, 1, 0)
                || all_lower(r, c, 0, 1))
                nr_visible++;
    printf("%ld\n", nr_visible);
}

bool all_lower(int r, int c, int dr, int dc)
{
    int h = height[r][c];
    for (;;)
    {
        r += dr;
        c += dc;
        if (r < 0 || r >= SIZE || c < 0 || c >= SIZE)
            return TRUE;
        if (height[r][c] >= h)
            return FALSE;
    }
}

int main(int argc, char *argv[])
{
    ...
    puzzle1();
}
```

There was a small bug in the `read_input` function, which I
would have found, if I had written some function to output
the `height` array.

### Second puzzle

So, we have to find a tree for which the multiplication
of the number of trees (up and including the first tree
that is of equal or higher) that can be viewed in every
direction. It has no use to check trees on the border,
because in one direction their will be zero visible trees.
```c
void puzzle2()
{
    long max = 0;
    for (int r = 1; r < SIZE-1; r++)
        for (int c = 1; c < SIZE-1; c++)
        {
            int m = visible(r, c, -1, 0)
                  * visible(r, c, 0, -1)
                  * visible(r, c, 1, 0)
                  * visible(r, c, 0, 1);
            if (m > max)
                max = m;
        }
    printf("%ld\n", max);
}

int visible(int r, int c, int dr, int dc)
{
    int nr = 0;
    int h = height[r][c];
    for (;;)
    {
        r += dr;
        c += dc;
        if (r < 0 || r >= SIZE || c < 0 || c >= SIZE)
            break;
        nr++;
        if (height[r][c] >= h)
            break;
    }
    return nr;
}

int main(int argc, char *argv[])
{
    ...
    puzzle2();
}
```

### Some standard definitions

```c
typedef int bool;
#define FALSE 0
#define TRUE 1

int read_int(const char **ref_s)
{
    int v = 0;
    for (; '0' <= **ref_s && **ref_s <= '9'; (*ref_s)++)
        v = 10 * v + **ref_s - '0';
    return v;
}
```

The command I use to process this markdown file, is:
```
../IParse/software/MarkDownC Day08.md >day08.c; gcc day08.c -o day08; ./day08
```
