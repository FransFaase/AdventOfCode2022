# Day 16 of Advent of Code 2022

I have no idea what is the best way to solve this puzzle yet.
But lets first write some code to read the input and store it
in some graph structure.
```c
typedef struct valve valve_t;
struct valve
{
    char name[3];
    int rate;
    valve_t *to[6];
    int nr_to;
};
valve_t valves[60];
int nr_valves = 0;

valve_t *find_valve(const char *name)
{
    for (int i = 0; i < nr_valves; i++)
        if (name[0] == valves[i].name[0] && name[1] == valves[i].name[1])
            return &valves[i];
    valve_t *valve = &valves[nr_valves++];
    valve->name[0] = name[0];
    valve->name[1] = name[1];
    valve->name[2] = '\0';
    valve->nr_to = 0;
    return valve;
}

void read_input()
{
    FILE *f = fopen("input/day16.txt", "r");
    if (f == 0)
        return;
    char buffer[200];
    while (fgets(buffer, 200, f))
    {
        const char *s = buffer + strlen("Valve ");
        valve_t *valve = find_valve(s);
        s += strlen("__ has flow rate=");
        valve->rate = read_int(&s);
        s += strlen("; tunnels lead to valve");
        if (*s == 's') s++;
        s++;
        for (;;)
        {
            valve->to[valve->nr_to++] = find_valve(s);
            s += 2;
            if (*s != ',')
                break;
            s += 2;
        }
    }
    fclose(f);
}

int main(int argc, char *argv[])
{
    read_input();
    for (int i = 0; i < nr_valves; i++)
    {
        printf("%2d %s %d %2d: ", i, valves[i].name, valves[i].rate, valves[i].nr_to);
        for (int j = 0; j < valves[i].nr_to; j++)
            printf(" %s", valves[i].to[j]->name);
        printf("\n");
    }
}
```
At 8:30, I succeeded in reading the input with the above code.

Lets first try a recursive approach:
```c

struct valve
{
    ...
    bool open;
};

int main(int argc, char *argv[])
{
    ...
    for (int i = 0; i < nr_valves; i++)
        valves[i].open = FALSE;    
    search(find_valve("AA"), 0, 0, 0);
    printf("\n\n%ld\n", max_released);
}

long max_released = 0;

void search(valve_t *valve, int minute, long rate, long released)
{
    if (minute == 30)
    {
        printf(" %ld", released);
        if (released > max_released)
            max_released = released;
        return;
    }
    released += rate;
    
    minute += 2;
    for (int i = 0; i < valve->nr_to; i++)
    {
        valve_t *to = valve->to[i];
        if (to->open)
        {
            search(to, minute, rate, released + rate);
        }
        else
        {
            to->open = TRUE;
            search(to, minute, rate + to->rate, released + rate + to->rate);
            to->open = FALSE;
        }
    }
}
```
It looks like it is doable to use this approach. With my input, I did not
get the right answer. It now 9:01.

I now see why. If you do not open a valve, it takes no time. I think it
is also a good idea to remeber the number of valves you have opened,
because once all are open, there is no need to walk again.
```c

int nr_closed = 0;

int main(int argc, char *argv[])
{
    ...
    for (int i = 0; i < nr_valves; i++)
    {
        valves[i].open = FALSE;
        if (valves[i].rate > 0)
            nr_closed++;
    }    
    search(find_valve("AA"), 0, 0, 0);
    printf("\n\n%ld\n", max_released);
}

void search(valve_t *valve, int minute, long rate, long released)
{
    released += rate;
    if (minute == 30)
    {
        if (released > max_released)
        {
            printf(" %ld\n", released);
            max_released = released;
        }
        return;
    }
    minute++;
    if (nr_closed == 0)
        search(valve, minute, rate, released);
    else if (valve->rate > 0 && !valve->open)
    {
        valve->open = TRUE;
        nr_closed--;
        search(valve, minute, rate + valve->rate, released);
        nr_closed++;
        valve->open = FALSE;
    }
    else
    {
        for (int i = 0; i < valve->nr_to; i++)
            search(valve->to[i], minute, rate, released);
    }
}
```

That did not return an answer within acceptable time. I thought about
it during the day. I am going to try to calculate the shortest distances
between the valves and only visit valves that can still be opened.
```c
#define NR 59
#define MINUTES 30
int dist[NR][NR];

void calc_dist()
{
    for (int i = 0; i < NR; i++)
        for (int j = 0; j < NR; j++)
            dist[i][j] = i == j ? 0 : 1000;
            
    for (int i = 0; i < NR; i++)
        for (int j = 0; j < valves[i].nr_to; j++)
        {
            int k = valves[i].to[j] - valves;
            dist[i][k] = 1;
        }

    // Is it symmetric
    bool sym = TRUE;
    for (int i = 0; i < NR-1; i++)
        for (int j = i+1; j < NR; j++)
            if (dist[i][j] != dist[j][i])
                sym = FALSE;
    
    printf("%s\n", sym ? "Symmetric" : "Directional");
    
    for (int j = 0; j < NR; j++)
        for (int i = 0; i < NR; i++)
            for (int k = 0; k < NR; k++)
                if (dist[i][j] + dist[j][k] < dist[i][k])
                    dist[i][k] = dist[i][j] + dist[j][k];
}

void search2(int v, int minute, long rate, long released)
{
    if (minute == MINUTES)
    {
        if (released > max_released)
        {
            printf(" %ld\n", released);
            max_released = released;
        }
        return;
    }

    valves[v].open = TRUE;
    rate += valves[v].rate;
    bool some = FALSE;
    for (int nv = 0; nv < NR; nv++)
        if (valves[nv].rate > 0 && !valves[nv].open && nv != v)
        {
            int d = dist[v][nv] + 1;
            if (minute + d <= MINUTES)
            {
                search2(nv, minute + d, rate, released + rate * d);
                some = TRUE;
            }
        }
    if (!some)
        search2(v, MINUTES, rate, released + rate * (MINUTES - minute));
    valves[v].open = FALSE;
}

int main(int argc, char *argv[])
{
    read_input();
    for (int i = 0; i < nr_valves; i++)
    {
        valves[i].open = FALSE;
        if (valves[i].rate > 0)
            nr_closed++;
    }
    calc_dist();
    search2(find_valve("AA") - valves, 0, 0, 0);
    printf("\n\n%ld\n", max_released);
}
```
That indeed gives the necessary speed-up and found the solution for my input.

### Second part

Okay, now we have to search with two positions. Lets rewrite the above
search a bit.
```c
void search3(int v1, int steps1, int minute, long rate, long released)
{
    released += rate;
    if (minute == MINUTES)
    {
        if (released > max_released)
        {
            printf(" %ld\n", released);
            max_released = released;
        }
        return;
    }
    
    minute++;
    
    if (steps1 > 0)
    {
        int rate1 = rate;
        search3(v1, steps1 - 1, minute, rate1, released );
    }
    else
    {
        valves[v1].open = TRUE;
        int rate1 = rate + valves[v1].rate;
        bool some = FALSE;
        for (int nv1 = 0; nv1 < NR; nv1++)
            if (valves[nv1].rate > 0 && !valves[nv1].open && nv1 != v1)
            {
                int d1 = dist[v1][nv1];
                if (minute + d1 <= MINUTES)
                {
                    search3(nv1, d1, minute, rate1, released);
                    some = TRUE;
                }
            }
        if (!some)
            search3(v1, 0, minute, rate, released);
        valves[v1].open = FALSE;
    }
}

int main(int argc, char *argv[])
{
    read_input();
    for (int i = 0; i < nr_valves; i++)
    {
        valves[i].open = FALSE;
        if (valves[i].rate > 0)
            nr_closed++;
    }
    calc_dist();
    search3(find_valve("AA") - valves, 0, 0, 0, 0);
    printf("\n\n%ld\n", max_released);
}
```

Okay, now make a function with two valves:

```c
void search_v1(int v1, int steps1, int v2, int steps2, int minute, long rate, long released)
{
    released += rate;
    if (minute == MINUTES)
    {
        if (released > max_released)
        {
            printf(" %ld\n", released);
            max_released = released;
        }
        return;
    }
    
    minute++;
    
    if (steps1 > 0)
    {
        int rate1 = rate;
        search_v2(v1, steps1 - 1, v2, steps2, minute, rate1, released);
    }
    else
    {
        valves[v1].open = TRUE;
        int rate1 = rate + valves[v1].rate;
        bool some = FALSE;
        for (int nv1 = 0; nv1 < NR; nv1++)
            if (valves[nv1].rate > 0 && !valves[nv1].open && nv1 != v1)
            {
                int d1 = dist[v1][nv1];
                if (minute + d1 <= MINUTES)
                {
                    search_v2(nv1, d1, v2, steps2, minute, rate1, released);
                    some = TRUE;
                }
            }
        if (!some)
            search_v2(v1, 0, v2, steps2, minute, rate1, released);
        valves[v1].open = FALSE;
    }
}

void search_v2(int v1, int steps1, int v2, int steps2, int minute, long rate, long released)
{
    if (steps2 > 0)
    {
        int rate2 = rate;
        search_v1(v1, steps1, v2, steps2 - 1, minute, rate2, released );
    }
    else
    {
        valves[v2].open = TRUE;
        int rate2 = rate + valves[v2].rate;
        bool some = FALSE;
        for (int nv2 = 0; nv2 < NR; nv2++)
            if (valves[nv2].rate > 0 && !valves[nv2].open && nv2 != v2)
            {
                int d2 = dist[v2][nv2];
                if (minute + d2 <= MINUTES)
                {
                    search_v1(v1, steps1, nv2, d2, minute, rate2, released);
                    some = TRUE;
                }
            }
        if (!some)
            search_v1(v1, steps1, v2, 0, minute, rate2, released);
        valves[v2].open = FALSE;
    }
}

int main(int argc, char *argv[])
{
    read_input();
    for (int i = 0; i < nr_valves; i++)
    {
        valves[i].open = FALSE;
        if (valves[i].rate > 0)
            nr_closed++;
    }
    calc_dist();
    int i_AA = find_valve("AA") - valves; 
    search_v1(i_AA, 0, i_AA, 0, 4, 0, 0);
    printf("\n\n%ld\n", max_released);
}
```

That did not find the right answer. From the example, it looks like but one
valve can be opened in the same minute. After some debugging, I begin to understand
what is wrong with the above algorithm. It looks like some valves have been opened
by both. In the remaining code, the `open` should be interpretted as: is going to
be opened. Thus preventing both parties going to the same valve. By carefully
studying the example answer of the second part, I noticed that both parties can
open a valve in the same step.

```c

void search_v1_m(int v1, int steps1, int v2, int steps2, int minute, long rate, long released, char *m)
{
    released += rate;
    if (minute == MINUTES)
    {
        if (released > max_released)
        {
            printf("%s: %ld\n", m, released);
            max_released = released;
        }
        return;
    }
    minute++;
    
    sprintf(m + strlen(m), " %ld", rate);
    char *nm = m + strlen(m);
    
    if (steps1 == 0)
    {
        sprintf(nm, " %s", valves[v1].name);
        rate += valves[v1].rate;
        bool some = FALSE;
        for (int nv1 = 0; nv1 < NR; nv1++)
            if (valves[nv1].rate > 0 && !valves[nv1].open)
            {
                int d1 = dist[v1][nv1];
                if (minute + d1 <= MINUTES)
                {
                    valves[nv1].open = TRUE;
                    search_v2_m(nv1, d1, v2, steps2, minute, rate, released, m);
                    valves[nv1].open = FALSE;
                    some = TRUE;
                }
            }
        if (!some)
            search_v2_m(v1, MINUTES, v2, steps2, minute, rate, released, m);
    }
    else
        search_v2_m(v1, steps1 - 1, v2, steps2, minute, rate, released, m);
    *nm = '\0';
}

void search_v2_m(int v1, int steps1, int v2, int steps2, int minute, long rate, long released, char *m)
{
    char *nm = m + strlen(m);
    
    if (steps2 == 0)
    {
        sprintf(nm, " %s", valves[v2].name);
        rate += valves[v2].rate;
        bool some = FALSE;
        for (int nv2 = 0; nv2 < NR; nv2++)
            if (valves[nv2].rate > 0 && !valves[nv2].open)
            {
                int d2 = dist[v2][nv2];
                if (minute + d2 <= MINUTES)
                {
                    valves[nv2].open = TRUE;
                    search_v1_m(v1, steps1, nv2, d2, minute, rate, released, m);
                    valves[nv2].open = FALSE;
                    some = TRUE;
                }
            }
        if (!some)
            search_v1_m(v1, steps1, v2, MINUTES, minute, rate, released, m);
    }
    else
        search_v1_m(v1, steps1, v2, steps2 - 1, minute, rate, released, m);
    *nm = '\0';
}


int main(int argc, char *argv[])
{
    read_input();
    for (int i = 0; i < nr_valves; i++)
    {
        printf("%2d %s %d %2d: ", i, valves[i].name, valves[i].rate, valves[i].nr_to);
        for (int j = 0; j < valves[i].nr_to; j++)
            printf(" %s", valves[i].to[j]->name);
        printf("\n");
    }
    for (int i = 0; i < nr_valves; i++)
    {
        valves[i].open = FALSE;
        if (valves[i].rate > 0)
            nr_closed++;
    }
    calc_dist();
    char mess[10000];
    int i_AA = find_valve("AA") - valves;
    int minute = 5;
    for (int v1 = 0; v1 < NR; v1++)
        if (valves[v1].rate > 0 && !valves[v1].open)
        {
            int d1 = dist[i_AA][v1];
            if (minute + d1 <= MINUTES)
            {
                valves[v1].open = TRUE;            
                for (int v2 = 0; v2 < NR; v2++)
                    if (valves[v2].rate > 0 && !valves[v2].open)
                    {
                        int d2 = dist[i_AA][v2];
                        if (minute + d2 <= MINUTES)
                        {
                            valves[v2].open = TRUE;
                            mess[0] = '\0';
                            search_v1_m(v1, d1, v2, d2, minute, 0, 0, mess);
                            valves[v2].open = FALSE;
                        }
                    }
                valves[v1].open = FALSE;
            }
        }
    printf("\n\n%ld\n", max_released);
}

```
And after (too) many hours, I found the answer to the second part
of the puzzle.

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
../IParse/software/MarkDownC Day16.md >day16.c; gcc -Wall -g day16.c -o day16; ./day16
```
