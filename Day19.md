# Day 19 of Advent of Code 2022

Lets first write the function to read the input and
meanwhile think a bit about the approach to solve this
puzzle.

```c
void read_input()
{
    FILE *f = fopen("input/day19.txt", "r");
    if (f == 0)
        return;
    char buffer[200];
    while (fgets(buffer, 200, f))
    {
        const char *s = buffer;
        s += strlen("Blueprint ");
        nr = read_int(&s);
        s += strlen(": Each ore robot costs ");
        cost_ore = read_int(&s);
        s += strlen(" ore. Each clay robot costs ");
        cost_clay = read_int(&s);
        s += strlen(" ore. Each obsidian robot costs ");
        cost_obsidian = read_int(&s);
        s += strlen(" ore and ");
        clay_obsidian = read_int(&s);
        s += strlen(" clay. Each geode robot costs ");
        cost_geode = read_int(&s);
        s += strlen(" ore and ");
        obsidian_geode = read_int(&s);
        process();
    }
    fclose(f);
}

int nr;
int cost_ore;
int cost_clay;
int cost_obsidian;
int clay_obsidian;
int cost_geode;
int obsidian_geode;

void process()
{
    printf("Blueprint %d: Each ore robot costs %d ore. Each clay robot costs %d ore. Each obsidian robot costs %d ore and %d clay. Each geode robot costs %d ore and %d obsidian.\n",
        nr,
        cost_ore,
        cost_clay,
        cost_obsidian,
        clay_obsidian,
        cost_geode,
        obsidian_geode);
}

int main(int argc, char *argv[])
{
    read_input();
}
```

That produces the same output as the input. Lets try for the recursive
approach.

```c
void process()
{
    max_geode = 0;
    search(0, 0, 0, 0, 0, 1, 0, 0, 0);
}

int max_geode;

void search(int minute, int a_ore, int a_clay, int a_obsidian, int a_geode, int a_ore_robot, int a_clay_robot, int a_obsidian_robot, int a_geode_robot)
{
    //printf("%*.*ssearch %2d %2d %2d %2d | %2d %2d %2d %2d\n", minute, minute, "", a_ore, a_clay, a_obsidian, a_geode, a_ore_robot, a_clay_robot, a_obsidian_robot, a_geode_robot);
    if (minute == 24)
    {
        if (a_geode > max_geode)
        {
            //printf("%d\n", a_geode);
            max_geode = a_geode;
        }
        return;
    }
    minute++;
    int options = 0;
    if (a_ore >= cost_ore)
    {
        search(minute, 
            a_ore      + a_ore_robot      - cost_ore, 
            a_clay     + a_clay_robot, 
            a_obsidian + a_obsidian_robot,
            a_geode    + a_geode_robot,
            a_ore_robot + 1, a_clay_robot, a_obsidian_robot, a_geode_robot);
    }
    if (a_ore >= cost_clay)
    {
        search(minute, 
            a_ore      + a_ore_robot      - cost_clay, 
            a_clay     + a_clay_robot, 
            a_obsidian + a_obsidian_robot,
            a_geode    + a_geode_robot,
            a_ore_robot, a_clay_robot + 1, a_obsidian_robot, a_geode_robot);
        options++;
    }
    if (a_ore >= cost_obsidian && a_clay >= clay_obsidian)
    {
        search(minute,
            a_ore      + a_ore_robot      - cost_obsidian,
            a_clay     + a_clay_robot     - clay_obsidian, 
            a_obsidian + a_obsidian_robot, 
            a_geode    + a_geode_robot, 
            a_ore_robot, a_clay_robot, a_obsidian_robot + 1, a_geode_robot);
        options++;
    }
    if (a_ore >= cost_geode && a_obsidian >= obsidian_geode)
    {
        search(minute,
            a_ore      + a_ore_robot      - cost_geode, 
            a_clay     + a_clay_robot,
            a_obsidian + a_obsidian_robot - obsidian_geode, 
            a_geode    + a_geode_robot,
            a_ore_robot, a_clay_robot, a_obsidian_robot, a_geode_robot + 1);
        options++;
    }
    if (a_ore < cost_ore || a_ore < cost_clay || a_ore < cost_obsidian || a_ore < cost_geode)
        search(minute,
            a_ore + a_ore_robot,
            a_clay + a_clay_robot,
            a_obsidian + a_obsidian_robot,
            a_geode + a_geode_robot,
            a_ore_robot, a_clay_robot, a_obsidian_robot, a_geode_robot);
}
```

And lets first test this on the example input to verify the above code:
```c
int main(int argc, char *argv[])
{
    nr = 1;
    cost_ore = 4;
    cost_clay = 2;
    cost_obsidian = 3;
    clay_obsidian = 14;
    cost_geode = 2;
    obsidian_geode = 7;
    process();
    printf("max = %d\n", max_geode);
    nr = 1;
    cost_ore = 2;
    cost_clay = 3;
    cost_obsidian = 3;
    clay_obsidian = 8;
    cost_geode = 3;
    obsidian_geode = 12;
    process();
    printf("max = %d\n", max_geode);
}
```

The code for letting it work on my input file:
```c
void process()
{
    ...
    answer1 += nr * max_geode;
    printf("%d %d %d\n", nr, max_geode, answer1);
}

int answer1 = 0;
int main(int argc, char *argv[])
{
    read_input();
    printf("%d\n", answer1);
}
```

It does not return the correct answer. I stopped after having
worked on it for about two hours in the morning.

In the evening, I continued with some debugging, concluding that
the algorithm is correct for the two examples. No where in the text it
says that at most one robot can be constructed at once. I spend a lot
of experimenting with all kind of variants of the code, to see if making
more than one robot per minute could be the answer, but I found that if
I do this, the second blueprint returns 13 instead of 12.

It is indeed the case that you can build but one robot per minute.

I found the clue on [reddit](https://www.reddit.com/r/adventofcode/comments/zpy5rm/2022_day_19_what_are_your_insights_and/):

    Note that we can do a bit better: For any resource R that's not geode:
    if you already have X robots creating resource R, a current stock of Y for
    that resource, T minutes left, and no robot requires more than Z of resource
    R to build, and X * T+Y >= T * Z, then you never need to build another robot mining R anymore.

I suspect that my last if statement in the `search` function is incorrect.

```c
void search(int minute, int a_ore, int a_clay, int a_obsidian, int a_geode, int a_ore_robot, int a_clay_robot, int a_obsidian_robot, int a_geode_robot)
{
    //printf("%*.*ssearch %2d %2d %2d %2d | %2d %2d %2d %2d\n", minute, minute, "", a_ore, a_clay, a_obsidian, a_geode, a_ore_robot, a_clay_robot, a_obsidian_robot, a_geode_robot);
    if (minute == 24)
    {
        if (a_geode > max_geode)
        {
            //printf("%d\n", a_geode);
            max_geode = a_geode;
        }
        return;
    }
    minute++;
    int n_ore      = a_ore      + a_ore_robot;
    int n_clay     = a_clay     + a_clay_robot;
    int n_obsidian = a_obsidian + a_obsidian_robot;
    int n_geode    = a_geode    + a_geode_robot;
    int options = 0;
    if (a_ore >= cost_ore)
    {
        search(minute, 
            n_ore      - cost_ore, 
            n_clay, 
            n_obsidian,
            n_geode,
            a_ore_robot + 1, a_clay_robot, a_obsidian_robot, a_geode_robot);
    }
    if (a_ore >= cost_clay)
    {
        search(minute, 
            n_ore - cost_clay, 
            n_clay, 
            n_obsidian,
            n_geode,
            a_ore_robot, a_clay_robot + 1, a_obsidian_robot, a_geode_robot);
        options++;
    }
    if (a_ore >= cost_obsidian && a_clay >= clay_obsidian)
    {
        search(minute,
            n_ore - cost_obsidian,
            n_clay - clay_obsidian, 
            n_obsidian, 
            n_geode, 
            a_ore_robot, a_clay_robot, a_obsidian_robot + 1, a_geode_robot);
        options++;
    }
    if (a_ore >= cost_geode && a_obsidian >= obsidian_geode)
    {
        search(minute,
            n_ore - cost_geode, 
            n_clay,
            n_obsidian - obsidian_geode, 
            n_geode,
            a_ore_robot, a_clay_robot, a_obsidian_robot, a_geode_robot + 1);
        options++;
    }
    if (a_ore <= cost_ore || a_ore <= cost_clay || a_ore <= cost_obsidian || a_ore <= cost_geode)
        search(minute,
            n_ore,
            n_clay,
            n_obsidian,
            n_geode,
            a_ore_robot, a_clay_robot, a_obsidian_robot, a_geode_robot);
}
```

Indeed, it looks like the last condition was wrong. Just changed the '<' into '<=' was enough
to find the correct answer for the first part of the puzzle.

### Second part

So, for the second part, you just have to go from 24 to 32 minutes. And you only have to process
the first three inputs. I guess that my function is too slow for doing this. Due to all the
useless repetitions. If I try it on the example input, it indeed takes far too long.

The cost for building a robot is never more than four units. So, if we never need more than
four ore building robots. Lets also implement the clue mentioned above.

```c
#define MINUTES 32
void search(int minute, int a_ore, int a_clay, int a_obsidian, int a_geode, int a_ore_robot, int a_clay_robot, int a_obsidian_robot, int a_geode_robot)
{
    //printf("%*.*ssearch %2d %2d %2d %2d | %2d %2d %2d %2d\n", minute, minute, "", a_ore, a_clay, a_obsidian, a_geode, a_ore_robot, a_clay_robot, a_obsidian_robot, a_geode_robot);
    if (minute == MINUTES)
    {
        if (a_geode > max_geode)
        {
            //printf("%d\n", a_geode);
            max_geode = a_geode;
        }
        return;
    }
    
    int left = MINUTES - minute;
    
    minute++;
    int n_ore      = a_ore      + a_ore_robot;
    int n_clay     = a_clay     + a_clay_robot;
    int n_obsidian = a_obsidian + a_obsidian_robot;
    int n_geode    = a_geode    + a_geode_robot;
    int options = 0;
    if (a_ore >= cost_ore && a_ore_robot * left + a_ore < left * 4)
    {
        search(minute, 
            n_ore      - cost_ore, 
            n_clay, 
            n_obsidian,
            n_geode,
            a_ore_robot + 1, a_clay_robot, a_obsidian_robot, a_geode_robot);
    }
    if (a_ore >= cost_clay && a_clay_robot * left + a_clay < left * clay_obsidian)
    {
        search(minute, 
            n_ore - cost_clay, 
            n_clay, 
            n_obsidian,
            n_geode,
            a_ore_robot, a_clay_robot + 1, a_obsidian_robot, a_geode_robot);
        options++;
    }
    if (a_ore >= cost_obsidian && a_clay >= clay_obsidian)
    {
        search(minute,
            n_ore - cost_obsidian,
            n_clay - clay_obsidian, 
            n_obsidian, 
            n_geode, 
            a_ore_robot, a_clay_robot, a_obsidian_robot + 1, a_geode_robot);
        options++;
    }
    if (a_ore >= cost_geode && a_obsidian >= obsidian_geode)
    {
        search(minute,
            n_ore - cost_geode, 
            n_clay,
            n_obsidian - obsidian_geode, 
            n_geode,
            a_ore_robot, a_clay_robot, a_obsidian_robot, a_geode_robot + 1);
        options++;
    }
    if (a_obsidian_robot == 0 && (a_ore <= cost_ore || a_ore <= cost_clay || a_ore <= cost_obsidian || a_ore <= cost_geode))
        search(minute,
            n_ore,
            n_clay,
            n_obsidian,
            n_geode,
            a_ore_robot, a_clay_robot, a_obsidian_robot, a_geode_robot);
}
```

That takes some time, but it does finish for the example input. Lets adapt `process` for the second answer:

```c
int answer2 = 1;
void process()
{
    if (nr > 3) return;
    max_geode = 0;
    search(0, 0, 0, 0, 0, 1, 0, 0, 0);
    answer2 *= max_geode;
    printf("%d %d %d\n", nr, max_geode, answer2);
}
```

The output this produced was:
```
1 10 10
2 54 540
3 37 19980
0
```

I did not submit the answer, because I felt that I cheated.

But on Tuesday, December 20, I decided to see if the program would find an answer when I
removed the code that is based on the hint. I had figured out myself, that I never need
more than four ore robots. I also figured out a way to reduce the number of the last
recursive calls.

```c
int max_cost;

void search(int minute, int a_ore, int a_clay, int a_obsidian, int a_geode, int a_ore_robot, int a_clay_robot, int a_obsidian_robot, int a_geode_robot)
{
    //printf("%*.*ssearch %2d %2d %2d %2d | %2d %2d %2d %2d\n", minute, minute, "", a_ore, a_clay, a_obsidian, a_geode, a_ore_robot, a_clay_robot, a_obsidian_robot, a_geode_robot);
    if (minute == MINUTES)
    {
        if (a_geode > max_geode)
        {
            //printf("%d\n", a_geode);
            max_geode = a_geode;
        }
        return;
    }
    
    minute++;
    int n_ore      = a_ore      + a_ore_robot;
    int n_clay     = a_clay     + a_clay_robot;
    int n_obsidian = a_obsidian + a_obsidian_robot;
    int n_geode    = a_geode    + a_geode_robot;
    int options = 0;
    int need = 0;

    if (a_ore_robot < 4)
    {
        if (a_ore < cost_ore)
            need = cost_ore;
        else
        {
            search(minute, 
                n_ore      - cost_ore, 
                n_clay, 
                n_obsidian,
                n_geode,
                a_ore_robot + 1, a_clay_robot, a_obsidian_robot, a_geode_robot);
        }
    }

    if (a_ore >= cost_clay)
    {
        search(minute, 
            n_ore - cost_clay, 
            n_clay, 
            n_obsidian,
            n_geode,
            a_ore_robot, a_clay_robot + 1, a_obsidian_robot, a_geode_robot);
        options++;
    }
    else if (cost_clay > need)
        need = cost_clay;
        
    if (a_ore >= cost_obsidian && a_clay >= clay_obsidian)
    {
        search(minute,
            n_ore - cost_obsidian,
            n_clay - clay_obsidian, 
            n_obsidian, 
            n_geode, 
            a_ore_robot, a_clay_robot, a_obsidian_robot + 1, a_geode_robot);
        options++;
    }
    else if (cost_obsidian > need)
        need = cost_obsidian;

    if (a_ore >= cost_geode && a_obsidian >= obsidian_geode)
    {
        search(minute,
            n_ore - cost_geode, 
            n_clay,
            n_obsidian - obsidian_geode, 
            n_geode,
            a_ore_robot, a_clay_robot, a_obsidian_robot, a_geode_robot + 1);
        options++;
    }
    else if (cost_geode > need)
        need = cost_geode;

    if (a_ore <= need)
        search(minute,
            n_ore,
            n_clay,
            n_obsidian,
            n_geode,
            a_ore_robot, a_clay_robot, a_obsidian_robot, a_geode_robot);
}

void process()
{
    if (nr > 3) return;
    max_cost = cost_ore;
    if (cost_clay > max_cost) max_cost = cost_clay;
    if (cost_obsidian > max_cost) max_cost = cost_obsidian;
    if (cost_geode > max_cost) max_cost = cost_geode;
    max_geode = 0;
    search(0, 0, 0, 0, 0, 1, 0, 0, 0);
    answer2 *= max_geode;
    printf("%d %d %d\n", nr, max_geode, answer2);
}

```

The program required a bit more than half an hour to produce
the same answer as the earlier version (with some of the hints)
implemented. The answer was correct.

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
./IParse/software/MarkDownC Day19.md >p.c; gcc -Wall -g p.c -o p && ./p
```
