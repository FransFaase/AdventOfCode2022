# Day 3 of Advent of Code 2022

I first did some shopping and started my laptop at 10:42.

The input consists of lines with letters. Lets asume that these lines are
at most 100 characters long. The code to read the file is:
```c
void read_input()
{
    FILE *f = fopen("input/day03.txt","r");
    if (f == 0)
        return;
    char buffer[100];
    while (fgets(buffer, 100, f))
    {
        
        process(buffer);
    }
}
```
There are various ways to solve this puzzle. It seems that
every letter has some value. Values from 1 to 52. Lets use
an integer array to count the occurences of each letter in
both of the comparments.
```c
long answer1 = 0;

void process(const char *input)
{
    int first[53]; for (int v = 1; v <= 52; v++) first[v] = 0;
    int second[53]; for (int v = 1; v <= 52; v++) second[v] = 0;

    int len = strlen(input);
    if (len % 2 == 1) printf("Odd input\n");
    
    const char *s = input;
    for (int i = 0; i < len/2; i++)
        first[value(*s++)]++;
    for (int i = len/2; i < len; i++)
        second[value(*s++)]++;
    
    for (int v = 1; v <= 52; v++)
        if (first[v] > 0 && second[v] > 0)
        {
            //printf("%s %d\n", input, v);
            answer1 += v;
            break;
        }
}

int value(char c)
{
    if ('a' <= c && c <= 'z')
        return c - 'a' + 1;
    if ('A' <= c && c <= 'Z')
        return c - 'A' + 27;
    return 0;
}

int main(int argc, char *argv[])
{
    if (argc > 1)
    {
        process("vJrwpWtwJgWrhcsFMMfFFhFp");
        process("jqHRNqRjqzjGDLGLrsFMfFZSrLrFZsSL");
        process("PmmdzqPrVvPwwTWBwg");
        process("wMqvLMZHhHMvwLHjbvcjnnSBnvTQFn");
        process("ttgJtRGJQctTZtZT");
        process("CrZsJsPPZsGzwwsLwLmpwMDw");
    }
    else
        read_input();
    printf("%ld\n", answer1);
}

```

I did not get the right answer of the first try. I modified the code above to
try out with the example input. Strange, the program does give the right
answer for the example input. The difference is caused by the newline character
at the end of the input. This is fixed in the following `read_input` function:
```c
void read_input()
{
    FILE *f = fopen("input/day03.txt","r");
    if (f == 0)
        return;
    char buffer[100];
    while (fgets(buffer, 100, f))
    {
        remove_newline(buffer);
        process(buffer);
    }
}

void remove_newline(char *line)
{
    for (int i = strlen(line) - 1; i >= 0 && line[i] < ' '; i--)
        line[i] = '\0';
}

```

At about 11:25, I found the solution. I did read the second puzzle. But I had
to stop.

## Second puzzle

At 13:11, I continued. I think, it is best to count how many elves have
a certain item and than if three elves have been processed, see for which item
the value is 3. (I use the functionality of MarkDownC to extend existing functions
by using the `...` to indicate existing code.)

```c

int items[53];
int nr_elf = 0;
long answer2 = 0;

void process(const char *input)
{
    ...
    
    //printf("elf %d\n", nr_elf);
    if (nr_elf == 0)
        for (int v = 1; v <= 52; v++) items[v] = 0;
    nr_elf++;
    
    for (int v = 1; v <= 52; v++)
        if (first[v] > 0 || second[v] > 0)
            items[v]++;
    
    if (nr_elf == 3)
    {
        for (int v = 1; v <= 52; v++)
            if (items[v] == 3)
            {
                //printf("Together %d\n", v);
                answer2 += v;
                break;
            }
        nr_elf = 0;
    }
}
        
int main(int argc, char *argv[])
{
    ...
    
    printf("%ld\n", answer2);
}
```

For some reason the `...` are not working. I fixed this
in [commit 06d1c886](https://github.com/FransFaase/IParse/commit/06d1c886199ed75fd0543a3526278e14096e5236)
in the IParse repository. I also spend some time looking
into the unparse functionality. At 16:09:20, I submitted
the correct answer to the second part of todays puzzle.


### Some standard definitions

```c
typedef int bool;
#define FALSE 0
#define TRUE 1
```

The command I use to process this markdown file, is:
```
../IParse/software/MarkDownC Day03.md >day03.c; gcc day03.c -o day03; ./day03
```
