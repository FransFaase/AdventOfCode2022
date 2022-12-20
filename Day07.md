# Day 7 of Advent of Code 2022

About recursive scanning directories. I assumed that every
directory was just visited once. I checked that the `cd /` command
was only given once at the start of the file, thsus could be
ignored. I decided to go for a speed-run in attempt to solve
the puzzle in the limited time I had before I had to go to work.

Lets start with main.

```c
int main(int argc, char *argv[])
{
    FILE *f = fopen("input/day07.txt", "r");
    if (f == 0)
        return 0;
    scan(f);
}
```

Now implement the scan function:
```c
long long answer1 = 0;

long long scan(FILE *f)
{
    long long size = 0;
    for (;;)
    {
        char buffer[100];
        if (!fgets(buffer, 100, f))
            break;
        printf("%s", buffer);
        if (strncmp(buffer, "$ cd /", 6) == 0)
        {
        }
        else if (strncmp(buffer, "$ cd ..", 7) == 0)
            break;
        else if (strncmp(buffer, "$ cd ", 5) == 0)
        {
            size += scan(f);
        }
        else if (strncmp(buffer, "$ ls", 4) == 0)
        {
        }
        else
        {
            const char *s = buffer;
            size += read_int(&s);
            printf("## = %lld\n", size);
        }
    }
    if (size <= 100000)
        answer1 += size;
    return size;
}

int main(int argc, char *argv[])
{
    ...
    printf("%lld\n", answer1);
}
```

With this, I did find the solution to the first part of the
puzzle. I read the second part and realized that I have to
rewrite most of the above code, if I want to have a program
that finds the solution for the first and the second puzzle.
I think about reading the input file twice and just a function
pointer to specific functions for the first and second part
of the puzzle. Lets first write that functions:
```c

long long scan_dirs(FILE *f, void (*process)(long long dir_size))
{
    long long size = 0;
    for (;;)
    {
        char buffer[100];
        if (!fgets(buffer, 100, f))
            break;
        if (strncmp(buffer, "$ cd /", 6) == 0)
        {
        }
        else if (strncmp(buffer, "$ cd ..", 7) == 0)
            break;
        else if (strncmp(buffer, "$ cd ", 5) == 0)
        {
            size += scan_dirs(f, process);
        }
        else if (strncmp(buffer, "$ ls", 4) == 0)
        {
        }
        else
        {
            const char *s = buffer;
            size += read_int(&s);
        }
    }
    process(size);
    return size;
}
```

Now write the funtion to process the whole input file and
also return the total size of all directories, because that
is needed for the second part of the puzzle:
```c
long long read_input(void (*process)(long long dir_size))
{
    long long total_size = 0;
    FILE *f = fopen("input/day07.txt", "r");
    if (f != 0)
    {
        total_size = scan_dirs(f, process);
        fclose(f);
    }
    return total_size;
}
```

Now create the function for the first part:
```c

void process1(long long size)
{
    if (size <= 100000)
        answer1 += size;
}

int main(int argc, char *argv[])
{
    long long total_size = read_input(process1);
    printf("%lld\n", answer1);
    printf("%lld\n", total_size);
}
```

This does return the same answer as before. The space
that needs to be deleted, is at least the total size
minus 40000000. And then we have to find the smallest
directory equal or larger than that value. The smallest
is smaller than the total size. So lets use that to
initialize for the answer for the second part.

```c
long long answer2;
long long needed;

void process2(long long size)
{
    if (size >= needed && size < answer2)
        answer2 = size;
}

int main(int argc, char *argv[])
{
    ...
    answer2 = total_size;
    needed = total_size - 40000000;
    printf("Needed: %lld\n", needed);
    read_input(process2);
    
    printf("%lld\n", answer2);
}
```

With this, I found the answer to the second part of
the puzzle.

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
../IParse/software/MarkDownC Day07.md >day07.c; gcc day07.c -o day07; ./day07
```
