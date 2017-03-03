# Contributing to Morphux

## Reporting Issues
If you're having an issue with a Morphux service, please report it to the specific service.

| Target | Maintainer(s) | Issues URL |
|------------|------------|-----------|
| Global     | [Louis](mailto:louis@morphux.org) | [Morphux/Morphux/issues](https://github.com/Morphux/Morphux/issues) 
| Installer  | [Louis](mailto:louis@morphux.org) | [Morphux/Installer/issues](https://github.com/Morphux/installer/issues) |
| Morphux Package Manager | [Louis](mailto:louis@morphux.org) & [Gosti](mailto:) | [Morphux/Mpm/issues](https://github.com/Morphux/mpm/issues) |
| Builder | [Enerdhil](mailto:enerdhil@morphux.org) | [Morphux/Builder/issues](https://github.com/Morphux/Builder/issues) |
| Websites | [Louis](mailto:louis@morphux.org) | [Morphux/Morphux/issues](https://github.com/Morphux/Morphux/issues)

## Git

### Commit
```
Action(Target): Title:

Precision (If needed)

```

Example:
```
Add(Tests): Add tests for m_log functions:

Testing a bad behavior on m_log_1 test unit
Fix a memory leak on m_log_init

```

### Branchs
The ```master``` is used as a **stable** branch. All the development code and 
the Pull Requests must happen on the ```unstable``` branch.

The repository maintainer must decide when too merge the ```unstable``` branch in the ```master```.

```
Code -> Review -> Merge in unstable -> Delivery in master
```

## C Code style

### Indentation
- All the identation must be done with 4 spaces.
- No line should exceed 80 columns.

### Braces
- Bad:
```
if (i > 0) {
    /* Something */
} else {
    /* Something */
}
```

```
if (i > 0)
    for (int z = 0; z < i; z++)
        ret += z;
```

- Good:
```
if (i > 0)
{
    /* Something */
}
else
{
    /* Something */
}
```

```
if (i > 0)
{
    for (int z = 0; z < i; z++)
        ret += z;
}
```

For functions, arrays and structs, braces must be like that:
```
struct          something_s {
    /* ... */
};
```

```
void something(int j) {
    /* ... */
}
```

### Naming
- Bad:
```
bool ParseUserArgs(const char *args);

bool parseuserargs(const char *args);

bool parse(const char *userinput);

static bool
parse(const char *user_input);
```

- Good:
```
bool parse_user_args(const char *args);

bool parse(const char *user_input);

static bool parse(const char *user_input);
```

### Alignement
- Bad:
```
int             j;
void *ret;
```

- Good:
```
int     j;
void    *ret;
```

### Preprocessor
- Bad:
```
#ifndef DEBUG
/* ... */
#endif
```

```
#ifndef DEBUG
#define DEBUG
/* ... */
#endif
```

- Good:
```
#ifndef DEBUG
/* ... */
#endif /* DEBUG */
```

```
#ifndef DEBUG
# define DEBUG
/* ... */
#endif /* DEBUG */
```

### Typedefs
All user used structure must be typedefed:
```
typedef struct          something_s {
    /* ... */
}                       something_t;
```
Enums too:
```
typedef enum    my_enum_e {
    /* ... */
}               my_enum_t;
```

### Namings
- Globals: ```g_var_name```
- Typedefs: ```var_name_t```
- Enums: ```var_name_e```
- Structs: ```var_name_s```

### Comments
- Functions:
```
/*!
 * \brief Brief title
 * \param[in] a Explanation of the variable
 * \param[out] s Explanation of the variable
 *
 * Full Explanation
 * \note Something worth noting
 * \return true on success, false on failure
 */
bool something(int a, char **s);
```

All the inline comments (not doxygen) must be like ```/* ... */```, **no** single lines comment: ```// ...```

### One line instructions
- Bad:
```
for (int i = 0; i < 10; i++);
```

```
if (very == true && long == true
    && condition == true)
    i++;
```

- Good:
```
for (int i = 0; i < 10; i++)
    ;
```

```
if (very == true && long == true
    && condition == true)
{
    i++;
}
```
- If a condition necessit a line return, add braces around it.
- The operator (|| && ...) must begin the new line
