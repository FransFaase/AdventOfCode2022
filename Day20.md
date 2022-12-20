# Day 20 of Advent of Code 2022

About a complex method of mixing with itself. Lets first implement the
algorithm for the example input:
```c
#define NR 7

void mix(int *input)
{
    int line[NR];
    for (int i = 0; i < NR; i++) line[i] = input[i];
    
    for (int k = 0; k < NR; k++) printf("%2d, ", line[k]); printf("\n\n");

    for (int i = 0; i < NR; i++)
    {
        int m = input[i];
        
        // find location of m in l
        int j = 0;
        while (line[j] != m)
             j++;
        //printf("%2d At %d\n", m, j);
        if (m > 0)
        {
            for (int s = 0; s < m; s++)
            {
                if (j == NR - 1)
                {
                    for (int k = NR - 1; k > 0; k--)
                        line[k] = line[k-1];
                    line[0] = m;
                    j = 0;
                }
                line[j] = line[j + 1];
                line[j + 1] = m;
                j++;
                //for (int k = 0; k < NR; k++) printf("%2d, ", line[k]); printf("\n");
            }
        }
        if (m < 0)
        {
            for (int s = 0; s < -m; s++)
            {
                if (j == 0)
                {
                    for (int k = 0; k < NR - 1; k++)
                        line[k] = line[k+1];
                    line[NR-1] = m;
                    j = NR - 1;
                }
                line[j] = line[j - 1];
                line[j - 1] = m;
                j--;
                //for (int k = 0; k < NR; k++) printf("%2d, ", line[k]); printf("\n");
            }
            if (j == 0)
            {
                for (int k = 0; k < NR - 1; k++)
                    line[k] = line[k+1];
                line[NR-1] = m;
                j = NR - 1;
                //for (int k = 0; k < NR; k++) printf("%2d, ", line[k]); printf("\n");
            }
        }
        //printf("\n");
    }
    
    for (int k = 0; k < NR; k++) printf("%2d, ", line[k]); printf("\n\n");
    int j = 0;
    while (line[j] != 0)
         j++;
    int a1 = line[(j + 1000) % NR];
    int a2 = line[(j + 2000) % NR];
    int a3 = line[(j + 3000) % NR];
    printf("%d %d %d = %d\n", a1, a2, a3, a1 + a2 + a3);
}

void run_example()
{
    int example[NR] = { 1, 2, -3, 3, -2, 0, 4 };
    mix(example);
}

int main(int argc, char *argv[])
{
    run_example();    
}
```
I worked on this in the morning, for about almost two hours, and than got it
working shortly after I started looking at 17:00.

Now lets see, if we get it working for the puzzle input. I still needed to
add some code to print the answer.

I saw that there are 5000 numbers in my input file and also that many of those
numbers are larger 5000. Lets do some experiment to see if we can apply some
modulo operation:
```c
void run_example()
{
    int example[NR] = { 1+6, 2+12, -3-6, 3+6, -2-12, 0, 4+18 };
    mix(example);
}
```

Okay, so you have to use NR-1. I do think we need this for the first part of
the puzzle. Lets try it on the input:
```c
#define NR 5000

void run_input()
{
    int data[NR];
    FILE *f = fopen("input/day20.txt", "r");
    if (f == 0)
        return;
    for (int i = 0; i < NR; i++)
    {
        char buffer[10];
        fgets(buffer, 10, f);
        const char *s = buffer;
        data[i] = read_int(&s);
    }
    fclose(f);
    mix(data);
}

int main(int argc, char *argv[])
{
    run_input();    
}
```

That does not return the correct answer.

Okay, I see why it does not work, some number occur more than once,
and the search always finds the first. It looks like we also have to
keep track of where the numbers are. The 0 occures only once.

```c

typedef struct index_pair index_pair_t;
struct index_pair
{
    int i;
    int v;
};

void mix(int *input)
{
    index_pair_t line[NR];
    for (int i = 0; i < NR; i++) { line[i].i = i; line[i].v = input[i]; }
    
    //for (int k = 0; k < NR; k++) printf("%2d, ", line[k]); printf("\n\n");

    for (int i = 0; i < NR; i++)
    {
        // find location of m in l
        int j = 0;
        while (line[j].i != i)
             j++;
        index_pair_t m = line[j];
        
        //printf("%2d At %d\n", m, j);
        if (m.v > 0)
        {
            for (int s = 0; s < m.v; s++)
            {
                if (j == NR - 1)
                {
                    for (int k = NR - 1; k > 0; k--)
                        line[k] = line[k-1];
                    line[0] = m;
                    j = 0;
                }
                line[j] = line[j + 1];
                line[j + 1] = m;
                j++;
                //for (int k = 0; k < NR; k++) printf("%2d, ", line[k]); printf("\n");
            }
        }
        if (m.v < 0)
        {
            for (int s = 0; s < -m.v; s++)
            {
                if (j == 0)
                {
                    for (int k = 0; k < NR - 1; k++)
                        line[k] = line[k+1];
                    line[NR-1] = m;
                    j = NR - 1;
                }
                line[j] = line[j - 1];
                line[j - 1] = m;
                j--;
                //for (int k = 0; k < NR; k++) printf("%2d, ", line[k]); printf("\n");
            }
            if (j == 0)
            {
                for (int k = 0; k < NR - 1; k++)
                    line[k] = line[k+1];
                line[NR-1] = m;
                j = NR - 1;
                //for (int k = 0; k < NR; k++) printf("%2d, ", line[k]); printf("\n");
            }
        }
        //printf("\n");
    }
    
    //for (int k = 0; k < NR; k++) printf("%2d, ", line[k]); printf("\n\n");
    int j = 0;
    while (line[j].v != 0)
         j++;
    int a1 = line[(j + 1000) % NR].v;
    int a2 = line[(j + 2000) % NR].v;
    int a3 = line[(j + 3000) % NR].v;
    printf("%d %d %d = %d\n", a1, a2, a3, a1 + a2 + a3);
}
```
Okay, that did give the right answer.

### Second part

```c
typedef struct index_pair2 index_pair2_t;
struct index_pair2
{
    int i;
    num_t v;
};

void mix2(num_t *input)
{
    index_pair2_t line[NR];
    for (int i = 0; i < NR; i++) { line[i].i = i; line[i].v = input[i] * 811589153; }
    
    //for (int k = 0; k < NR; k++) printf("%lld, ", line[k].v); printf("\n\n");

    for (int t = 0; t < 10; t++)
    {
        for (int i = 0; i < NR; i++)
        {
            // find location of m in l
            int j = 0;
            while (line[j].i != i)
                 j++;
            index_pair2_t m = line[j];
            
            //printf("%2d At %d\n", m, j);
            if (m.v > 0)
            {
                int m_v = (int)(m.v % (NR-1));
                for (int s = 0; s < m_v; s++)
                {
                    if (j == NR - 1)
                    {
                        for (int k = NR - 1; k > 0; k--)
                            line[k] = line[k-1];
                        line[0] = m;
                        j = 0;
                    }
                    line[j] = line[j + 1];
                    line[j + 1] = m;
                    j++;
                    //for (int k = 0; k < NR; k++) printf("%2d, ", line[k]); printf("\n");
                }
            }
            if (m.v < 0)
            {
                int m_v = (int)(-m.v % (NR-1));
                for (int s = 0; s < m_v; s++)
                {
                    if (j == 0)
                    {
                        for (int k = 0; k < NR - 1; k++)
                            line[k] = line[k+1];
                        line[NR-1] = m;
                        j = NR - 1;
                    }
                    line[j] = line[j - 1];
                    line[j - 1] = m;
                    j--;
                    //for (int k = 0; k < NR; k++) printf("%2d, ", line[k]); printf("\n");
                }
                if (j == 0)
                {
                    for (int k = 0; k < NR - 1; k++)
                        line[k] = line[k+1];
                    line[NR-1] = m;
                    j = NR - 1;
                    //for (int k = 0; k < NR; k++) printf("%2d, ", line[k]); printf("\n");
                }
            }
            //printf("\n");
        }
    
        for (int k = 0; k < NR; k++) printf("%lld, ", line[k].v); printf("\n\n");
    }
    
    int j = 0;
    while (line[j].v != 0)
         j++;
    num_t a1 = line[(j + 1000) % NR].v;
    num_t a2 = line[(j + 2000) % NR].v;
    num_t a3 = line[(j + 3000) % NR].v;
    printf("%lld %lld %lld = %lld\n", a1, a2, a3, a1 + a2 + a3);
}

#define NR 7

int main(int argc, char *argv[])
{
    num_t example[NR] = { 1, 2, -3, 3, -2, 0, 4 };
    mix2(example);
}
```
That produces the same result as the example.

```c
#define NR 5000

int main(int argc, char *argv[])
{
    num_t data[NR];
    FILE *f = fopen("input/day20.txt", "r");
    if (f == 0)
        return 0;
    for (int i = 0; i < NR; i++)
    {
        char buffer[10];
        fgets(buffer, 10, f);
        const char *s = buffer;
        data[i] = read_int(&s);
    }
    fclose(f);
    mix2(data);
}
```
This returned the answer.


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
./IParse/software/MarkDownC Day20.md >p.c; gcc -Wall -g p.c -o p && ./p
```
