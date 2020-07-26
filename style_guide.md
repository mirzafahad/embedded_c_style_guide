# General Rule

#### Line Width
Limit the line width to 80 characters. This is specially helpful if somebody wants to print codes on a paper.

<hr>

#### Braces
- Always use braces around conditional statements and loops. Even if it is an empty statement.
- Start/Left brace should be on the following line, aligned with the end/right brace.

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

#### Parentheses
- Do not rely on C’s operator precedence rules, as they may not be obvious to those who maintain the code. To aid clarity, use parentheses (and/or break long statements into multiple lines of code) to ensure proper execution order within a sequence of operations.

- Unless it is a single identifier or constant, each operand of the logical AND (&&) and logical OR (||) operators shall be surrounded by parentheses.

Example:
```C
if ((killCount > 100) && (detectedByEnemies == false))
{
    characterTitle = TITLE_SILENT_ASSASSIN;
}
```

<hr>

#### Keywords to Avoid

- Don't use `goto`. The use of `goto` makes code hard to follow. The occasional use of goto to handle an exceptional circumstance is acceptable if it simplifies and clarifies the code.

- It is a preferred practice to avoid all use of the `continue` keyword. The keyword `continue` still serve purposes in the language, but their use too often results in spaghetti code.

<hr>

#### Keywords to Use Frequently
- Use the `static` keyword for any variables that do not need to be visible outside of the module in which they are declared. For example, any global variable declared in __C__ files. At the module-level, global variables and functions declared `static` are protected from external use. Heavyhanded use of `static` in this way thus decreases coupling between modules.

- The `const` keyword shall be used:
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

- The `volatile` keyword shall be used:
    - To declare a global variable accessible by an interrupt service routine.
    - To declare a global variable accessible by two or more threads.
    - To declare a delay loop counter.
    - To declare a pointer to a memory-mapped I/O peripheral register set.

```C
volatile sTimerReg_t  *pTimer = (sTimerReg_t *) HW_TIMER_ADDR;
```

Proper use of volatile eliminates a whole class of difficult-to-detect bugs by preventing compiler optimizations that would eliminate requested reads or writes to variables or registers.

<hr>
<br>

# Comment

- For single-line comments use C++ style i.e., preceded by `//`. You can expand that to 2-3 line comments too. But if it becomes a paragraph, it is probably better to use `/*...*/`. Use your good judgment.
- Add comments before every function you write:

```C
/******************************************************************************
* @brief   Briefly tell what the function does
* @param   List the arguments
* @retval  If the function returns anything
* @note    Anything that needs to be mentiond. Otherwise omit it.
******************************************************************************/
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

# Functions

#### Naming
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

<hr>

#### Limiting scope
The `static` keyword shall be used to declare all functions that do not need to be visible outside of the module in which they are declared. If it is a private function, use `static`.

Example:

```C
static bool verify_coordinates_range(uint16_t x, uint16_t y);
```


<hr>

#### Divide and Conquer
"__And__" in the name of a function is a <font color="red">RED</font> flag. Divide it into two functions. Write functions that have one job and one job only. It will be easier to test. For example:

```C
// Don't do this...
static void parse_and_execute_cmd(char *cmd);

// ...instead do this.
static eCmdType_t parse_cmd(char *cmd);
static void execute_cmd(eCmdType_t cmd);
```

<hr>

#### CONST argument
If one of your function arguments is a pointer (to something) whose value will not change/alter inside the function, use `const`. For example, if you want to compare `char` array with predefined strings (i.e. no changing of the argument array contents):

```C
static bool is_it_valid_cmd(const char *cmd);
```

<hr>

#### Parameterized Macro
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

The extensive use of parentheses (as shown in the example above) does not eliminate the unintended double increment possibility of a call such as MAX(i++, j++). 

Other risks of macro misuse include a comparison of signed and unsigned data or any test of floating-point data. Making matters worse, macros are invisible at run-time and thus impossible to step into within the debugger.

<hr>

#### Thread of Execution
All functions that encapsulate threads of execution (a.k.a., tasks, processes) shall be given names ending with `_thread` (or `_task`, `_process`).

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

#### Interrupt Service Routines
- All functions that implement ISRs shall be given names ending with “_isr”.

- To ensure that ISRs are not inadvertently called from other parts of the software (they may corrupt the CPU and call stack if this happens), each ISR function shall be declared static and/or be located at the end of the associated driver module as permitted by the target platform. This is not always possible but keep this in mind.

<hr>
<br>

# Variables

#### Naming

- Use CamelCase for variable naming. Keep the first letter lowercase.
- If it is a global variable prepend with '__g__'.
- Function's arguments are local variables.

```C
// Global variable
static uint8_t gCurrentDifficultyLevel = LEVEL_HARD;

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
<br>

# White Space

#### Spaces
- Each of the keywords `if`, `while`, `for`, `switch`, and `return` shall be followed by one space when there is additional program text on the same line.

- Each semicolon separating the elements of a for statement shall always be followed by one space.

```C
if (depthInFt > 10){}

while (1){}

for (uint8_t i = 0; i < 10; i++){}

switch (catType){}

return SOUND_MEOW;
```

- Each of the assignment operators `=`, `+=`, `-=`, `*=`, `/=`, `%=`, `&=`, `|=`, `^=`, `~=`, and `!=` shall always be preceded and followed by one space.

```C
weaponCount += 1;
```

- Each of the binary operators +, -, *, /, %, <, <=, >, >=, ==,!=, <<, >>, &, |, ^, &&, and || shall always be preceded and followed by one space.

```C
HW_ADC_CONF_REG = previousConfiguration | (1 << START_BIT);
```

- Each of the unary operators ++, --, ! , and ~, shall be written without a space on the operand side.

```C
bonusCoin++;

if (!playGames)
{
    return SAD_EMOJI;
}
```


- The ? and : characters that comprise the ternary operator shall each always be preceded and followed by one space.

```C
uint32_t find_maximum(uint32_t a, uint32_t b)
{
    return ((a > b) ? a : b);
} 
```

- The structure pointer and structure member operators (-> and ., respectively) shall always be __without__ surrounding spaces.

- The left and right brackets of the array subscript operator ([ and ]) shall be __without__ surrounding spaces.

- The left and right parentheses of the function call operator shall always be __without__ surrounding spaces.

- Except when at the end of a line, each comma separating function parameters shall always be followed by one space.

- Each semicolon shall follow the statement it terminates __without__ a preceding space.

<hr>

#### Alignment

<hr>

#### Indentation

<hr>

#### Tabs