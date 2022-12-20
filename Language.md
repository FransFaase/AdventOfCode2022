# Language and interpreter

This page describes a specification oriented language implemented in C
for solving AventOfCode puzzles. The language contains a LL1 parser for
parsing possible input files.

For [Day 1](Day01.md) we like to be able to interpret the following
specification (for the second puzzle):
```
input = parse("input/day01.txt", ((decimal_integer newline)SEQ newline)SEQ)
Sum(Highest(3, (Sum(l) | l in input)))
```

Lets use a LISP like syntax for easy parsing. Something like:
```
def(input parse("input/day01.txt" SEQ(SEQ(decimal_integer newline) newline)))
Sum(Highest(3, Collect(Sum(l) in(l input))))
```


## Data structure

Define some generic data structure. Let's not worry about using too much
space and avoid using a `union`.

```c
typedef long long integer_t;
typedef enum data_type_t data_type_t;
enum data_type_t { dt_bool, dt_int, dt_string, dt_list, dt_set, dt_tree };

typedef struct data data_t, *data_p;

struct data
{
    data_type_t type;
    integer_t int_value;
    const char *str_value;
    data_t **children;
    int nr_children;
    int size;
    data_t *next;
};

data_t *new_data(data_type_t type)
{
    data_t *d = (data_t*)malloc(sizeof(data_t));
    d->type = type;
    d->str_value = 0;
    d->children = 0;
    d->nr_children = 0;
    d->size = 0;
    return d;
}

data_t *new_data_bool(bool v) { data_t *d = new_data(dt_bool); d->int_value = v; return d; }
bool data_is_bool(data_t *expr) { return expr != 0 && expr->type == dt_bool; }
bool data_bool(data_t *expr) { return data_is_bool(expr) ? (expr->int_value == 0 ? FALSE : TRUE) : FALSE; }

data_t *new_data_int(integer_t v) { data_t *d = new_data(dt_int); d->int_value = v; return d; }
bool data_is_int(data_t *expr) { return expr != 0 && expr->type == dt_int; }
integer_t data_int(data_t *expr) { return data_is_int(expr) ? expr->int_value : 0; }

data_t *new_data_string(const char *s) { data_t *d = new_data(dt_string); d->str_value = s; return d; }
bool data_is_string(data_t *expr) { return expr != 0 && expr->type == dt_string; }
const char *data_string(data_t *expr) { return data_is_string(expr) ? expr->str_value : ""; }

data_t *new_data_list() { data_t *d = new_data(dt_list); return d; }
bool data_is_list(data_t *expr) { return expr != 0 && expr->type == dt_list; }
data_t *new_data_set() { data_t *d = new_data(dt_set); return d; }
bool data_is_set(data_t *expr) { return expr != 0 && expr->type == dt_set; }
bool data_is_coll(data_t *expr) { return expr != 0 && (expr->type == dt_list || expr->type == dt_set); }

data_t *new_data_tree(const char *n) { data_t *d = new_data(dt_tree); d->str_value = n; return d; }
bool data_is_tree(data_t *expr, const char *name) { return expr != 0 && expr->type == dt_tree && strcmp(expr->str_value, name) == 0; }
bool data_is_a_ident(data_t *expr) { return expr != 0 && expr->type == dt_tree && expr->nr_children == 0; }
const char* data_ident(data_t *expr) { return data_is_a_ident(expr) ? expr->str_value : ""; }
void data_add_child(data_t *expr, data_t *child)
{
    if (expr == 0 || (expr->type != dt_list && expr->type != dt_tree)) return;
    if (expr->nr_children + 1 > expr->size)
    {
        expr->size = expr->size == 0 ? 8 : 2 * expr->size;
        expr->children = (data_t**)realloc(expr->children, sizeof(data_p) * expr->size);
    }
    expr->children[expr->nr_children++] = child;
}
int data_nr_children(data_t *expr) { return expr == 0 ? 0 : expr->nr_children; }
data_t* data_child(data_t *expr, int n)
{
    if (expr == 0 || n > expr->nr_children)
        return 0;
    return expr->children[n - 1];
}
void data_insert_child(data_t *expr, data_t *child)
{
    if (!data_is_set(expr)) return;
    int i = 0;
    while (i < expr->nr_children && data_smaller(expr->children[i], child))
        i++;
    if (i < expr->nr_children && !data_smaller(child, expr->children[i]))
        return;
    if (expr->nr_children + 1 > expr->size)
    {
        expr->size = expr->size == 0 ? 8 : 2 * expr->size;
        expr->children = (data_t**)realloc(expr->children, sizeof(data_p) * expr->size);
    }
    for (int j = expr->nr_children - 1; j >= i; j--)
        expr->children[j+1] = expr->children[j];
    expr->children[i] = child;
    expr->nr_children++;
}


void data_print(FILE *f, data_t *data)
{
    if (data == 0)
    {
        fprintf(f, "NULL");
        return;
    }
    
    if (data->type == dt_bool)
        fprintf(f, "%s", data->int_value ? "TRUE" : "FALSE");
    else if (data->type == dt_int)
        fprintf(f, "%lld", data->int_value);
    else if (data->type == dt_string)
        fprintf(f, "\"%s\"", data->str_value);
    else if (data->type == dt_tree)
        fprintf(f, "%s", data->str_value);
    else if (data->type == dt_set)
        fprintf(f, "SET");
        
    if (data->nr_children > 0)
    {
        fprintf(f, "(");
        bool first = TRUE;
        for (int i = 0; i < data->nr_children; i++)
        {
            if (first)
                first = FALSE;
            else
                fprintf(f, " ");
            data_print(f, data->children[i]);
        }
        fprintf(f, ")");
    }
}
    

```

## simple parser

First some error collecting and printing functions:

```c

const char *error = 0;
const char *error_at = 0;
data_t *error_data = 0;

void clear_error() { error = 0; error_at = 0; error_data = 0; }

void set_error_at(const char *err, const char *at)
{
    if (error == 0)
    {
        error = err;
        error_at = at;
        error_data = 0;
        error_print(stdout);
    }
}

void set_error_for(const char *err, data_t *d)
{
    if (error == 0)
    {
        error = err;
        error_at = 0;
        error_data = d;
        error_print(stdout);
    }
}

void error_print(FILE *f)
{
    if (error != 0)
    {
        fprintf(f, "Error: %s", error);
        if (error_at != 0)
            fprintf(f, " at %10.10s", error_at);
        if (error_data != 0)
        {
            fprintf(f, " for ");
            data_print(f, error_data);
        }
        fprintf(f, "\n");
    }
}
```

The actual parse function:
```c
void parse(const char **ref_s, data_t *result)
{
    for (;;)
    {
        while (**ref_s == ' ' || **ref_s == '\r' || **ref_s == '\n')
            (*ref_s)++;
        if (**ref_s == '\0' || **ref_s == ')')
            break;
        if (**ref_s == '"')
        {
            (*ref_s)++;
            const char *end = *ref_s;
            while (*end != '\0' && *end != '"')
                end++;
            if (*end == '\0')
            {
                set_error_at("Unterminated string const", *ref_s);
                return;
            }
            long len = end - *ref_s;
            char *s = (char*)malloc(len + 1);
            strncpy(s, *ref_s, len);
            s[len] = '\0';
            data_add_child(result, new_data_string(s));
            *ref_s = end + 1;
        }
        else if ('0' <= **ref_s && **ref_s <= '9')
        {
            integer_t v = 0;
            while ('0' <= **ref_s && **ref_s <= '9')
            {
                v = 10 * v + **ref_s - '0';
                (*ref_s)++;
            }
            data_add_child(result, new_data_int(v));
        }
        else
        {
            data_t *d = 0;
            if (('a' <= **ref_s && **ref_s <= 'z') || ('A' <= **ref_s && **ref_s <= 'Z'))
            {
                const char *end = *ref_s;
                while (('a' <= *end && *end <= 'z') || ('A' <= *end && *end <= 'Z') || ('0' <= *end && *end <= '9') || *end == '_')
                    end++;
                long len = end - *ref_s; 
                char *s = (char*)malloc(len + 1);
                strncpy(s, *ref_s, len);
                s[len] = '\0';
                d = new_data_tree(s);
                *ref_s = end;
            }
            if (**ref_s == '(' && d == 0)
            {
                d = new_data_list();
            }
            if (d == 0)
                break;
            data_add_child(result, d);
            if (**ref_s == '(')
            {
                (*ref_s)++;
                parse(ref_s, d);
                if (**ref_s != ')')
                {
                    set_error_at("Unterminated list", *ref_s);
                    return;
                }
                (*ref_s)++;
            }
        }
    }
}

```

Some code to test the above:
```c
void test_parse(const char *s)
{
    clear_error();
    const char *c = s;
    const char **ref_s = &c;
    data_t *d = new_data_list();
    parse(ref_s, d);
    printf("parse |%s| left |%s| <%s> |", s, c, error);
    data_print(stdout, d);
    printf("|\n");
}

void parser_tests()
{
    test_parse("1");
    test_parse("abds");
    test_parse("\"abc\"");
    test_parse("(1 2)");
    test_parse("a(1 3 \"gg\")");
}

int main(int argc, char *argv[])
{
    parser_tests();
}

```

Lets try to process the above specification for Day 1:

```c

integer_t execute(const char *s, FILE *print_spec)
{
    clear_error();
    const char *c = s;
    const char **ref_s = &c;
    data_t *spec = new_data_list();
    parse(ref_s, spec);
    if (error != 0)
        return -1;
    if (print_spec != 0)
    {
        data_print(print_spec, spec);
        printf("\n");
    }
    data_t *result = 0;
    context_t context;
    context_init(&context, 0);
    for (int i = 0; i < spec->nr_children && error == 0; i++)
    {
        result = eval(spec->children[i], &context);
    }
    if (error != 0)
        return -1;
    if (data_is_int(result) && result->next == 0)
        return data_int(result);
    error = "Non integer answer";
    return -1;
}

bool trace = FALSE;

data_t *eval(data_t *expr, context_t *context)
{
    if (trace)
    {
        printf("Eval: ");
        data_print(stdout, expr);
        printf("\n");
    }
    if (expr == 0)
        return 0;
    if (data_is_int(expr))
        return new_data_int(data_int(expr));    
    if (data_is_string(expr))
        return new_data_string(data_string(expr));
    if (data_is_tree(expr, "parse") && data_is_string(data_child(expr, 1)))
    {
        const char *str = read_file_contents(data_string(data_child(expr, 1)));
        if (str == 0)
            return 0;
        const char *s = str;
        data_t *r = parse_rule(&s, expr, 1);
        if (trace) data_print(stdout, r);
        return r;
    }
    if (data_is_tree(expr, "Sum"))
    {
        integer_t v = 0;
        for (int i = 0; i < expr->nr_children; i++)
        {
            data_t *child = expr->children[i];
            data_t *value = eval(child, context);
            if (data_is_int(value))
                v += data_int(value);
            else if (data_is_coll(value))
            {
                for (int j = 0; j < value->nr_children; j++)
                {
                    data_t *elem = value->children[j];
                    if (data_is_int(elem))
                        v += data_int(elem);
                    else
                        set_error_for("Non integer value", elem);
                }
            }
            else
                set_error_for("Non integer value or list", child);
        }
        if (trace) printf("Sum = %lld\n", v);
        return new_data_int(v);
    }
    if (data_is_tree(expr, "Max"))
    {
        integer_t max = 0;
        for (int i = 0; i < expr->nr_children; i++)
        {
            data_t *child = expr->children[i];
            data_t *value = eval(child, context);
            if (data_is_int(value))
            {
                integer_t v = data_int(value);
                if (v > max)
                    max = v;
                    
            }
            else if (data_is_coll(value))
            {
                for (int j = 0; j < value->nr_children; j++)
                {
                    data_t *elem = value->children[j];
                    if (data_is_int(elem))
                    {
                        integer_t v = data_int(elem);
                        if (v > max)
                            max = v;
                    }
                    else
                        set_error_for("Non integer value", elem);
                }
            }
            else
                set_error_for("Non integer value or list", child);
        }
        return new_data_int(max);
    }
    if (data_is_tree(expr, "Size"))
    {
        if (data_nr_children(expr) != 1)
        {
            set_error_for("Size has one argument", expr);
            return 0;
        }
        data_t *value = eval(expr->children[0], context);
        if (!data_is_coll(value))
        {
            set_error_for("Size not on collection", value);
            return 0;
        }
        return new_data_int(data_nr_children(value));
    }
    if (data_is_tree(expr, "Highest"))
    {
        if (data_nr_children(expr) != 2)
        {
            set_error_for("Highest expect two arguments", expr);
            return 0;
        }
        data_t *first = eval(data_child(expr, 1), context);
        if (!data_is_int(first))
        {
            set_error_for("First argument of Highest should be integer", expr);
            return 0;
        }
        int n = data_int(first);
        data_t *second = eval(data_child(expr, 2), context);
        if (!data_is_coll(second))
        {
            set_error_for("Second argument of Highest should be collection of integers\n", expr);
            return 0;
        }
        integer_t *highest = malloc(n * sizeof(integer_t));
        int nr_highest = 0;
        for (int i = 0; i < second->nr_children; i++)
        {
            data_t *elem = second->children[i];
            if (data_is_int(elem))
            {
                int v = data_int(elem);
                for (int i = 0; i < nr_highest; i++)
                    if (v > highest[i])
                    {
                        integer_t temp = highest[i];
                        highest[i] = v;
                        v = temp;
                    }
                if (nr_highest < n)
                    highest[nr_highest++] = v;
            }
        }
        data_t *result = new_data_list();
        for (int i = 0; i < nr_highest; i++)
            data_add_child(result, new_data_int(highest[i]));
        free(highest);
        return result;
    }
    if (data_is_tree(expr, "Collect"))
    {
        if (data_nr_children(expr) == 0)
        {
            set_error_for("Collect needs arguments", expr);
            return 0;
        }
        return eval_collect(expr, context);
    }
    if (data_is_tree(expr, "Range"))
    {
        if (data_nr_children(expr) != 2)
        {
            set_error_for("Range requires two arguments", expr);
            return 0;
        }
        data_t *first = eval(data_child(expr, 1), context);
        if (!data_is_int(first))
        {
            set_error_for("First argument of Range does not evaluate to number", expr);
            return 0;
        }
        int f = data_int(first);
        data_t *second = eval(data_child(expr, 2), context);
        if (!data_is_int(second))
        {
            set_error_for("Second argument of Range does not evaluate to number", expr);
            return 0;
        }
        int t = data_int(second);
        printf("Range %d-%d\n", f, t);
        data_t *result = new_data_set();
        for (integer_t i = f; i <= t; i++)
            data_insert_child(result, new_data_int(i));
        return result;
    }
    if (data_is_tree(expr, "or"))
    {
        bool b = FALSE;
        for (int i = 0; i < expr->nr_children; i++)
        {
            data_t *value = eval(expr->children[i], context);
            if (!data_is_bool(value))
            {
                set_error_for("Not a Boolean", value);
                return 0;
            }
            if (data_bool(value))
            {
                b = TRUE;
                break;
            }
        }
        return new_data_bool(b);
    }
    if (data_is_tree(expr, "nth"))
    {
        if (data_nr_children(expr) != 2)
        {
            set_error_for("nth requires two arguments", expr);
            return 0;
        }
        data_t *first = eval(data_child(expr, 1), context);
        integer_t n = data_int(first);
        if (!data_is_int(first) || n <= 0)
        {
            set_error_for("nth first argument should be positive integer", first);
            return 0;
        }
        data_t *second = eval(data_child(expr, 2), context);
        if (!data_is_coll(second))
        {
            set_error_for("nth second argument should be collection", second);
            return 0;
        }
        return data_child(second, n);
    }
    if (data_is_tree(expr, "subset"))
    {
        printf("Subset\n");
        if (data_nr_children(expr) != 2)
        {
            set_error_for("subset requires two arguments", expr);
            return 0;
        }
        data_t *first = eval(data_child(expr, 1), context);
        if (!data_is_coll(first))
        {
            set_error_for("subset first argument should be positive collection", first);
            return 0;
        }
        data_t *second = eval(data_child(expr, 2), context);
        if (!data_is_coll(second))
        {
            set_error_for("subset second argument should be collection", second);
            return 0;
        }
        bool is_subset = TRUE;
        for (int i = 0; i < first->nr_children && is_subset; i++)
        {
            data_t *f_child = first->children[i];
            is_subset = FALSE;
            for (int j = 0; j < second->nr_children && !is_subset; j++)
                is_subset = data_equal(f_child, second->children[j]);
        }
        printf("Answer %d\n", is_subset);
        return new_data_bool(is_subset);
    }
    if (data_is_a_ident(expr))
    {
        variable_t *var = context_variable(context, data_ident(expr));
        if (var != 0 && var->has_value)
            return var->value;
    }
    set_error_for("Cannot evaluate", expr);
    return 0;
}

bool data_smaller(data_t *first, data_t *second)
{
    if (data_is_int(first) && data_is_int(second))
        return data_int(first) < data_int(second);
    printf("Cannot < compare: ");
    data_print(stdout, first);
    printf(" with ");
    data_print(stdout, second);
    set_error_for("Cannot compare", second);
    return FALSE;
}

bool data_equal(data_t *first, data_t *second)
{
    if (data_is_int(first) && data_is_int(second))
        return data_int(first) == data_int(second);
    printf("Cannot == compare: ");
    data_print(stdout, first);
    printf(" with ");
    data_print(stdout, second);
    set_error_for("Cannot compare", second);
    return FALSE;
}

data_t *eval_collect(data_t *expr, context_t *parent_context)
{
    context_t context;
    context_init(&context, parent_context);
    
    add_free_vars(expr->children[0], &context);
    
    data_t *result = new_data_list();

    eval_conditions(expr, 1, &context, result);
    
    return result;
}

void add_free_vars(data_t *expr, context_t *context)
{
    if (expr->type == dt_tree)
    {
        if (expr->nr_children == 0)
        {
            variable_t *var = context_variable(context, expr->str_value);
            if (var == 0)
                context_add_variable(context, expr->str_value);
        }
        else
        {
            for (int i = 0; i < expr->nr_children; i++)
            {
                data_t *child = expr->children[i];
                add_free_vars(child, context);
            }
        }
    }
}

void eval_conditions(data_t *expr, int i, context_t *context, data_t *result_list)
{
    if (i >= expr->nr_children)
    {
        data_t *value = eval(expr->children[0], context);
        if (value != 0)
        {
            data_add_child(result_list, value);
        }
        return;
    }
    data_t *condition = expr->children[i];
    if (trace)
    {
        printf("Eval Condition(%d): ", i);
        data_print(stdout, condition);
    }
    if (data_is_tree(condition, "in"))
    {
        if (data_nr_children(condition) != 2 || !data_is_a_ident(data_child(condition, 1)))
        {
            set_error_for("in needs to arguments, where the first should be a ident", condition);
            return;
        }
        variable_t *var = context_variable(context, data_ident(data_child(condition, 1)));
        if (var == 0)
        {
            if (trace) printf("|%s|\n", data_child(condition, 1)->str_value);
            set_error_for("variable in in not defined", condition);
            return;
        }        
        else if (var->has_value)
        {
            set_error_for("in for value not implemented yet", condition);
            return;
        }
        data_t *second = eval(data_child(condition, 2), context);
        if (!data_is_coll(second))
        {
            set_error_for("second argument of in does not evaluate to a collection", condition);
            return;
        }
        if (trace) printf("Var |%s| has value\n", var->name);
        var->has_value = TRUE;
        for (int j = 0; j < second->nr_children; j++)
        {
            var->value = second->children[j];
            eval_conditions(expr, i+1, context, result_list);
        }
        if (trace) printf("Var |%s| has no value\n", var->name);
        var->has_value = FALSE;
        return;
    }
    if (data_is_tree(condition, "let"))
    {
        if (data_nr_children(condition) != 2 || !data_is_a_ident(data_child(condition, 1)))
        {
            if (trace) printf("%d %d\n", data_nr_children(condition), data_is_a_ident(data_child(condition, 1))); 
            set_error_for("let needs two arguments and the first must be an ident", condition);
            return;
        }
        data_t *value = eval(data_child(condition, 2), context);
        context_t let_context;
        context_init(&let_context, context);
        context_add_variable_with_value(&let_context, data_ident(data_child(condition, 1)), value);
        eval_conditions(expr, i+1, &let_context, result_list);
        return;
    }
    data_t *value = eval(condition, context);
    if (!data_is_bool(value))
    {
        set_error_for("condition does not evaluate to Boolean", condition);
        return;
    }
    if (data_bool(value))
        eval_conditions(expr, i+1, context, result_list);
        
    //set_error_for("Unknown condition in collect", conditions);
}

typedef struct context context_t;
struct context
{
    variable_t *variables;
    context_t *parent;
};

void context_init(context_t *context, context_t *parent)
{
    context->variables = 0;
    context->parent = parent;
}

variable_t *context_add_variable(context_t *context, const char *name)
{
    variable_t *new_var = malloc(sizeof(variable_t));
    new_var->name = name;
    new_var->has_value = FALSE;
    new_var->value = 0;
    new_var->next = context->variables;
    context->variables = new_var;
    return new_var;
}

variable_t *context_add_variable_with_value(context_t *context, const char *name, data_t *value)
{
    variable_t *new_var = context_add_variable(context, name);
    new_var->has_value = TRUE;
    new_var->value = value;
    return new_var;
}

typedef struct variable variable_t;
struct variable
{
    const char *name;
    bool has_value;
    data_t *value;
    variable_t *next;
};

variable_t *context_variable(context_t *context, const char *name)
{
    for (; context != 0; context = context->parent)
    {
        for (variable_t *var = context->variables; var != 0; var = var->next)
            if (strcmp(var->name, name) == 0)
                return var;
    }
    return 0;
}

data_t *context_value_for(context_t *context, const char *name)
{
    variable_t *var = context_variable(context, name);
    return var != 0 ? var->value : 0;
}

bool trace_parse = FALSE;

data_t *parse_rule(const char **ref_s, data_t *rule, int start)
{
    if (trace_parse)
    {
        printf("Start parse ");
        data_print(stdout, rule);
        printf("\n");
    }
    int nr_val = 0;
    data_t *result = 0;
    for (int i = start; i < rule->nr_children; i++)
    {
        data_t *rule_elem = rule->children[i];
        if (data_is_string(rule_elem))
        {
            const char *lit = data_string(rule_elem);
            size_t len = strlen(lit);
            if (strncmp(*ref_s, lit, len) != 0)
                return 0;
            *ref_s += len;
        }
        else if (data_is_tree(rule_elem, "newline"))
        {
            if (**ref_s != '\n' && **ref_s != '\r')
                return 0;
            if (**ref_s == '\r')
                (*ref_s)++;
            if (**ref_s == '\n')
                (*ref_s)++;
        }
        else
        {
            data_t *new_value = 0;
            if (data_is_tree(rule_elem, "SEQ"))
            {
                new_value = new_data_list();
                for (;;)
                {
                    data_t *elem = parse_rule(ref_s, rule_elem, 0);
                    if (elem == 0)
                        break;
                    data_add_child(new_value, elem);
                }
            }
            else if (data_is_tree(rule_elem, "decimal_integer"))
            {
                if (**ref_s < '0' || '9' < **ref_s)
                    return 0;
                integer_t v = 0;
                for (; '0' <= **ref_s && **ref_s <= '9'; (*ref_s)++)
                {
                    v = 10 * v + **ref_s - '0';
                }
                new_value = new_data_int(v);
            }
            else
            {
                set_error_for("Cannot parse", rule_elem);
                return 0;
            }
            if (trace_parse)
            {
                printf("Parse rule :");
                data_print(stdout, rule_elem);
                printf(" gave ");
                data_print(stdout, new_value);
                printf("\n");
            }
            nr_val++;
            if (nr_val == 1)
                result = new_value;
            else
            {
                if (nr_val == 2)
                {
                    data_t *d = new_data_list();
                    data_add_child(d, result);
                    result = d;
                }
                data_add_child(result, new_value);
            }
        }
    }
    if (trace_parse)
    {
        printf("End parse ");
        data_print(stdout, rule);
        printf(" returns ");
        data_print(stdout, result);
        printf("\n");
    }
    return result;
}
         

```

A testing framework
```c

void unit_tests()
{
    test_expr("Max(Collect(Sum(l) in(l parse(\"input/day01.txt\" SEQ(SEQ(decimal_integer newline) newline))))))", 69310);
    test_expr("Sum(Highest(3 Collect(Sum(l) in(l parse(\"input/day01.txt\" SEQ(SEQ(decimal_integer newline) newline)))))))", 206104);
}

void test_expr(const char *expr, integer_t answer)
{
    integer_t result = execute(expr, 0);
    if (error != 0)
    {
        fprintf(stderr, "ERROR: Executing '%s' ", expr);
        error_print(stderr);
    }
    else if (result != answer)
    {
        fprintf(stderr, "ERROR: Executing '%s' returned %lld, expected %lld\n", expr, result, answer);
    }
}
 
int main(int argc, char *argv[])
{
    unit_tests();
    integer_t result = execute("Size(Collect(l in(l parse(\"input/day04.txt\" SEQ(decimal_integer \"-\" decimal_integer \",\" decimal_integer \"-\" decimal_integer newline))) let(first Range(nth(1 l) nth(2 l))) let(second Range(nth(3 l) nth(4 l))) or(subset(first second) subset(second first))))", stderr);
    error_print(stderr);
    printf("result: %lld\n", result);
    return 0;
}
```


A function to read a file into a string:

```c
#include "unistd.h"


const char *read_file_contents(const char *fn)
{
    FILE *f = fopen(fn, "rt");
    if (f == 0)
        return 0;
    int fh = fileno(f);
    unsigned long len = lseek(fh, 0L, SEEK_END);
    lseek(fh, 0L, SEEK_SET);
    char *str = (char*)malloc(len+1);
    len = read(fh, str, len);
    str[len] = '\0';
    fclose(f);
    return str;
}
```


### Some standard definitions

```c
typedef int bool;
#define FALSE 0
#define TRUE 1
```

The command I use to process this markdown file, is:
```
../IParse/software/MarkDownC Language.md >lang.c; gcc lang.c -Wall -o lang; ./lang
```
