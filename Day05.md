# Day 5 of Advent of Code 2022

I started around 18:45

This puzzle is about stacks and moving boxes from one stack to the
ohter. In the input file the begin state of the stacks is given
from top to bottom. In [my input file](input/day05.txt) there are
nine stacks with at most eight boxes. That is a total of at most
72 boxes. I will use strings to represent a stack from bottom to
top, because it will allow me to use `strcpy` to move boxes from
one stack to the other. I think it is also a good idea to keep
the height of the stacks. Some constants and declarations:
```c
#define NR_STACKS 9
#define INIT_HEIGHT 8
struct
{
    char boxes[NR_STACKS * INIT_HEIGHT + 1];
    int height;
} stacks[NR_STACKS];
```

Lets write some code to read the initial configuration and some code
to test it.
```c

void read_initial_state(FILE *f)
{
    for (int s = 0; s < NR_STACKS; s++)
    {
        stacks[s].height = INIT_HEIGHT;
        stacks[s].boxes[INIT_HEIGHT] = '\0';
    }
        
    
    char buffer[100];
    for (int h = INIT_HEIGHT - 1; h >= 0; h--)
    {
        fgets(buffer, 100, f);
        for (int s = 0; s < NR_STACKS; s++)
        {
            char b = buffer[1 + 4*s];
            if (b == ' ')
            {
                stacks[s].height = h;
                stacks[s].boxes[h] = '\0';
            }
            else
                stacks[s].boxes[h] = b;
        }
    }
    fgets(buffer, 100, f);
    fgets(buffer, 100, f);
}

void print_state()
{
    for (int s = 0; s < NR_STACKS; s++)
        printf("%d %s\n", stacks[s].height, stacks[s].boxes);
    printf("\n");
}

int main(int argc, char *argv[])
{
    FILE *f = fopen("input/day05.txt", "r");
    if (f == 0)
        return 0;
    read_initial_state(f);
    
    
    print_state();

    return 0;
}
```

Looks okay. Now lets, write a function to read the moves.

```c
void read_moves(FILE *f)
{
    char buffer[100];
    while (fgets(buffer, 100, f))
    {
        const char* s = buffer;
        s += 5;
        int nr = read_int(&s);
        s += 6;
        int from = read_int(&s);
        s += 4;
        int to = read_int(&s);
        
        move(nr, from, to);
    }
}

void move(int nr, int from, int to)
{
    printf("move %d from %d to %d\n", nr, from, to);
}

int main(int argc, char *argv[])
{
    FILE *f = fopen("input/day05.txt", "r");
    if (f == 0)
        return 0;
    read_initial_state(f);
    
    read_moves(f);
    
    return 0;
}

```

This seems to work. Lets write some code to perform the moving
operations.
```c
void move(int nr, int from, int to)
{
    from--;
    to--;
    if (stacks[from].height < nr)
    {
        printf("ERROR: stack %d has %d, need %d", from, stacks[from].height, nr);
        return;
    }
    stacks[from].height -= nr;
    strcpy(stacks[to].boxes + stacks[to].height, stacks[from].boxes + stacks[from].height);
    stacks[from].boxes[stacks[from].height] = '\0';
    stacks[to].height += nr;

    print_state();
}
```

And expand the `main` function with some code to write the answer:
```c
int main(int argc, char *argv[])
{
    FILE *f = fopen("input/day05.txt", "r");
    if (f == 0)
        return 0;
    read_initial_state(f);
    
    read_moves(f);

    printf("\n");
    for (int s = 0; s < NR_STACKS; s++)
        printf("%c", stacks[s].boxes[stacks[s].height-1]);
    printf("\n");    
    return 0;
}
```

This is not the correct solution. I did not read the puzzle correctly.
The boxes are reversed. A new implementation for the `move` function is:
```c
void move(int nr, int from, int to)
{
    from--;
    to--;
    if (stacks[from].height < nr)
    {
        printf("ERROR: stack %d has %d, need %d", from, stacks[from].height, nr);
        return;
    }
    for (int i = 0; i < nr; i++)
        stacks[to].boxes[stacks[to].height++] = stacks[from].boxes[--stacks[from].height];
    stacks[from].boxes[stacks[from].height] = '\0';
    stacks[to].boxes[stacks[to].height] = '\0';
}
```

That resulted in the correct answer.

### Second puzzle

Okay. The second puzzle is keeping the boxes in the same order. I already had
found that solution, so, I justed looked it up.


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
../IParse/software/MarkDownC Day05.md >day05.c; gcc day05.c -o day05; ./day05
```
