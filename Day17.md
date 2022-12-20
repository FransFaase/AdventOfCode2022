# Day 17 of Advent of Code 2022

The puzzle of today looks a lot easier than the one of yesterday,
but there are a lot of detais. I read the text around 11 in the
morning. It is now 14:23 in the afternoon. Here a method for
reading the input and a function that return the next airflow

```c

char jets[11000];

void read_input()
{
    FILE *f = fopen("input/day17.txt", "r");
    if (f == 0)
        return;
    fgets(jets, 11000, f);
    fclose(f);
    for (int i = strlen(jets); jets[i-1] < ' '; i--)
        jets[i-1] = '\0';
}

int main(int argc, char *argv[])
{
    read_input();
    for (int i = 0; i < nr_valves; i++)
    {
        printf("%2d %s %d %2d: ", i, valves[i].name, valves[i].rate, valves[i].nr_to);
        for (int j = 0; j < valves[i].nr_to; j++)
            printf(" %s", valves[i].to[j]->name);
        printf("\n");
    }
    cur_jet = jets;
}

char *cur_jet;

char next_jet()
{
    char jet = *cur_jet++;
    if (*cur_jet < ' ')
        cur_jet = jets;
    return jet;
}

```

How to represent the five kind of rocks. Lets careful read the
puzzle description first. Okay, lets represent the stones with
coordinates. Just asume each stone has five coordinated. Maybe
also height and width;
```c
typedef struct stone stone_t;
struct stone
{
    int height, width;
    int x[5];
    int y[5];
};
stone_t stones[5] = {
    { 1, 4, { 0, 1, 2, 3, 3 }, { 0, 0, 0, 0, 0 } },
    { 3, 3, { 0, 1, 1, 1, 2 }, { 1, 0, 1, 2, 1 } },
    { 3, 3, { 0, 1, 2, 2, 2 }, { 0, 0, 0, 1, 2 } },
    { 4, 1, { 0, 0, 0, 0, 0 }, { 0, 1, 2, 3, 3 } },
    { 2, 2, { 0, 0, 1, 1, 1 }, { 0, 1, 0, 1, 1 } },
};
```
Now the representation of the cave:
```c
#define MAX_NR_STONES 2022
#define HEIGHT ((1+(MAX_NR_STONES / 5))*13)
#define WIDTH 7

char cave[WIDTH][HEIGHT];

void init_cave()
{
    for (int x = 0; x < WIDTH; x++)
        for (int y = 0; y < HEIGHT; y++)
            cave[x][y] = '.';
}
```
Maybe also good to write a method for printing the
current state of the cave as shown in the puzzle description.
```c
int cur_height_stones = 0;
int stone_x;
int stone_y;
int stone_nr;

void print()
{
    int max_y = stone_y + stones[stone_nr].height;
    for (int y = max_y - 1; y >= 0; y--)
    {
        printf("|");
        for (int x = 0; x < WIDTH; x++)
        {
            char ch = cave[x][y];
            for (int i = 0; i < 5; i++)
                if (x == stone_x + stones[stone_nr].x[i] && y == stone_y + stones[stone_nr].y[i])
                    ch = '@';
            printf("%c", ch);
        }
        printf("|\n");
    }
    printf("+");
    for (int x = 0; x < WIDTH; x++)
        printf("-");
    printf("+\n\n");
}
```

Now, lets write some code to test the print routine for the
start position of the first stone.
```c
#define FROM_LEFT_WALL 2
#define ABOVE_TOP 3
void test_print()
{
    cur_height_stones = 0;
    stone_nr = 0;
    stone_x = FROM_LEFT_WALL;
    stone_y = cur_height_stones + ABOVE_TOP;
    
    print();
}

int main(int argc, char *argv[])
{
    read_input();
    init_cave();
    test_print();
}
```
After some fixes in the code above, it does print the first state.
Now lets write the run process:

```c
#define NR_STONES 5

void run(num_t nr_stones, bool debug_print)
{
    for (num_t s = 0; s < nr_stones; s++)
    {
        // place stone at start location
        stone_nr = s % 5;
        stone_x = FROM_LEFT_WALL;
        stone_y = cur_height_stones + ABOVE_TOP;
        
        for (;;)
        {
            if (debug_print) print();
            char jet = next_jet();
            if (debug_print) printf("%c\n", jet);
            if (jet == '<')
            {
                if (can_move_to(stone_x - 1, stone_y))
                    stone_x--;
            }
            else
            {
                if (can_move_to(stone_x + 1, stone_y))
                    stone_x++;
            }
            if (can_move_to(stone_x, stone_y - 1))
                stone_y--;
            else
            {
                place_stone();
                break;
            }
        }
    }
}

bool can_move_to(int x, int y)
{
    for (int i = 0; i < 5; i++)
        if (!cave_empty(stones[stone_nr].x[i] + x, stones[stone_nr].y[i] + y))
            return FALSE;
    return TRUE;
}

bool cave_empty(int x, int y)
{
    return 0 <= x && x < WIDTH && y >= 0 && cave[x][y] == '.';
}

void place_stone()
{
    for (int i = 0; i < 5; i++)
        cave[stones[stone_nr].x[i] + stone_x][stones[stone_nr].y[i] + stone_y] = '*';
    int h = stone_y + stones[stone_nr].height;
    if (h > cur_height_stones)
        cur_height_stones = h;
}

int main(int argc, char *argv[])
{
    strcpy(jets, ">>><<><>><<<>><>>><<<>>><<<><<<>><>><<>>");
    cur_jet = jets;
    init_cave();
    run(5, TRUE);
}
```

I got this working, after some debugging. The dimensions for the declaration
of `cave` were switched. Now lets run it for 2022 rocks and see how high it gets.
```c
int main(int argc, char *argv[])
{
    strcpy(jets, ">>><<><>><<<>><>>><<<>>><<<><<<>><>><<>>");
    cur_jet = jets;
    init_cave();
    run(2022, FALSE);
    printf("%d\n", cur_height_stones);
}
```
That did return the right answer for the example. Lets see what it returns for
my input:
```c
int main(int argc, char *argv[])
{
    read_input();
    cur_jet = jets;
    init_cave();
    run(2022, FALSE);
    printf("%d\n", cur_height_stones);
}
```
That did return the correct answer.

### Second part

Okay, I would expect that the second part would be a bit harder, but this is a lot
harder. I wonder how the top of the cave looks like the above and where the next jet
is in the sequence.
```c
int main(int argc, char *argv[])
{
    ...
    printf("%ld\n", cur_jet - jets);
    print();
}
```
It is somewhere in the middle of the jets.

Of course, we could move the contents of the cave, once a line is complety filled.
Adapt: `place_stone`:
```c

num_t moved_down = 0;

void place_stone()
{
    ...
    int y = stone_y;
    for (int j = 0; j < stones[stone_nr].height; j++, y++)
    {
        bool filled = TRUE;
        for (int x = 0; x < WIDTH; x++)
            if (cave[x][y] == '.')
            {
                filled = FALSE;
                break;
            }
        if (filled)
        {
            y++;
            int shift_y = y;
            //printf("Shift down: %d\n", shift_y);
            moved_down += shift_y;
            int y_to = 0;
            for (;y <= cur_height_stones; y++, y_to++)
                for (int x = 0; x < WIDTH; x++)
                    cave[x][y_to] = cave[x][y];
            for (;y_to <= cur_height_stones; y_to++)
                for (int x = 0; x < WIDTH; x++)
                    cave[x][y_to] = '.';
            cur_height_stones -= shift_y;
            break;
        }
    }
}

int main(int argc, char *argv[])
{
    read_input();
    cur_jet = jets;
    init_cave();
    run(1000000000000, FALSE);
    printf("%lld\n", moved_down + cur_height_stones);
}
```

I made some small modifications in the code of `run` above. Replaced
some `int` by `num_t` and also added a print statement that prints one
line every 1,000,000 steps. It prints about once per second. Because
there are 3600 seconds in a hour, it will take about 30 hours to
finish.

I was away for about an hour away, driving, and took some time to
think about the problem. 1000000000000 is a five fold. I got some idea.
Lets for each position in the jet string record when we arrived and how
many stones we have put there. My input is 10091 characters long:
```c
#define JETS 10091
int visits[JETS];
num_t at[JETS];
num_t height[JETS];
num_t diff_at[JETS];
num_t diff_height[JETS];

int main(int argc, char *argv[])
{
    read_input();
    cur_jet = jets;
    init_cave();
    for (int i = 0; i < JETS; i++)
        visits[i] = 0;
    num_t steps = 0;
    int cycle_length = 0;
    int cycle_height = 0;
    while (steps < 200000000000)
    {
        run(5, FALSE);
        steps += 1;
        int at_jet = cur_jet - jets;
        num_t h = moved_down + cur_height_stones;
        printf("At %6d", at_jet, steps, h);
        
        if (visits[at_jet] == 0)
        {
            at[at_jet] = steps;
            height[at_jet] = h;
            visits[at_jet] = 1;
            printf(" first\n");
        }
        else if (visits[at_jet] == 1)
        {
            diff_at[at_jet] = steps - at[at_jet];
            diff_height[at_jet] = h - height[at_jet];
            at[at_jet] = steps;
            height[at_jet] = h;
            visits[at_jet] = 2;
            printf(" diff %lld %lld\n", diff_at[at_jet], diff_height[at_jet]);
        }
        else
        {
            if (   diff_at[at_jet] == steps - at[at_jet]
                && diff_height[at_jet] == h - height[at_jet])
            {
                cycle_length = diff_at[at_jet];
                cycle_height = diff_height[at_jet];
                // looks like we found a cycle
                printf(" cycle??\n");
                break;
            }
            diff_at[at_jet] = steps - at[at_jet];
            diff_height[at_jet] = h - height[at_jet];
            at[at_jet] = steps;
            height[at_jet] = h;
            printf(" diff %lld %lld\n", diff_at[at_jet], diff_height[at_jet]);
        }
    }
    num_t tot_cycle_height = 0;
    if (cycle_length > 0)
    {
        while (steps + cycle_length < 200000000000)
        {
            steps += cycle_length;
            tot_cycle_height += cycle_height;
        }
    }
    for (; steps < 200000000000; steps++)
        run(5, FALSE);
    
    printf("%lld\n", moved_down + cur_height_stones + tot_cycle_height);
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
../IParse/software/MarkDownC Day17.md >day17.c; gcc -Wall -g day17.c -o day17; ./day17
```
