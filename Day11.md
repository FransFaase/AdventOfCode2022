# Day 11 of Advent of Code 2022

Rather complex input. I noticed that there are eight monkeys in my input file.
I am simply going to use that. Lets first define some structure for representing
the state of the monkeys:
```c
#define NR_M 8

struct monkey {
	int nr_items;
	int items[100];
	char oper;
    int arg;
    int div;
    int to_true;
    int to_false;
} monkeys[NR_M];
```

Now lets write some code to read the input and print it out in a condensed form.

```c
void read_input()
{
    FILE *f = fopen("input/day11.txt", "r");
    if (f == 0)
        return;
    for (int i = 0; i < NR_M; i++)
    {
        const char *s;
        char buffer[200];
        fgets(buffer, 200, f);
        monkeys[i].nr_items = 0;
        fgets(buffer, 200, f);
        s = buffer + 18;
        for (;;)
        {
            monkeys[i].items[monkeys[i].nr_items++] = read_int(&s);
            if (*s != ',')
                break;
            s += 2;
        }
        fgets(buffer, 200, f);
        monkeys[i].oper = buffer[23];
        if (buffer[25] == 'o')
            monkeys[i].oper = '^';
        else
        {
            s = buffer + 25;
            monkeys[i].arg = read_int(&s);
        }
        fgets(buffer, 200, f);
        s = buffer + 21;
        monkeys[i].div = read_int(&s);
        fgets(buffer, 200, f);
        s = buffer + 29;
        monkeys[i].to_true = read_int(&s);
        fgets(buffer, 200, f);
        s = buffer + 30;
        monkeys[i].to_false = read_int(&s);
        fgets(buffer, 200, f);
    }
    fclose(f);
}

int main(int argc, char *argv[])
{
    read_input();
    for (int i = 0; i < NR_M; i++)
    {
        printf("%d [", monkeys[i].nr_items);
        for (int j = 0; j < monkeys[i].nr_items; j++)
            printf(" %d", monkeys[i].items[j]);
        printf("] %c %d %d %d %d\n",
            monkeys[i].oper,
            monkeys[i].arg,
            monkeys[i].div,
            monkeys[i].to_true,
            monkeys[i].to_false);
    }
}
```

That looks like it is working. I verified that no monkey is throwing
items to itself. That means we can simple process the list of items
and clear if afterwards. Lets write the code for one monkey throwing
items:
```c
void monkey_throws(int m)
{
    for (int i = 0; i < monkeys[m].nr_items; i++)
    {
        int item = monkeys[m].items[i];
        if (monkeys[m].oper == '+') item = item + monkeys[m].arg;
        if (monkeys[m].oper == '*') item = item * monkeys[m].arg;
        if (monkeys[m].oper == '^') item = item * item;
        int target = (item % monkeys[m].div) == 0 ? monkeys[m].to_true : monkeys[m].to_false;
        item = (item + 1) / 3;
        monkeys[target].items[monkeys[target].nr_items++] = item;
    }
    monkeys[m].nr_items = 0;
}
```

To find the answer, we also need to count the number of times each
monkey threw an item. For this, we have to extend the structure:
```c
struct monkey
{
    ...
    int nr_throws;
};
```

With this we can update the `main` function to find the answer:
```c
int main(int argc, char *argv[])
{
    read_input();

    for (int m = 0; m < NR_M; m++) monkeys[m].nr_throws = 0;
    
    for (int r = 0; r < 20; r++)
        for (int m = 0; m < NR_M; m++)
        {
            monkeys[m].nr_throws += monkeys[m].nr_items;
            monkey_throws(m);
        }
    
    int max = monkeys[0].nr_throws;
    int second_max;
    for (int m = 0; m < NR_M; m++)
    {
        if (monkeys[m].nr_throws >= max)
        {
            second_max = max;
            max = monkeys[m].nr_throws;
        }
        else if (monkeys[m].nr_throws > max)
            second_max = monkeys[m].nr_throws;
    }
    printf("%d\n", max * second_max);
}    
```
The answer returned was not correct. Lets debug with the example input.
I replaced my input file with the example input. Also have to change
the define. (I fixed the problem with respect to redefines in MarkDownC.)
```c
#define NR_M 4
```
It returns the wrong answer. Lets add a `one_round` function that prints
the state after each round. And also adapt `main`:
```c
void one_round()
{
    for (int m = 0; m < NR_M; m++)
    {
        monkeys[m].nr_throws += monkeys[m].nr_items;
        monkey_throws(m);
    }
    for (int m = 0; m < NR_M; m++)
    {
        printf("Monkey %d ", m);
        for (int i = 0; i < monkeys[m].nr_items; i++)
            printf("%c %lld", i == 0 ? ':' : ',', monkeys[m].items[i]);
        printf("\n");
    }
    printf("\n");
}

int main(int argc, char *argv[])
{
    read_input();

    for (int m = 0; m < NR_M; m++) monkeys[m].nr_throws = 0;
    
    for (int r = 0; r < 20; r++)
        one_round();
    
    int max = monkeys[0].nr_throws;
    int second_max;
    for (int m = 0; m < NR_M; m++)
    {
        printf("Monkey %d: %d\n", m, monkeys[m].nr_throws);
        if (monkeys[m].nr_throws >= max)
        {
            second_max = max;
            max = monkeys[m].nr_throws;
        }
        else if (monkeys[m].nr_throws > max)
            second_max = monkeys[m].nr_throws;
    }
    printf("%d\n", max * second_max);
}
```

It seems the problem is in the `monkey_throws` function. Let also add some
debug statements there:
```c
void monkey_throws(int m)
{
    printf("Monkey %d\n", m);
    for (int i = 0; i < monkeys[m].nr_items; i++)
    {
        int item = monkeys[m].items[i];
        printf(" %d", item);
        if (monkeys[m].oper == '+') item = item + monkeys[m].arg;
        if (monkeys[m].oper == '*') item = item * monkeys[m].arg;
        if (monkeys[m].oper == '^') item = item * item;
        printf(" %d", item);
        printf(" %s", (item % monkeys[m].div) == 0 ? "DIV" : "---");
        int target = (item % monkeys[m].div) == 0 ? monkeys[m].to_true : monkeys[m].to_false;
        item = (item + 1) / 3;
        printf(" %d %d\n", item, target);
        monkeys[target].items[monkeys[target].nr_items++] = item;
    }
    monkeys[m].nr_items = 0;
}
```
There seems to be wrong in the division by 3. I missed the word 'down'
in the text 'divided by three and rounded down to the nearest integer'.
I also noticed that the division by three happens before the check on
divisability. Again a case of not reading the puzzle text carefully
enough, Lets fix the function:
```c
void monkey_throws(int m)
{
    printf("Monkey %d\n", m);
    for (int i = 0; i < monkeys[m].nr_items; i++)
    {
        int item = monkeys[m].items[i];
        printf(" %d", item);
        if (monkeys[m].oper == '+') item = item + monkeys[m].arg;
        if (monkeys[m].oper == '*') item = item * monkeys[m].arg;
        if (monkeys[m].oper == '^') item = item * item;
        printf(" %d", item);
        printf(" %s", (item % monkeys[m].div) == 0 ? "DIV" : "---");
        item = item / 3;
        int target = (item % monkeys[m].div) == 0 ? monkeys[m].to_true : monkeys[m].to_false;
        printf(" %d %d\n", item, target);
        monkeys[target].items[monkeys[target].nr_items++] = item;
    }
    monkeys[m].nr_items = 0;
}
```
Now it returns the right answer to the example input. Lets return
to my input after a small change to the program:
```c
#define NR_M 8
```

### Second part

The second part of the puzzle is a bit harder. I guess there must
be a solution using some modular arithmetics using the base consisting
of (the square of) the smallest common multiplier of the division factors.

I continued, after having going on a walk (for almost one hour).
```c

long long divider;

struct monkey {
    int nr_items;
    long long items[100];
    char oper;
    int arg;
    int div;
    int to_true;
    int to_false;
    long nr_throws;
} monkeys[NR_M];

void monkey_throws(int m)
{
    for (int i = 0; i < monkeys[m].nr_items; i++)
    {
        long long item = monkeys[m].items[i];
        if (monkeys[m].oper == '+') item = item + monkeys[m].arg;
        if (monkeys[m].oper == '*') item = item * monkeys[m].arg;
        if (monkeys[m].oper == '^') item = item * item;
        item = item % divider;
        int target = (item % monkeys[m].div) == 0 ? monkeys[m].to_true : monkeys[m].to_false;
        monkeys[target].items[monkeys[target].nr_items++] = item;
    }
    monkeys[m].nr_items = 0;
}

int main(int argc, char *argv[])
{
    read_input();

    for (int m = 0; m < NR_M; m++) monkeys[m].nr_throws = 0;
 
     divider = 1;
     for (int m = 0; m < NR_M; m++) divider *= monkeys[m].div;
     printf("Divider %lld\n", divider);
        
    for (int r = 0; r < 10000; r++)
        one_round();
    
    long max = monkeys[0].nr_throws;
    long second_max;
    for (int m = 0; m < NR_M; m++)
    {
        printf("Monkey %d: %ld\n", m, monkeys[m].nr_throws);
        if (monkeys[m].nr_throws >= max)
        {
            second_max = max;
            max = monkeys[m].nr_throws;
        }
        else if (monkeys[m].nr_throws > max)
            second_max = monkeys[m].nr_throws;
    }
    printf("%ld\n", max * second_max);
}
```

Yes, that did the trick.

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
../IParse/software/MarkDownC Day11.md >day11.c; gcc -Wall day11.c -o day11; ./day11
```
