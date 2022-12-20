# Day 10 of Advent of Code 2022

I opened my laptop just after 10 in the morning. I read the
puzzle description. Lets first write the function reading
the input.

```c
void read_input()
{
    FILE *f = fopen("input/day10.txt", "r");
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
    if (strncmp(line, "noop", 4) == 0)
        process_noop();
    if (strncmp(line, "addx ", 5) == 0)
    {
        const char *s = line + 5;
        int v = read_int(&s);
        process_addx(v);
    }
}

void process_noop()
{
    printf("noop\n");
}

void process_addx(int v)
{
    printf("addx %d\n", v);
}

int main(int argc, char *argv[])
{
    read_input();
}
```
Okay, that echoes the input fine. Now, implement the `process_`
function and introduce an `inspect` function that is called on
each cycle.

```c
integer_t x = 1;
integer_t cycle = 1;

void process_noop()
{
    cycle++;
    inspect();
}

void process_addx(int v)
{
    cycle++;
    inspect();
    cycle++;
    x += v;
    inspect();
}

integer_t answer1 = 0;

void inspect()
{
    if ((cycle + 20) % 40 == 0 && cycle <= 220)
        answer1 += cycle * x;
}    

int main(int argc, char *argv[])
{
    ...
    printf("%lld\n", answer1);
}
```
With a little tweaking, I had to read the text again to
realize that `x` and `cycle` should both be initialized to 1.

### Second half of the puzzle.

Now that is bit more complicated. I leave that for later.

At 20:46, I started looking at this again. I am attempting
to do this in one try. There are a lot of possible one-off
errors that could be made in this part of the puzzle. That
is one reason why I decided to leave it for later this morning.
I often still need pen and paper to figure out these kind of
details.

If I understand it correctly, the `x` register represents the
position of a sprite. It takes up positions `x+1`, `x, and `x+1`.
Lets introduce a variable `v_pos` that represents the vertical
position. It starts at 1 and whenever it reaches 40 (at the end
of a cycle), it is reset again to 1. So, a pixel is visible
when 'x-1 <= v_pos && v_pos <= x+1'. Let extend the `inspect`
function:
```c
int v_pos = 1;

void inspect()
{
    ...
    printf("%c", x-1 <= v_pos && v_pos <= x+1 ? '#' : '.');
    v_pos++;
    if (v_pos > 40)
    {
        v_pos = 1;
        printf("\n");
    }
}
```
That does not look very complicated. I ran it and did get some
interesting looking output, that looked like eight capital letters,
and indeed proved to be the answer to the second part of the puzzle.


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
../IParse/software/MarkDownC Day10.md >day10.c; gcc day10.c -o day10; ./day10
```
