# Day 9 of Advent of Code 2022

I opened my laptop just after 8 in the morning. I read the
puzzle description. Lets first write the function reading
the input.
```c
void read_input()
{
    FILE *f = fopen("input/day09.txt", "r");
    if (f == 0)
        return;
    char buffer[200];
    while (fgets(buffer, 200, f))
    {
        parse_line(buffer);
    }
    fclose(f);
}

void parse_line(const char *line)
{
    char dir = line[0];
    const char *s = line + 2;
    int steps = read_int(&s);
    process(dir, steps);
}

void process(char dir, int steps)
{
    printf("%c %d\n", dir, steps);
}

int main(int argc, char *argv[])
{
    read_input();
}
```
That seems to work. (8:15-8:19, started boiling some eggs.)
It looks like we have two positions that we have to track and
a field of places. I pressume we start at the left bottom corner.
I am first goint to test my algorith against the example input.
```c
//#define SIZE 6

bool field[SIZE][SIZE];

int h_x, h_y;
int t_x, t_y;

void init()
{
    for (int i = 0; i < SIZE; i++)
        for (int j = 0; j < SIZE; j++)
            field[i][j] = FALSE;
    h_x = h_y = t_x = t_y = 0;
}

void process(char dir, int steps)
{
    for (int s = 0; s < steps; s++)
    {
        if (dir == 'U') h_y++;
        if (dir == 'D') h_y--;
        if (dir == 'R') h_x++;
        if (dir == 'L') h_x--;
        if (t_x < h_x - 1) { t_x = h_x - 1; t_y = h_y; }
        if (t_x > h_x + 1) { t_x = h_x + 1; t_y = h_y; }
        if (t_y < h_y - 1) { t_y = h_y - 1; t_x = h_x; }
        if (t_y > h_y + 1) { t_y = h_y + 1; t_x = h_x; }
        field[t_x][t_y] = TRUE;
        for (int y = SIZE-1; y >= 0; y--)
        {
            for (int x = 0; x < SIZE; x++)
                if (x == h_x && y == h_y)
                    printf("H");
                else if (x == t_x && y == t_y)
                    printf("T");
                else
                    printf(".");
            printf("\n");
        }
        printf("\n");
    }
}

        
int main(int argc, char *argv[])
{
    init();
    if (argc > 1)
    {
        parse_line("R 4");
        parse_line("U 4");
        parse_line("L 3");
        parse_line("D 1");
        parse_line("R 4");
        parse_line("D 1");
        parse_line("L 5");
        parse_line("R 2");
    }
    else
        read_input();
    long count = 0;
    for (int y = SIZE-1; y >= 0; y--)
    {
        for (int x = 0; x < SIZE; x++)
        {
            printf("%c", field[x][y] ? '#' : '.');
            if (field[x][y])
                count++;
        }
        printf("\n");
    }
    printf("%ld\n", count);    
}
```

Okay, after some debugging, it seems to work for the example
input. Now lets try my input with size 100.
```c
//#define SIZE 100

void process(char dir, int steps)
{
    for (int s = 0; s < steps; s++)
    {
        if (dir == 'U') h_y++;
        if (dir == 'D') h_y--;
        if (dir == 'R') h_x++;
        if (dir == 'L') h_x--;
        if (t_x < h_x - 1) { t_x = h_x - 1; t_y = h_y; }
        if (t_x > h_x + 1) { t_x = h_x + 1; t_y = h_y; }
        if (t_y < h_y - 1) { t_y = h_y - 1; t_x = h_x; }
        if (t_y > h_y + 1) { t_y = h_y + 1; t_x = h_x; }
        mark(t_x, t_y);
    }
}

int mark(int t_x, int t_y)
{
    if (t_x >= 0 && t_x < SIZE && t_y >= 0 && t_y < SIZE)
        field[t_x][t_y] = TRUE;
    else
        printf(" %d,%d", t_x, t_y); 
}
```

It does get much larger, and even goes into negative values.
I noticed that there is a bug in MarkDownC that it cannot deal with
repeated `#define` statements. So, had to comment out something in
my earlier code.

```c
#define SIZE 1000
#define OFFSET 500

int mark(int t_x, int t_y)
{
    t_x += OFFSET;
    t_y += OFFSET;
    if (t_x >= 0 && t_x < SIZE && t_y >= 0 && t_y < SIZE)
        field[t_x][t_y] = TRUE;
    else
    {
        printf(" %d,%d", t_x, t_y);
        exit(1);
    } 
}
```

### Second part of the puzzle.

Lets see if it is possible to do the second puzzle in 10
minutes. I started at 9:17.

```c
int r_x[10];
int r_y[10];

void init()
{
    ...
    for (int r = 0; r < 10; r++)
        r_x[r] = r_y[r] = 0;
}

void process(char dir, int steps)
{
    for (int s = 0; s < steps; s++)
    {
        if (dir == 'U') r_y[0]++;
        if (dir == 'D') r_y[0]--;
        if (dir == 'R') r_x[0]++;
        if (dir == 'L') r_x[0]--;
        for (int r = 1; r < 10; r++)
        {
            if (r_x[r] < r_x[r-1] - 1) { r_x[r] = r_x[r-1] - 1; r_y[r] = r_y[r-1]; }
            if (r_x[r] > r_x[r-1] + 1) { r_x[r] = r_x[r-1] + 1; r_y[r] = r_y[r-1]; }
            if (r_y[r] < r_y[r-1] - 1) { r_y[r] = r_y[r-1] - 1; r_x[r] = r_x[r-1]; }
            if (r_y[r] > r_y[r-1] + 1) { r_y[r] = r_y[r-1] + 1; r_x[r] = r_x[r-1]; }
        }
        mark(r_x[9], r_y[9]);
    }
}
```

That did not work because I used a rope of 11, while it should have been 10.
I tested this with the example for the second puzzle:

```c

int main(int argc, char *argv[])
{
    init();
    if (argc > 1)
    {
        parse_line("R 5");
        parse_line("U 8");
        parse_line("L 8");
        parse_line("D 3");
        parse_line("R 17");
        parse_line("D 10");
        parse_line("L 25");
        parse_line("U 20");
    }
    else
        read_input();
    long count = 0;
    for (int y = SIZE-1; y >= 0; y--)
    {
        for (int x = 0; x < SIZE; x++)
        {
            printf("%c", field[x][y] ? '#' : '.');
            if (field[x][y])
                count++;
        }
        printf("\n");
    }
    printf("%ld\n", count);    
}
```
Now, I got the correct answer for the example input, but not for my input.
I guess, I have to leave it with this now and go to work first. (At 9:39)
There is a warning in the puzzle text saying: 'be careful: more types of motion
are possible than before.'

In the evening, I spend some debugging and puzzling. I came up with the
following solution for the `process` function:
```c
void process(char dir, int steps)
{
    for (int s = 0; s < steps; s++)
    {
        if (dir == 'U') r_y[0]++;
        if (dir == 'D') r_y[0]--;
        if (dir == 'R') r_x[0]++;
        if (dir == 'L') r_x[0]--;
        for (int r = 1; r < 10; r++)
        {
            if (   r_x[r] < r_x[r-1] - 1 || r_x[r] > r_x[r-1] + 1
                || r_y[r] < r_y[r-1] - 1 || r_y[r] > r_y[r-1] + 1)
            {
                if (r_x[r] < r_x[r-1]) r_x[r]++;
                if (r_x[r] > r_x[r-1]) r_x[r]--;
                if (r_y[r] < r_y[r-1]) r_y[r]++;
                if (r_y[r] > r_y[r-1]) r_y[r]--;
            }
        }
        mark(r_x[9], r_y[9]);
    }
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
../IParse/software/MarkDownC Day09.md >day09.c; gcc day09.c -o day09; ./day09
```
