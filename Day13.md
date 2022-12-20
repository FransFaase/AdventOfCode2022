# Day 13 of Advent of Code 2022

I started around 9:17 to look at the puzzle. After having read the
description, I wonder wether it is necessary to read the input into
a tree data structure, or whether you can just parse the strings
without having to convert them to a tree first. Because the tree
is always parsed from the start to the end, we could also make use
of an iterator that just walks over the tree. Maybe that is the
approach I am going to try. Maybe I should first write the compare
function assuming that the lists match:
```c
int compare(iter_t *t1, iter_t *t2)
{
    //printf("compare:\n");
    //iter_print(t1);
    //iter_print(t2);
    if (t1->is_int && t2->is_int)
        return t1->val - t2->val;
    open_list(t1);
    open_list(t2);
    for (;;)
    {
        if (!t1->more || !t2->more)
        {
            if (!t1->more && t2->more)
                return -1;
            if (!t2->more && t1->more)
                return 1;
            break;
        }
        int c = compare(t1, t2);
        if (c != 0)
            return c;
        read_next(t1);
        read_next(t2);
    }
    close_list(t1);
    close_list(t2);
    return 0;
}
```

Now we are left with the task for writing the missing functions:
```c
void open_list(iter_t *t)
{
    //printf("open_list %s\n", t->s);
    if (t->is_int)
    {
        t->fake_list++;
        t->more = TRUE;
    }
    else
    {
        t->s++; // read '['
        t->more = *t->s != ']';
        if (t->more)
        {
            t->is_int = '0' <= *t->s && *t->s <= '9';
            if (t->is_int)
                t->val = read_int(&t->s);
        }
    }
}

void read_next(iter_t *t)
{
    //printf("read_next %s\n", t->s);
    if (t->fake_list > 0)
        t->more = FALSE;
    else
    {
        t->more = *t->s == ',';
        if (t->more)
        {
            t->s++;
            t->is_int = '0' <= *t->s && *t->s <= '9';
            if (t->is_int)
                t->val = read_int(&t->s);
        }
    }
}

void close_list(iter_t *t)
{
    if (t->fake_list > 0)
    {
        t->fake_list--;
    }
    else
    {
        for (int d = 1; d > 0; t->s++)
        {
            if (*t->s == ']')
                d--;
            if (*t->s == '[')
                d++;
        }
        //printf("%s\n", t->s);
    }
}

void init_iter(iter_t *t, const char *s)
{
    t->s = s;
    t->is_int = FALSE;
    t->fake_list = 0;
}
    
typedef struct iter iter_t;
struct iter
{
    const char *s;
    bool is_int;
    int val;
    bool more;
    int fake_list;
};
```

Now, lets implement some code to check this with the example input.

```c

int ind = 1;
int sum_index = 0;

bool smaller(const char *s1, const char *s2)
{
    iter_t i1;
    init_iter(&i1, s1);
    iter_t i2;
    init_iter(&i2, s2);
    int c = compare(&i1, &i2);
    printf("%s\n%s\n%d %s %d\n\n", s1, s2, c, c < 0 ? "OK" : "NOT OK", ind);
    if (c < 0)
        sum_index += ind;
    ind++;
    return c;
}

void tests()
{
    smaller("[1,1,3,1,1]", "[1,1,5,1,1]");
    smaller("[[1],[2,3,4]]", "[[1],4]");
    smaller("[9]", "[[8,7,6]]");
    smaller("[[4,4],4,4]", "[[4,4],4,4,4]");
    smaller("[7,7,7,7]", "[7,7,7]");
    smaller("[]", "[3]");
    smaller("[[[]]]", "[[]]");
    smaller("[1,[2,[3,[4,[5,6,7]]]],8,9]", "[1,[2,[3,[4,[5,6,0]]]],8,9]");
    printf("Answer %d\n", sum_index);
}

int main(int argc, char* argv[])
{
    tests();
}
```

Not sure, if this works as expected. I added temporary added some debug
code to the above using the function below. It work correctly and I placed
the code in comments.

```c
void iter_print(iter_t *it)
{
    printf("  ");
    if (it->is_int)
        printf("%d", it->val);
    if (it->fake_list > 0)
        printf(" (%d)", it->fake_list);
    printf(" %s\n", it->s);
}
```

Now lets try to process the input.
```c
void process_input()
{
    FILE *f = fopen("input/day13.txt", "r");
    if (f == 0)
        return;
    char buffer1[2000];
    char buffer2[2000];
    while (fgets(buffer1, 2000, f))
    {
        while (buffer1[strlen(buffer1)-2] < ' ') buffer1[strlen(buffer1)-2] = '\0';
        fgets(buffer2, 2000, f);
        while (buffer2[strlen(buffer2)-2] < ' ') buffer2[strlen(buffer2)-2] = '\0';
        smaller(buffer1, buffer2);
        fgets(buffer2, 2000, f);
    }
    printf("Answer %d\n", sum_index);
}

int main(int argc, char* argv[])
{
    process_input();
}
```
The answer is not because I the code assumes that the input lines where at most
200 characters long.

### Second part

For the second part all the input lines have to be sorted. My input has 300 lines
with data. Lets reads them into an array of strings.
```c
void process_input2()
{
    char *key1 = "[[2]]";
    char *key2 = "[[6]]";
    
    char *lines[302];
    int n = 0;
    
    lines[n++] = key1;
    lines[n++] = key2;

    FILE *f = fopen("input/day13.txt", "r");
    if (f == 0)
        return;
    char buffer[2000];
    while (fgets(buffer, 2000, f))
    {
        int len = strlen(buffer);
        while (len > 0 && buffer[len-1] < ' ') buffer[--len] = '\0';
        if (len > 0)
        {
            lines[n] = (char *)malloc(len+1);
            strcpy(lines[n], buffer);
            n++;
        }
    }
    printf("n = %d\n", n);
    qsort(lines, n, sizeof(char_p), comp);
    
    
    for (int i = 0; i < n; i++)
        printf("%s\n", lines[i]);

    printf("Done\n");
    
    // find the keys;
    int key1_pos;
    int key2_pos;
    for (int i = 0; i < n; i++)
        if (lines[i] == key1)
            key1_pos = i+1;    
        else if (lines[i] == key2)
            key2_pos = i+1;
    
    printf("Answer %d x %d = %d\n", key1_pos, key2_pos, key1_pos * key2_pos);    
}

typedef char * char_p; // The syntax for sizeof in MarkDownC only accepts a single identifier 

int comp(const void* elem1, const void* elem2)
{
    //printf("compare %s %s\n", *((const char**)elem1), *((const char**)elem2));
    iter_t i1;
    iter_t i2;
    init_iter(&i1, *((const char**)elem1));
    init_iter(&i2, *((const char**)elem2));
    return compare(&i1, &i2);
}


int main(int argc, char* argv[])
{
    process_input2();
}
```
It took me some time to get everything round the use of `qsort` working
correctly, before I found the answer to the second part of the puzzle.

### Some standard definitions

```c
#include "stdlib.h"
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
../IParse/software/MarkDownC Day13.md >day13.c; gcc -Wall day13.c -o day13; ./day13
```
