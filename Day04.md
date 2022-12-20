# Day 4 of Advent of Code 2022

I started around 11.

I understand it is about ranges. Lets define a structure for that:
```c
typedef struct range range_t;
struct range
{
    int f;
    int t;
};
```
And a function which returns true when the first range is a subset
of the second range:
```c
bool range_subset(range_t *first, range_t *second)
{
    return second->f <= first->f && first->t <= second->t;
}
```

And a function that can read a range from a string (which asumes that
the input is correct):
```c
void read_range(const char **ref_s, range_t *range)
{
    range->f = read_int(ref_s);
    (*ref_s)++;
    range->t = read_int(ref_s);
}
```

Now we can write the function to read the input:
```c
void read_input()
{
    FILE *f = fopen("input/day04.txt","r");
    if (f == 0)
        return;
    char buffer[100];
    while (fgets(buffer, 100, f))
    {
        process(buffer);
    }
    fclose(f);
}
```
And the code to process one line of the input:
```c
int answer1 = 0;

void process(const char *line)
{
    range_t first;
    range_t second;
    const char *s = line;
    read_range(&s, &first);
    s++;
    read_range(&s, &second);
    
    if (range_subset(&first, &second) || range_subset(&second, &first))
        answer1++;
}
```

And the code for `main`:
```c
int main(int argc, char *argv[])
{
    if (argc > 1)
    {
        process("2-4,6-8");
        process("2-3,4-5");
        process("5-7,7-9");
        process("2-8,3-7");
        process("6-6,4-6");
        process("2-6,4-8");    
    }
    else
        read_input();
    printf("%d\n", answer1);
}

```
After some debugging, I was reading the input file from yesterday (classical
cut and paste bug), I found the answer to the first puzzle at 11:34.

## Second puzzle

It seems we have to do a similar check, but now with overlap. Lets write a
function for that:
```c
bool range_overlap(range_t *first, range_t *second)
{
    return first->f <= second->t && second->f <= first->t;
}
```

And now we only need to extend the `process` and `main` functions:
```c
int answer2 = 0;

void process(const char *line)
{
    ...
    if (range_overlap(&first, &second))
        answer2++;
}

int main(int argc, char *argv[])
{
    ...
    printf("%d\n", answer2);
}
```

With this, I found the correct answer at 11:43.

Lets see, if I can extend [the language](Language.md) to solve todays
puzzle.
 
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
../IParse/software/MarkDownC Day04.md >day04.c; gcc day04.c -o day04; ./day04
```
