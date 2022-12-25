# Day 25 of Advent of Code 2022

Really easy if you have written all kinds of number parsing and
printing functions in your life before. parsing a SNAFU is
rather straight forward. Just multiply your previous result with
the base and add the value of the charater.

```c
num_t parse_snafu(const char *s)
{
    num_t result = 0;
    for (;; s++)
        if ('0' <= *s && *s <= '2')
            result = 5 * result + *s - '0';
        else if (*s == '-')
            result = 5 * result - 1;
        else if (*s == '=')
            result = 5 * result - 2;
        else
            break;
    return result;
}
```

Printing a SNAFU is a little more tricky, because you have to know
the multiplier for the largest digit if you want to print it from
the start till the end. And there are some more tricky things. But
if you print the characters in reverse, from last till the first,
than it is not so complicated. Only have to realize you have to
add one to the remaining value for the more significant digits if
you print a negative digit.

```c
void print_snafu(num_t value)
{
    char buffer[30];
    buffer[29] = '\0';
    int p = 29;
    while (value != 0)
    {
        int digit = value % 5;
        value = value / 5;
        if (digit <= 2)
            buffer[p--] = digit + '0';
        else
        {
            buffer[p--] = digit == 3 ? '=' : '-';
            value++;
        }
    }
    printf("%s (%lld)", buffer + p + 1, parse_snafu(buffer + p + 1));
    
}

int main(int argc, char *argv[])
{
    print_snafu(4890);
    printf("\n");
}
```

I only tested the print method. I made three small errors in the code.
The first time I tested it, it printed nothing. That was because the
last statement of the function read:
```
    printf("%s (%lld)", buffer + p);
```

Then it printed `2==1=0`, because the first statement in the else-branch
read like:
```
            buffer[p--] = digit == '3' ? '-' : '=';
```
After, I removed the quotes around the '3', it printed `2-=1-0`, which made
me realize, I had the '-' and '=' characters swapped. 3 = 5 - 2.

Next, I just wrote the code to process my input file.

```c
int main(int argc, char *argv[])
{
    FILE *f = fopen("input/day25.txt", "r");
    char buffer[30];
    num_t sum = 0;
    while (fgets(buffer, 29, f))
    {
        sum += parse_snafu(buffer);
    }
    print_snafu(sum);
    printf("\n");
}
```

When I ran it, it returned the correct answer.

(I added the text afterwards.)

### Post script

When I had finished writing this and closed my laptop, I realize there
was a bug in `print_snafy` in the sence that it might not work correctly
for any compiler and/or processor. The correct code is:
```c
void print_snafu(num_t value)
{
    char buffer[30];
    buffer[29] = '\0';
    int p = 29;
    while (value != 0)
    {
        int digit = value % 5;
        value = value / 5;
        if (digit <= 2)
            buffer[--p] = digit + '0';
        else
        {
            buffer[--p] = digit == 3 ? '=' : '-';
            value++;
        }
    }
    printf("%s (%lld)", buffer + p, parse_snafu(buffer + p));
    
}
```

### Some standard definitions

The only one used:
```c
typedef int long long num_t;
```


The command I use to process this markdown file, is:
```
../IParse/software/MarkDownC Day25.md >p.c; gcc -Wall -g p.c -o p && ./p
```
