# Day 23 of Advent of Code 2022

Lets first try to read the example input. I will leave some
space around it.

```c
#define WI 7
#define HI 7
#define STEPS 10
#define W (WI+2*STEPS)
#define H (HI+2*STEPS)

char map[H][W];

#define INPUT_FN "day23.txt"

void read_map()
{
    FILE *f = fopen(INPUT_FN, "r");
    if (f == 0)
        return;
    
	for (int i = 0; i < H; i++)
		for (int j = 0; j < W; j++)
			map[i][j] = ' ';
			
    for (int i = 0; i < HI; i++)
    {
        char buffer[WI+5];
        fgets(buffer, WI+5, f);
        for (int j = 0; j < WI; j++)
            map[i + STEPS][j + STEPS] = buffer[j];
    }
    fclose(f);
}

void print_map()
{
    for (int i = 0; i < H; i++)
    {
        for (int j = 0; j < W; j++)
            printf("%c", map[i][j]);
        printf("\n");
    }
    printf("\n");
}


int main(int argc, char *argv[])
{
    read_map();
    print_map();
}
```

Now lets implement the function for one step:

```c

char proposed[W][H];
char next_map[W][H];

void do_step()
{
	for (int i = 0; i < W; i++)
		for (int j = 0; j < H; j++)
			proposed[i][j] = 0;
	
	for (int i = 0; i < W; i++)
		for (int j = 0; j < H; j++)
			next_map[i][j] = ' ';

	for (int r = 0; r < 2; r++)
	{
		for (int i = 1; i < W-1; i++)
			for (int j = 1; j < H-1; j++)
				if (map[i][j] == '#')
				{
					bool can_N = map[i-1][j-1] != '#' && map[i-1][j] != '#' && map[i-1][j+1] != '#';
					bool can_S = map[i+1][j-1] != '#' && map[i+1][j] != '#' && map[i+1][j+1] != '#';
					bool can_W = map[i-1][j+1] != '#' && map[i][j+1] != '#' && map[i+1][j+1] != '#';
					bool can_E = map[i-1][j-1] != '#' && map[i][j-1] != '#' && map[i+1][j-1] != '#';
					int n_i = i;
					int n_j = j;
					if (can_N || can_S || can_W || can_E)
					{
						if (can_N)
							n_i = i-1;
						else if (can_S)
							n_i = i+1;
						else if (can_W)
							n_j = j+1;
						else if (can_E)
							n_j = j-1;
					}
					if (r == 0)
						proposed[n_i][n_j]++;
					else
					{
						if (proposed[n_i][n_j] == 1)
							next_map[n_i][n_j] = '#';
						else
							next_map[i][j] = '#';
					}
				}
	}
	for (int i = 0; i < W; i++)
		for (int j = 0; j < H; j++)
			map[i][j] = next_map[i][j];
}

int main(int argc, char *argv[])
{
	...
	do_step();
	printf("---\n");
	print_map();
}
```

That looks like to work for one step. Lets now try it for 10 steps:

```c
int main(int argc, char *argv[])
{
    read_map();
    print_map();
    for (int s = 0; s < 10; s++)
    	do_step();
	printf("---\n");
	print_map();
}
```

That does not look correct.

```c
int main(int argc, char *argv[])
{
    read_map();
    print_map();
    for (int s = 0; s < 10; s++)
    {
    	do_step();
		printf("---\n");
		print_map();
	}
}
```




### Second part

After fixing a small mistake, it did return the correct answer.

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
../IParse/software/MarkDownC Day23.md >p.c; gcc -Wall -g p.c -o p && ./p
```
