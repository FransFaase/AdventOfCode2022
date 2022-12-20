# Day 6 of Advent of Code 2022

I started around 8:10 in the morning.

The puzzle is to find the first four characters that are not equal.
The whole input is on a single line with 4096 characters. Lets read
that into a single buffer.
```c
void read_input()
{
    FILE *f = fopen("input/day06.txt", "r");
    if (f == 0)
        return;
    char buffer[5000];
    fgets(buffer, 5000, f);
    fclose(f);
    process_puzzle1(buffer);
}
```

And next write a function for the check:
```c
void process_puzzle1(const char *line)
{
    for (int i = 0; line[i] >= ' '; i++)
    {
        bool all_different = TRUE;
        for (int j = 0; j < 3 && all_different; j++)
            for (int k = j+1; k < 4 && all_different; k++)
                all_different = line[i+j] != line[i+k];
        if (all_different)
        {
            printf("%d\n", i + 4);
            break;
        }
    }
}
```

Now first, check with the example input, because I am pretty sure
that the above code is correct, I think their might be a one off
error.
```c
int main(int argc, char *argv[])
{
    process_puzzle1("bvwbjplbgvbhsrlpgdmjqwftvncz");
    process_puzzle1("nppdvjthqldpwncqszvftbrmjlhg");
    process_puzzle1("nznrnfrfntjfmvfwmzdfjlvtqnbhcprsg");
    process_puzzle1("zcfzfwzzqfrljwzlrfnpqdbhtmscgvjw");
    
    read_input();
}
```
There was a three off error in the code, which I fixed in the above.

### Second puzzle

It looks the same, except that the 4 is replaced by a 14. I am
going to write a generic function for the check.
```c
void process_puzzle(const char *line, int n)
{
    for (int i = 0; line[i] >= ' '; i++)
    {
        bool all_different = TRUE;
        for (int j = 0; j < n-1 && all_different; j++)
            for (int k = j+1; k < n && all_different; k++)
                all_different = line[i+j] != line[i+k];
        if (all_different)
        {
            printf("%d\n", i + n);
            break;
        }
    }
}

void process_puzzle1(const char *line) { process_puzzle(line, 4); }
void process_puzzle2(const char *line) { process_puzzle(line, 14); }

void read_input()
{
    ...
    process_puzzle2(buffer);
}

int main(int argc, char *argv[])
{
    process_puzzle2("mjqjpqmgbljsphdztnvjfqwrcgsmlb");
    process_puzzle2("bvwbjplbgvbhsrlpgdmjqwftvncz");
    process_puzzle2("nppdvjthqldpwncqszvftbrmjlhg");
    process_puzzle2("nznrnfrfntjfmvfwmzdfjlvtqnbhcprsg");
    
    read_input();
}
```
With this, I found the solution for the second puzzle at 8:39:41.

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
../IParse/software/MarkDownC Day06.md >day06.c; gcc day06.c -o day06; ./day06
```
