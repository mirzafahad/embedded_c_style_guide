# 1. General Rule

#### 1.1 Line Width
Limit the line width to 80 characters. This is especially helpful if somebody wants to print codes on a paper.

<hr>

#### 1.2 Braces
- Always use braces around conditional statements and loops. Even if it is an empty statement.
- Start (Left) brace should be on the next line, aligned with the end (right) brace.

Example:

```C
if (skillLevel > 10) 
{
    characterTitle = TITLE_KING;
}
else if (skillLevel > 5)
{
    characterTitle = TITLE_KNIGHT;
}
else
{ 
    characterTitle = TITLE_SOLDIER;
}
```

If you think not using braces cannot be catastrophic, check [Apple's SSL bug](https://blog.codecentric.de/en/2014/02/curly-braces/). The risk of creeping bugs can be entirely eliminated by the use of braces. The placement of the left brace on the following line allows for easy visual checking for the corresponding right brace.

<hr>

#### 1.3 Parentheses
- Do not rely on C’s operator precedence rules, as they may not be obvious to those who maintain the code. To aid clarity, use parentheses (and/or break long statements into multiple lines of code) to ensure proper execution order within a sequence of operations.

- Unless it is a single identifier or constant, each operand of the logical AND (&&) and logical OR (||) operators should be surrounded by parentheses.

Example:
```C
if ((killCount > 100) && (detectedByEnemies == false))
{
    characterTitle = TITLE_SILENT_ASSASSIN;
}
```

<hr>

#### 1.4 Keywords to Avoid

- Don't use `goto`. The use of `goto` makes code hard to follow. The occasional use of goto to handle an exceptional circumstance is acceptable if it simplifies and clarifies the code.

- It is a preferred practice to avoid all use of the `continue` keyword. The keyword `continue` still serve purposes in the language, but their use too often results in spaghetti code.

<hr>

#### 1.5 Keywords to Use Frequently
- Use the `static` keyword for any variables that do not need to be visible outside of the module in which they are declared. For example, any global variable declared in __C__ files. At the module-level, global variables and functions declared `static` are protected from external use. Heavyhanded use of `static` in this way thus decreases coupling between modules.

- The `const` keyword should be used:
    - To declare variables that should not be changed after initialization.
    - To define fields in a struct that should not be modified (e.g., in a struct overlay for memory-mapped I/O peripheral registers).
    - As a strongly typed alternative to `#define` for numerical constants.

 ```C
// Instead of doing this...
#define MAX_SKILL_LEVEL (100U)

// ...do this
const uint8_t MAX_SKILL_LEVEL = 100;
```
The upside of using const as much as possible is compiler-enforced protection from unintended writes to data that should be read-only.

- The `volatile` keyword should be used:
    - To declare a global variable accessible by an interrupt service routine.
    - To declare a global variable accessible by two or more threads.
    - To declare a delay loop counter.
    - To declare a pointer to a memory-mapped I/O peripheral register set.

```C
volatile sTimerReg_t *pTimer = (sTimerReg_t *)HW_TIMER_ADDR;
```

Proper use of volatile eliminates a whole class of difficult-to-detect bugs by preventing compiler optimizations that would eliminate requested reads or writes to variables or registers.

<hr>
<br>

# 2. Comment

- For single-line comments use C++ style i.e., preceded by `//`. You can expand that to 2-3 line comments too. But if it becomes a paragraph, it is probably better to use `/*...*/`. Use your good judgment.
- Add comments before every function you write:

```C
/******************************************************************************
* @file    File name
* @brief   Briefly tell what the function does
* @param   List the arguments
* @retval  If the function returns anything
* @note    Anything that needs to be mentiond. Otherwise omit it.
******************************************************************************/
```

- For multi-line comments use space+asterisk for every line:

```C
/*
 * This is a multi-line comments.
 * This is the 2nd line.
 * And the 3rd line.
 */
 ```

- Do not comment out code. Instead use `#if 0 ... #endif`
- If it is a debug code, use `#ifdef DEBUG ... #endif`
- All assumptions should be spelled out in comments.
- Use these markers to highlight issues and make those searchable:
    - “__WARNING:__” alerts a maintainer there is a risk in changing this code. For example, a delay loop counter’s terminal value was determined empirically and may need to change when the code is ported or the optimization level tweaked.
    - “__NOTE:__” provides descriptive comments about the “__why__” of a chunk of code—as distinguished from the “__how__” usually placed in comments. For example, a chunk of driver code deviates from the datasheet because there was an errata in the chip. Or that an assumption is being made by the original programmer.
    - “__TODO:__” indicates an area of the code is still under construction and explains what remains to be done. When appropriate, an all-caps programmer name or set of initials may be included before the word TODO (e.g., “MJB TODO:”).
- Keep the documentation as close to the code as possible. Instead of at beginning of a section/function.

<hr>
<br>

# 3. Functions

#### 3.1 Naming
When it comes to naming a function:
- use underscores to separate words and use lowercase letters.
- if it is a public function prefix with the module name.

Let's say you are writing an LCD driver - `lcd.h`, `lcd.c`.

```C
// private function
bool verify_coordinates_range(uint16_t x, uint16_t y);

// Public function
eLcdStatus_t LCD_init(void);
eLcdStatus_t LCD_draw_line(uint16_t x1, uint16_t y1, uint16_t x2, uint16_t y2);
```
- Macro name should not contain any lowercase letters.

<hr>

#### 3.2 Alignment
- Align all function prototypes (with the same/similar functionality) for better readability.

```C
eStatus_t  set(uint8_t a);
char*      get(void);
void       check_something(void);
```

- When function returns pointer, align asterisk to return type.

```C
/* OK */
char* my_func(void);

/* Wrong */
char *my_func(void);
```

<hr>

#### 3.3 Limiting scope
The `static` keyword should be used to declare all functions that do not need to be visible outside of the module in which they are declared. If it is a private function, use `static`.

Example:

```C
static bool verify_coordinates_range(uint16_t x, uint16_t y);
```


<hr>

#### 3.4 Divide and Conquer
"__And__" in the name of a function is a <font color="red">RED</font> flag. Divide it into two functions. Write functions that have one job and one job only. It will be easier to test. For example:

```C
// Don't do this...
static void parse_and_execute_cmd(char *cmd);

// ...instead do this.
static eCmdType_t parse_cmd(char *cmd);
static void execute_cmd(eCmdType_t cmd);
```

<hr>

#### 3.5 CONST argument
If one of your function arguments is a pointer (to something) whose value will not change/alter inside the function, use `const`. For example, if you want to compare `char` array with predefined strings (i.e. no changing of the argument array contents):

```C
static bool is_it_valid_cmd(const char *cmd);
```

<hr>

#### 3.6 Parameterized Macro
Don't use function-like macros (Parameterized Macro), if a function can be written to accomplish the same behavior. There are a lot of risks associated with the use of preprocessor defines, and many of them relate to the creation of parameterized macros. Where performance is important, note that C99 added C++’s inline keyword.

Example:
```C
// Don’t do this ...
#define MAX(A, B) ((A) > (B) ? (A) : (B))
// ... when you can do this instead.
inline uint32_t max(uint32_t num1, uint32_t num2)
```

But for some reason if you need to use it, follow these rules:
- Surround the entire macro body with parentheses.
- Surround each use of a parameter with parentheses.
- Try to limit each use of parameter no more than once, to avoid unintended side effects.
- Never include a transfer of control (e.g., `return` keyword).
- When macro uses multiple statements, protect it using `do-while (0)` statement.

```C
#define SET_POINT(p, x, y)  do {    \   // Backslash indicates statement continues in new line
    (p)->px = (x);                  \
    (p)->py = (y);                  \
} while (0)                             // 2 statements. No semicolon after while loop
```

The extensive use of parentheses (as shown in the example above) does not eliminate the unintended double increment possibility of a call such as MAX(i++, j++). 

Other risks of macro misuse include a comparison of signed and unsigned data or any test of floating-point data. Making matters worse, macros are invisible at run-time and thus impossible to step into within the debugger.

<hr>

#### 3.7 Thread of Execution
All functions that encapsulate threads of execution (a.k.a., tasks, processes) should be given names ending with `_thread` (or `_task`, `_process`).

Example:

```C
void alarm_thread(void *p_data)
{
    eAlarm_t alarm = ALARM_NONE;
    
    while(1)
    {
        alarm = alarm_task();
        // Process alarm here.
    }
}
```

Each task in a real-time operating system (RTOS) is like a mini-main(), typically running forever in an infinite loop. It is valuable to easily identify these important, asynchronous functions during code reviews and debugging.

<hr>

#### 3.8 Interrupt Service Routines
- All functions that implement ISRs should be given names ending with “_isr”.

- To ensure that ISRs are not inadvertently called from other parts of the software (they may corrupt the CPU and call stack if this happens), each ISR function should be declared static and/or be located at the end of the associated driver module as permitted by the target platform. This is not always possible but keep this in mind.

<hr>
<br>

# 4. Variables

#### 4.1 Naming

- Use camelCase for variable naming. Keep the first letter lowercase.
- If it is a global variable prepend with '__g__'.
- Function's arguments are local variables.
- A macro name should not contain any lowercase letters (only if a `const` cannot be used).
- No variable should have a name that begins with an underscore.
- Each variable’s name should be descriptive of its purpose.

```C
// Macro
#define MAX_ARRAY_LENGTH    (100U)

// Global variable
static eDifficultyLevel_t gCurrentDifficultyLevel = LEVEL_HARD;

static void cheak_dead_count(uint8_t deadCount)
{
    // Local variable
    uint8_t killCount = get_kill_count();

    if (killCount > deadCount)
    {
        // Gamer is probably fine
        return ;
    }

    // Died too many times, check difficulty level
    if (gCurrentDifficultyLevel != LEVEL_NOOB)
    {
        printf("Lowering your difficulty level might help!");
    }
    else
    {
        printf("Have you ever heard of "The Sims"?);
    }
}
```

<hr>

#### 4.2 Declaration
- Global variables' definitions should be grouped and placed at the top of a source code file and should be declared as `static`.
- Do not declare same type of variable on the same line. The cost of placing each declaration on a line of its own is low. By contrast, the risk that either the compiler or a maintainer will misunderstand your intentions is high.

Example:
```C
char* x, y; // Was y intended to be a pointer also? Don’t do this.
```

- Declare pointer variables with asterisk aligned to type.

```C
char* myVariable;
```

<hr>

#### 4.3 Initialization

- Do not initialize static and global variables to `0`, compiler will do it for you. When a varianle is declared inside a function it is not initialised.

```C
static uint8_t a; // Global variable 'a' is set to 0

void foo() 
{
    uint8_t b;   // set to whatever happens to be in memory
}
```

- Any pointer variable lacking an initial address should be initialized to `NULL`. 
- It is preferable to define local variables as you need them, rather than all at the top of a function. For readability reasons, it is better to declare local variables as close as possible to their first use.

Static analysis tools can scan all of the source code before each build, to warn about variables used prior to initialization.

<hr>
<br>

# 5. White Space

#### 5.1 Spaces
- Each of the keywords `if`, `while`, `for`, `switch`, and `return` should be followed by one space when there is additional program text on the same line.

- Each semicolon separating the elements of a for statement should always be followed by one space.

```C
if (depthInFt > 10){}

while (1){}

for (uint8_t i = 0; i < 10; i++){}

switch (catType){}

return SOUND_MEOW;
```

- Each of the assignment operators `=`, `+=`, `-=`, `*=`, `/=`, `%=`, `&=`, `|=`, `^=`, `~=`, and `!=` should always be preceded and followed by one space.

```C
weaponCount += 1;
```

- Each of the binary operators `+`, `-`, `*`, `/`, `%`, `<`, `<=`, `>`, `>=`, `==`, `!=`, `<<`, `>>`, `&`, `|`, `^`, `&&`, and `||` should always be preceded and followed by one space.

```C
HW_ADC_CONF_REG = previousConfiguration | (1 << START_BIT);
```

- Each of the unary operators `++`, `--`, `!`, and `~`, should be written without a space on the operand side.

```C
bonusCoin++;

if (!playGames)
{
    return SAD_EMOJI;
}
```


- The `?` and `:` characters that comprise the ternary operator should each always be preceded and followed by one space.

```C
uint32_t find_maximum(uint32_t a, uint32_t b)
{
    return ((a > b) ? a : b);
} 
```

- The structure pointer and structure member operators (-> and ., respectively) should always be __without__ surrounding spaces.

- The left and right brackets of the array subscript operator ([ and ]) should be __without__ surrounding spaces.

- The left and right parentheses of the function call operator should always be __without__ surrounding spaces.

- Except when at the end of a line, each comma separating function parameters should always be followed by one space.

- Each semicolon should follow the statement it terminates __without__ a preceding space.

<hr>

#### 5.2 Alignment

- The names of variables within a series of declarations should have their first characters aligned.
- The names of struct and union members should have their first characters aligned.
- The assignment operators within a block of adjacent assignment statements should be aligned.
- The `#` in a preprocessor directive should always be located at the start of a line, though the directives themselves may be indented within a `#if` or `#ifdef` sequence.

Example:
```C
#ifdef USE_UNICODE_STRINGS
    #define BUFFER_BYTES 128
#else
    #define BUFFER_BYTES 64
#endif

typedef struct sString
{
    uint8_t buffer[BUFFER_BYTES];
    uint8_t checksum;
}sString_t;


someVar           = 1;
someVariable2     = 2;
someMoreVariable3 = 3;

```
Visual alignments are easy on eyes and emphasizes similarity and also can be seen as a block of
related lines of code.
<hr>

#### 5.3 Indentation

- Do not use tabs, use spaces instead.
- Use 4 spaces per indent level.
- Whenever a line of code is too long to fit within the maximum line width, indent the second and any subsequent lines in the most readable manner possible.

```C
void sys_error_handler(uint8_t err)
{
    // Purposefully misaligned indentation
    if ((first_very_long_comparison_here   && 
         second_very_long_comparison_here) || 
         third_very_long_comparison_here)
    {
        ...
    }
}
```

<hr>

#### 5.4 Tabs

- The tab character (ASCII 0x09) should never appear within any source code file.
- When indents are needed in the source code, align via spaces instead.

Example:
```C
// When tabs are needed inside a string, use the ‘\t’ character.
#define COPYRIGHT “Copyright (c) 2020 Fahad Mirza.\tAll rights reserved.”
```

The width of the tab character varies by text editors and programmer preference, making consistent visual layout a continual source of headaches during code reviews and maintenance.

<hr>
<br>

# 6. Module Rules

#### 6.1 Naming
- All module names should consist entirely of lowercase letters, numbers, and underscores. No spaces should appear within the module’s header and source file names.
- Any module containing a `main()` function should have the word “main” as part of its source file name.

Example: `adc.c`, `adc.h`

<hr>

#### 6.2 Header Files
- There should always be precisely one header file for each source file and they should always have the same root name.
- The header file should identify only the procedures, constants, and data types (macros, #define, typedefs) about which it is strictly necessary for other modules to be informed.
    - It is a preferred practice that no variable ever be declared (via `extern`) in a header file.
    - No storage for any variable should be allocated in a header file.
- No public header file should contain a `#include` of any private header file.
- Always use `<` and `>` for C Standard Library include files, eg. `#include <stdlib.h>`.
- Always use `"..."` for custom libraries, eg. `#include "my_library.h"`
- Each header file should contain a preprocessor guard against multiple inclusion, as shown in the example below:

```C
#ifndef ADC_H
#define ADC_H
...
#endif // ADC_H
```
- If the header file is written specifically for `C`, then include `C++` preprocessor guard. as shown below:

```C
#ifndef ADC_H
#define ADC_H

#ifdef __cplusplus
extern "C" {
#endif // __cplusplus

...

#ifdef __cplusplus
}
#endif // __cplusplus

#endif // ADC_H
```

The C language standard gives all variables and functions global scope by default. The downside of this is unnecessary (and dangerous) coupling between modules. To reduce inter-module coupling, keep as many procedures, constants, data types, and variables as possible privately hidden within a module’s source file.

A header template file is available to download.

<hr>

#### 6.3 Source Files
- Each source file should be comprised of some or all of the following sections, in the order listed: 
    - comment block; 
    - include statements; 
    - constant and macro definitions; 
    - static data declarations; 
    - private function prototypes; 
    - public function bodies; 
    - then private function bodies.

- Each source file should always #include the header file of the same name (e.g., file `adc.c` should `#include “adc.h”`), to allow the compiler to confirm that each public function and its prototype match.
- Absolute paths should not be used in include file names.
- Each source file should be free of unused include files.
- No source file should `#include` another source file.

A source template file is available to download.

<hr>
<br>

# 7. Data Type Rules

#### 7.1 Naming
- All new structures, unions, and enumerations should be declared via a `typedef` and a `name`.
- The names of all new data types, including structures, unions, and enumerations, should use camelCase characters, with type at the beginning (s, u, e) and end with `_t`. The basic name doesn't need `_t`.

Example:
```C
typedef struct sStruct
{
    uint32_t a;
    uint32_t b;
}sStruct_t;

typedef enum eEnum
{
    ...
}eEnum_t;

typedef union uUnion
{
    ...
}uUnion_t;
```

- When initializing structure on declaration, use `C99` initialization style:

```C
sStruct_t foo_var = {
    .a = 4,
    .b = 5,
};
```

<hr>

#### 7.2 Fixed-Width Integers
- Whenever the width, in bits or bytes, of an integer value matters in the program, one of the fixed-width data types should be used in place of `char`, `short`, `int`, `long`, or `long long`. 

| Integer Width |  Signed  | Unsigned |
|---------------|:--------:|---------:|
|     8 bits    | int8_t   | uint8_t  |
|    16 bits    | int16_t  | uint16_t |
|    32 bits    | int32_t  | uint32_t |
|    64 bits    | int64_t  | uint64_t |

- The keywords `short` and `long` should not be used.
- The use of the keyword `char` should be restricted to the declaration of and operations concerning characters and strings.

<hr>

#### 7.3 Signed and Unsigned Integers
- Bit-fields should not be defined within signed integer types.
- None of the bitwise operators (i.e., `&`, `|`, `~`, `^`, `<<`, and `>>`) should be used to manipulate signed integer data.
- Signed integers should not be combined with unsigned integers in comparisons or expressions. 
- Decimal constants using `#define` should be declared with a ‘U’ at the end.

Example:
```C
#define SOME_CONSTANT  (6U)

uint16_t unsigned_a = 6;
int16_t  signed_b   = -9;
if (unsigned_a + signed_b < 4)
{
    // Execution of this block appears reliably logical, as -9 + 6 is -3
    ...
}
// ... but compilers with 16-bit int may legally perform (0xFFFF – 9) + 6.
```

Several details of the manipulation of binary data within signed integer containers are implementation-defined behaviors of the ISO C standards. Additionally, the results of mixing signed and unsigned integers can lead to data-dependent outcomes like the one in the code above.

<hr>

#### 7.4 Floating Point
- Avoid the use of floating-point constants and variables whenever possible. Fixed-point math may be an alternative.
- When floating point calculations are necessary:
    - Use the C99 type names `float32_t`, `float64_t`, and `float128_t`.
    - Append an ‘`f`’ to all single-precision constants (e.g., pi = 3.141592f).
    - Ensure that the compiler supports double-precision if your math depends on it.
    - Never test for equality or inequality of floating-point values.
    - Always invoke the `isfinite()` macro to check that prior calculations have resulted in neither INFINITY nor NAN.

<hr>

#### 7.5 Structures and Unions
- Appropriate care should be taken to prevent the compiler from inserting padding bytes within struct or union types. To know more read [structure packing](https://mirzafahad.github.io/2018-12-11-structure-packing/).
- Appropriate care should be taken to prevent the compiler from altering the intended order of the bits within bit-fields.

Example:
```C
typedef struct sTimer
{
    uint16_t count;    // offset 0
    uint16_t maxCount; // offset 2
    uint16_t _unused;  // offset 4
    uint16_t enable    : 2; // offset 6 bits 15-14
    uint16_t interrupt : 1; // offset 6 bit 13
    uint16_t _unused1  : 7; // offset 6 bits 12-6
    uint16_t complete  : 1; // offset 6 bit 5
    uint16_t _unused2  : 4; // offset 6 bits 4-1
    uint16_t periodic  : 1; // offset 6 bit 0
} sTimer_t;

// Preprocessor check of timer register layout byte count.
#if ((8 != sizeof(sTimer_t))
#error sTimer_t struct size incorrect (expected 8 bytes)”
#endif
```

<hr>

#### 7.6 Booleans
- Boolean variables should be declared as type bool.
- Non-Boolean values should be converted to Boolean via the use of relational operators (e.g., `<` or `!=`), not via casts.

Example:
```C
#include <stdbool.h>
...
bool inMotion = (0 != speedInMph);
```

- Always use `size_t` for length or size variables.

<hr>
<br>

# 8. Statement Rules
#### 8.1 Conditional Statements
- It is a preferred practice that the shortest (measured in lines of code) of the `if` and `else if` clauses should be placed first. Long clauses can distract the human eye from the decision-path logic. By putting the shorter clause earlier, the decision path becomes easier to follow. (And easier to follow is always good for reducing bugs.)
- Nested `if…else` statements should not be deeper than two levels. Use function calls or switch statements to reduce complexity and aid understanding. Deeply nested `if…else` statements are a sure sign of a complex and fragile state machine implementation. There is always a safer and more readable way to do the same thing.
- Assignments should not be made within an `if` or `else if` test.
- Any `if` statement with an `else if` clause should end with an `else` clause.
- Place opening curly bracket at the next line.

Example:
```C
if (NULL == pObject)
{
    result = ERR_NULL_PTR;
}
else if (pObject = malloc(sizeof(sObject_t))) // No assignments!
{
    ...
}
else
{
    // Normal processing steps, which require many lines of code. 
    ...
}
```

<hr>

#### 8.2 Switch Statements
- Each `case`'s content should be enclosed by braces (`{}`).
- The `break` for each `case` should be indented to align with the associated `case`, rather than with the contents of the `case` code block. Switch statements are prone to errors such as omitted break statements and unhandled cases. By aligning the `case` labels with their `break` statements it is easier to spot a missing `break`.
- All switch statements should contain a `default` block.
- Any case designed to fall through to the next should be commented to clearly explain the absence of the corresponding `break`.

Example:
```C
switch (err)
{
    case ERR_A:
    {
        ...
    }
    break;

    case ERR_B:
    {
        ...
    }
    // Intentional fallthrough: Also perform the steps for ERR_C.
    case ERR_C:
    {
        ...
    }    
    break;
    
    default:
    {
        ...
    }
    break;
}
```

<hr>

#### 8.3 Loops
- Magic numbers should not be used as the initial value or in the endpoint test of a `while`, `do…while`, or `for` loop.
- Except for the initialization of a loop counter in the `for` loop and the change to the same variable in the same statement, no assignment should be made in any loop’s controlling expression.
- Declare counter variables in `for` loop, unless if you need to access the variable later:

```C
/* OK */
for (uint8_t i = 0; i < 100; i++)
{
    ...  ...  ...
}

// if you need the variable later, then it is fine
uint8_t i;
for (i = 0; i < 100; i+) 
{
    ...  ...  ...
}

if (i == 100) 
{

}

/* Wrong */
uint8_t i;
for (i = 0; i < 100; i++) 
{
    ...  ...  ...
}
```
- Infinite loops should be implemented via controlling expression `while (1)`.
- Empty `while`, `do-while` or `for` loops must include brackets. Empty loop should contain a comment to explain why nothing needs to be done until after the loop terminates.

```C
/* OK */
while (*addr & BIT_13) {}    // Wait for bit 13 to be ready
while (*addr & BIT_13) 
{
    // Wait for bit 13 to be ready
}    

/* Wrong */
// Wait for bit 13 to be ready
while (*addr & (1 << 13));    // Curly brackets are missing.
```

Example:
```C
// No magic number (e.g., “100”) in the loop
for (uint8_t row = 0; row < 100; row++)  /* Wrong */
{
    // Descriptively-named constants prevent defects and aid readability.
}

for (uint8_t col = 0; col < NUM_COLS; col++) /* OK */
{
    ...
}
```

It is always important to synchronize the number of loop iterations to the size of the underlying data structure. Doing this via a descriptively-named constant prevents defects that result when changes in one part of the code, such as the dimension of an array, are not matched in other areas of the code.

<hr>

#### 8.4 Equivalence Tests
- When evaluating the equality of a variable against a constant, the constant should always be placed to the left of the equal-to operator (==).

Example:
```C
if (NULL == pObject)
{
    return ERR_NULL_PTR;
}

// This throws error
if (NULL = pObject)  // <--- Assignment error
{
    return ERR_NULL_PTR;
}
```

It is always desirable to detect possible typos and as many other coding defects as possible at compile-time. Defect discovery in later phases is not guaranteed and often also more costly. By following this rule, any compiler will reliably detect erroneous attempts to assign (i.e., `=` instead of `==`) a new value to a constant.