# Day 1 of Advent of Code 2022

I read the puzzle in the morning, but only started writing this at 19:27.

The input consists of pairs of letters, separated by a space on lines. These
have to be processed. Lets first write the code to parse the input:
```c
void read_input()
{
    FILE *f = fopen("input/day02.txt","r");
    if (f == 0)
        return;
    char buffer[10];
    while (fgets(buffer, 10, f))
    {
        process(buffer[0], buffer[2]);
    }
}

void process(char a, char b)
{
    printf("%c %c\n", a, b);
}

int main(int argc, char *argv[])
{
    read_input();
}
```

The game is about [Rock Paper Scissors](https://en.wikipedia.org/wiki/Rock_paper_scissors)
where the different play options are represented by the letter 'A', 'B', and 'C'.
'A' defeats 'C', 'C' defeats 'B', and 'B' defeats 'A'. This can be implemented
in the following function:
```c
bool defeats(char a, char b)
{
    return a == 'A' && b == 'C' || a == 'C' && b == 'B' || a == 'B' && b == 'A';
}
```

The letter 'X', 'Y', and 'Z' are mapped on 'A', 'B', and 'C'. A function, which
implements this is:
```c
char map(char a)
{
    return "ABC"[a - 'X'];
}
```

With this we can implement the points of a round.
```c
int score_round(char a, char b)
{
    int score = 1 + b - 'X';
    char tb = map(b);
    return score + (defeats(a, tb) ? 0 : defeats(tb, a) ? 6 : 3);
}
```

These have to be summed:
```c
long sum;

void process(char a, char b)
{
    sum += score_round(a, b);
}

int main(int argc, char *argv[])
{
    sum = 0;
    read_input();
    printf("%ld\n", sum);
}
```

At 20:03, after some debugging, I found the answer for the first puzzle.
The second part of the puzzle is a little different than I had
expected. We need a function that to adjust what you need to play.
```c
int score_round2(char a, char b)
{
    int score = 0;
    char play;
    if (b == 'X')
    {
        play = "CAB"[a - 'A'];
    }
    else if (b == 'Y')
    {
        score = 3;
        play = "ABC"[a - 'A'];
    }
    else
    {
        score = 6;
        play = "BCA"[a - 'A'];
    }
    return score + play - 'A' + 1;
}

long sum2;

void process(char a, char b)
{
    sum += score_round(a, b);
    sum2 += score_round2(a, b);
}

int main(int argc, char *argv[])
{
    sum = 0;
    sum2 = 0;
    read_input();
    printf("%ld\n", sum);
    printf("%ld\n", sum2);
}
```

Around 20:25, I found the solution for the second puzzle.

### Some standard definitions

```c
typedef int bool;
#define FALSE 0
#define TRUE 1
```

The command I use to process this markdown file, is:
```
../IParse/software/MarkDownC Day02.md >day02.c; gcc day02.c -o day02; ./day02
```
