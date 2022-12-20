# Day 12 of Advent of Code 2022

We have to read the input in a two dimensional array.
My input it 42 lines by 161 columns.

```c
#define LINES 41
#define COLS 162

char map[LINES][COLS];

void read_input()
{
    FILE *f = fopen("input/day12.txt", "r");
    if (f == 0)
        return;
    for (int i = 0; i < LINES; i++)
    {
        char buffer[200];
        fgets(buffer, 200, f);
        for (int j = 0; j < COLS; j++)
            map[i][j] = buffer[j];
    }
}

int main(int argc, char *argv[])
{
    read_input();
    for (int i = 0; i < LINES; i++)
    {
        for (int j = 0; j < COLS; j++)
            printf("%c", map[i][j]);
        printf("\n");
    }
}
```
I was mistaken. It was 41 lines. So now have to find
the shortest way from the start to the end. I think it
is a good idea to create a map of the number of steps
to reach a certain location.
```c
int steps[LINES][COLS];

#define MAX 1000000

void answer1()
{
    for (int i = 0; i < LINES; i++)
        for (int j = 0; j < COLS; j++)
            steps[i][j] = map[i][j] == 'S' ? 0 : MAX;
            
    for (bool go = TRUE; go; )
    {
        printf("X\n");
        go = FALSE;
        for (int i = 0; i < LINES - 1; i++)
            for (int j = 0; j < COLS; j++)
            {
                if (connect(i, j, i+1, j)) go = TRUE;
                if (connect(i+1, j, i, j)) go = TRUE;
            }
        for (int i = 0; i < LINES; i++)
            for (int j = 0; j < COLS - 1; j++)
            {
                if (connect(i, j, i, j+1)) go = TRUE;
                if (connect(i, j+1, i, j)) go = TRUE;
            }
        for (int i = 0; i < LINES; i++)
        {
            for (int j = 0; j < COLS; j++)
                printf("%c", steps[i][j] < MAX ? map[i][j] : ' ');
            printf("\n");
        }
    }
    for (int i = 0; i < LINES; i++)
        for (int j = 0; j < COLS; j++)
            if (map[i][j] == 'E')
                printf("%d,%d: %d\n", i, j, steps[i][j]);
}

bool connect(int i1, int j1, int i2, int j2)
{
    int fs = steps[i1][j1];
    int *ts = &steps[i2][j2];
    if (*ts > fs + 1)
    {
        char f = map[i1][j1];
        char t = map[i2][j2];
        if (t == 'E' || t <= f + 1 || f == 'S')
        {
            *ts = fs + 1;
            if (t == 'E')
                printf("%d\n", *ts);
            return TRUE;
        }
    }
    return FALSE;
}

int main(int argc, char *argv[])
{
    ...
    answer1();
}

```
After some debugging and reading the description. I found the solution for
the first part.

### Second part

This is actually not so difficult as it seems, you just have to walk into
the other direction. I just copy the code from above and swap the function
of the `fs` and `ts` variables. And of course, have to add some code to find
the 'a' with the minimal value.
```c

void answer2()
{
    for (int i = 0; i < LINES; i++)
        for (int j = 0; j < COLS; j++)
            steps[i][j] = map[i][j] == 'E' ? 0 : MAX;
            
    for (bool go = TRUE; go; )
    {
        printf("X\n");
        go = FALSE;
        for (int i = 0; i < LINES - 1; i++)
            for (int j = 0; j < COLS; j++)
            {
                if (connect2(i, j, i+1, j)) go = TRUE;
                if (connect2(i+1, j, i, j)) go = TRUE;
            }
        for (int i = 0; i < LINES; i++)
            for (int j = 0; j < COLS - 1; j++)
            {
                if (connect2(i, j, i, j+1)) go = TRUE;
                if (connect2(i, j+1, i, j)) go = TRUE;
            }
        for (int i = 0; i < LINES; i++)
        {
            for (int j = 0; j < COLS; j++)
                printf("%c", steps[i][j] < MAX ? map[i][j] : ' ');
            printf("\n");
        }
    }
    int min = MAX;
    for (int i = 0; i < LINES; i++)
        for (int j = 0; j < COLS; j++)
            if (map[i][j] == 'a' && steps[i][j] < min)
                min = steps[i][j];
    printf("%d\n", min);
}

bool connect2(int i1, int j1, int i2, int j2)
{
    int *fs = &steps[i1][j1];
    int ts = steps[i2][j2];
    if (*fs > ts + 1)
    {
        char f = map[i1][j1];
        char t = map[i2][j2];
        if (t == 'E' || t <= f + 1 || f == 'S')
        {
            *fs = ts + 1;
            return TRUE;
        }
    }
    return FALSE;
}

int main(int argc, char *argv[])
{
    ...
    answer2();
}
```

With that I found the solution.

### Some standard definitions

```c
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
../IParse/software/MarkDownC Day12.md >day12.c; gcc -Wall day12.c -o day12; ./day12
```
