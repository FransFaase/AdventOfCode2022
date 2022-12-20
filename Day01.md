# Day 1 of Advent of Code 2022

At 9:25, I started reading the puzzle. Okay, it is a simple adding and
determining maximum of puzzle. I saved my input to the [day01_1.txt](input/day01_1.txt) file.

The grammar of the input is like:

```
root : elf_data SEQ .
elf_data : (int '\n') SEQ '\n' .
```

We can store this in a list of list. For the example input this could be represented
as (where we use square bracket to represent ordered collection):
```
input =
[[1000 2000 3000]
 [4000]
 [5000 6000]
 [7000 8000 9000]
 [10000]]
```

And what we have to calculate is:
```
answer = Max (Sum l | l in input)
```

To write a program to process the puzzle input in C, we first have to write some
code to read the input. We assume that the numbers on the input fit in the `long long`
type and that each input line is at most 30 characters long.

```c
void read_input()
{
    FILE *f = fopen("input/day01.txt","r");
    if (f == 0)
        return;
    read_file(f);
    fclose(f);    
}

void read_file(FILE *f)
{
    while (read_elf_data(f))
        ;
}

bool read_elf_data(FILE *f)
{
    for (;;)
    {
        char buffer[30];
        if (!fgets(buffer, 30, f))
            return FALSE;
        if (buffer[0] == '\n' || buffer[0] == '\r')
            break;
        add_value_for_elf(parse_int(buffer));
    }
    done_for_elf();
    return TRUE;
}

long long parse_int(const char *s)
{
    long long v = 0;
    for (; '0' <= *s && *s <= '9'; s++)
        v = 10 * v + *s - '0';
    return v;
}

```

To test if this parses the input, lets write some implementation for
the missing functions that prints the values for each elf on a single line:
```c
void add_value_for_elf(long long v)
{
    printf(" %lld", v);
}

void done_for_elf()
{
    printf("\n");
}

int main(int argc, char *argv[])
{
    read_input();
    return 0;
}
```

Okay. that seems to work. (I stopped at 10:19. And continued at 16:42. In the
mean time, I have been thinking about developing a language and interpreter
for this.)

## Summing and maximum

Just using some global variables for the summing and determining the maximum
value:
```c

long long max = 0;
long long sum = 0;

void add_value_for_elf(long long v)
{
    sum += v;
}

void done_for_elf()
{
    if (sum > max) max = sum;
    sum = 0;
}

int main(int argc, char *argv[])
{
    read_input();
    printf("%lld\n", max);
    return 0;
}
```

And then I discovered that MarkDownC does not output global variable definitions.
I fixed this with commit [1f095e69](https://github.com/FransFaase/IParse/commit/1f095e69959cb12563ebc1df2eab0184a8a31d0a)
of the IParse repository. Then the program did compile, run and returned the correct answer

## The second half of the puzzle

It seems you have to keep track of the top three highest sums. Which comes down
to sorting the values and summing the top three highest sums. If we would have a function
to get highest values from a list, we could specify this as:

```
answer = Max(Highest(3, (Sum(l) | l in input)))
```

This can be done with the following code:

```c
long long highest[3];
int nr_highest = 0;

void done_for_elf()
{
    for (int i = 0; i < nr_highest; i++)
    {
        if (sum > highest[i])
        {
            long long temp = highest[i];
            highest[i] = sum;
            sum = temp;
        }
    }
    if (nr_highest < 3)
    {
        highest[nr_highest++] = sum;
    }
    sum = 0;
}

int main(int argc, char *argv[])
{
    read_input();
    printf("%lld\n", highest[0] + highest[1] + highest[2]);
    return 0;
}
```

That resulted in a correct answer for the second part of the puzzle.

### Some standard definitions

```c
typedef int bool;
#define FALSE 0
#define TRUE 1
```

The command I use to process this markdown file, is:
```
../IParse/software/MarkDownC Day01.md >day01.c; gcc day01.c -o day01; ./day01
```
