# Day 21 of Advent of Code 2022

I read the puzzle description in the morning and during
the day, I thought a bit about the data structure to be
used.
```c
typedef struct expr expr_t;
struct expr
{
    char name[5];
    char oper;
    num_t value;
    char l_name[5];
    char r_name[5];
    expr_t *left;
    expr_t *right;
    bool calc;
};
expr_t exprs[3211];
int nr;

void read_input()
{
    FILE *f = fopen("input/day21.txt", "r");
    char buffer[100];
    int i = 0;
    while (fgets(buffer, 100, f))
    {
        const char *s = buffer;
        strncpy(exprs[i].name, s, 4);
        exprs[i].name[4] = '\0';
        s += 6;
        if (('0' <= *s && *s <= '9') || *s == '-')
        {
            exprs[i].oper = '0';
            exprs[i].value = read_int(&s);
            printf("%s: %d\n", exprs[i].name, exprs[i].value);
        }
        else
        {
            strncpy(exprs[i].l_name, s, 4);
            exprs[i].l_name[4] = '\0';
            s += 5;
            exprs[i].oper = *s;
            s += 2;
            strncpy(exprs[i].r_name, s, 4);
            exprs[i].r_name[4] = '\0';
            printf("%s: %s %c %s\n", exprs[i].name, exprs[i].l_name, exprs[i].oper, exprs[i].r_name);
        }
        exprs[i++].calc = FALSE;
    }
    nr = i;
    printf("%d\n", nr);
    for (i = 0; i < nr; i++)
        if (exprs[i].oper != '0')
        {
            exprs[i].left = find(exprs[i].l_name);
            exprs[i].right = find(exprs[i].r_name);
        }
}

expr_t *find(const char *name)
{
    for (int i = 0; i < nr; i++)
        if (strcmp(exprs[i].name, name) == 0)
            return exprs + i;
    printf("Did not find |%s|\n", name);
    return 0;
}

int main(int argc, char *argv[])
{
    read_input();
}
```

Okay, that reads the input. It does not report any errors, so all the
names are found.

```c
num_t eval(expr_t *expr)
{
    if (expr->calc || expr->oper == '0')
        return expr->value;
    num_t l_val = eval(expr->left);
    num_t r_val = eval(expr->right);
    if (expr->oper == '+') expr->value = l_val + r_val;
    if (expr->oper == '-') expr->value = l_val - r_val;
    if (expr->oper == '*') expr->value = l_val * r_val;
    if (expr->oper == '/') expr->value = l_val / r_val;
    expr->calc = TRUE;
    printf("%s = %lld\n", expr->name, expr->value);
    return expr->value;
}

int main(int argc, char *argv[])
{
    ...
    num_t result = eval(find("root"));
    printf("%lld\n", result);
}
```
I needed to fix some things in the first part of the code about. But
then it returned a rather big number and I wondered if it was correct,
so I added a print statement,             

### Second part

Okay, we have to solve a complex equation. Lets see if there is a simple
way to solve it. Lets find how often 'humn' is used.

```c
int count(expr_t *expr)
{
    if (strcmp(expr->name, "humn") == 0)
        return 1;
    if (expr->oper != '0')
        return count(expr->left) + count(expr->right);
    return 0;
}

int main(int argc, char *argv[])
{
    read_input();
    expr_t *root = find("root");
    printf("left %d\n", count(root->left));
    printf("right %d\n", count(root->right));
}
```

Okay, it only occurs once in one of the two sides. That makes it easy.
```c
void solve(expr_t *expr, num_t answer)
{
    if (strcmp(expr->name, "humn") == 0)
    {
        printf("humn = %lld\n", answer);
        return;
    }
    if (expr->oper == '0')
    {
        printf("ERROR\n");
        return;
    }
    if (count(expr->left) == 1)
    {
        num_t right = eval(expr->right);
        num_t answer_left;
        if (expr->oper == '+') answer_left = answer - right;
        if (expr->oper == '-') answer_left = answer + right;
        if (expr->oper == '*') answer_left = answer / right;
        if (expr->oper == '/') answer_left = answer * right;
        solve(expr->left, answer_left);
    }
    else
    {
        num_t left = eval(expr->left);
        num_t answer_right;
        if (expr->oper == '+') answer_right = answer - left;
        if (expr->oper == '-') answer_right = left - answer;
        if (expr->oper == '*') answer_right = answer / left;
        if (expr->oper == '/') answer_right = left / answer;
        solve(expr->right, answer_right);
    }
}
int main(int argc, char *argv[])
{
    ...
    solve(root->left, eval(root->right));
}
```
That again returned a large number, which to my surprise was correct.

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
../IParse/software/MarkDownC Day21.md >p.c; gcc -Wall -g p.c -o p && ./p
```
