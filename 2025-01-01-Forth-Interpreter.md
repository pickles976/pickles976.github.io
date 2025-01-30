---
layout: post
title: Forth Interpreter
---

![](/images/forth/forth_logo.png)

For some reason, I have this nagging desire to go backwards in time. To re-tread the paths walked by the greybeards. Maybe I fallaciously assume that
the further back I look in time, the further I will be able to see into the future.

Whatever the reason, there are others who I think feel the same. News articles pop up about Zoomers who eschew their smart phones altogether. There are projects like Collapse OS, 
which, along with the collapsenik ethos, harkens for a return to a simpler and more pure form of computing.

So I wrestled with SICP, too hard.  

Tried to work through Crafting Interpreters, and while I found it to be useful, certain creature comforts in the Lox language add a bit too much cognitive overhead for me to 
really grok the problem being solved. Operator precedence and infix notation is confusing.  

I read a book called "The Schematics of Computation" which has been described as "SICP for mortals".
I got the basics down, and did what I always do when learning a new language, implemented a Trie.

Writing a meta-circular evaluator in Scheme seems like putting the cart before the horse to me. Former students who took SICP claim that it was an "aha" moment for them,
but for me it's a bit like learning a language and then trying to write a thesis in it. Better to start of reading children's books in the new language in my opinion. But I still wanted to know how a Lisp interpreter worked. So I implemented one in a language I was very familiar with.

https://norvig.com/lispy.html

So code is a list and we can manipulate lists, therefore we can manipulate code. The heirarchy of our environments is reflected in the heirarchy of our abstract syntax tree.
So we can parse and evaluate Lisp at the same time, just recursively applying the same handful of functions as we go down. Neat!

I now feel that I have enough theory to write Lisp interpreters in other languages. But why bother writing one in *any* language. I should try writing it in a fast language that can compile to any target. C.

Yikes, I don't know C. I know there's `malloc` and `free` and `*&->`. Everything goes in a header and does linking or something? It's esoteric and weird and not particularly pretty or ergonomic. But hey, the best day to learn something was yesterday.

Off to Exercism to get some practice. I figure if I can make a [Stack](https://exercism.org/tracks/c/exercises/matching-brackets/iterations?idx=3), a [Linked List](https://exercism.org/tracks/c/exercises/linked-list), and a [Dynamic Array](https://exercism.org/tracks/c/exercises/list-ops/iterations?idx=4) I can probably write an interpreter (I can figure out a hash map on my own).

But Lisp has some recursive nonsense that's a bit difficult to wrap your head around. I'd like to just fumble around through the basics of parsing and evaluating before I have to worry about the more complex aspects of Lisp. What language has even *less* syntax than Lisp?

Enter Forth.

> Sidenote, it's a little embarrasing as a 27 year old to talk to people who have been coding things like virtual machines or 6502 assembly ROM hacks when they were literal children. But on the flip-side, that means that this should literally be child's play. 

1. The Naive implementation


[](https://github.com/pickles976/SDLForth/blob/forth-String-Table/dtable_dynamic.h#L25)

```C
typedef struct {
    size_t *keys;
    char **strings;
    StringArray **values;

    size_t capacity; // capacity
    size_t length; // length
} DD_Table;
```
2. Revelation

3. The oldheads on IRC

3. Indirect threaded bytecode interpreter

```C
void step(VM *vm, IntStack *data_stack, IntStack *call_stack, SD_Table *sd_table, DD_Table *dd_table) {
    
    Instruction ins = vm->code[vm->ip++];
    switch(ins.type) {
        case VALUE:
            push_int_to_stack(data_stack, (int)ins.bytecode.value);
            break;
        case FUNC:
            // call our builtin function
            ins.bytecode.builtin(data_stack);
            break;
        case JUMP:
            // Push current address to call stack so we dont forget it, and jump the IP to the specified address
            push_int_to_stack(call_stack, vm->ip);
            vm->ip = (size_t)ins.bytecode.address;
            break;
        case RETURN:
            int return_int;
            pop_int_from_stack(call_stack, &return_int);
            vm->ip = (size_t)return_int;
            break;
        default:
            break;
    }
}
```

```commandline
Keys: SWAP, *, +, -, ., /, DROP, DUP, 
> : SQUARE DUP * ;
VM: [FUNC][FUNC][RETURN]
> : FOURTH SQUARE SQUARE ;
VM: [FUNC][FUNC][RETURN][JUMP, 0][JUMP, 0][RETURN]
> 2
STACK: [2]
VM: [FUNC][FUNC][RETURN][JUMP, 0][JUMP, 0][RETURN][VALUE, 2]
> FOURTH 
STACK: [16]
VM: [FUNC][FUNC][RETURN][JUMP, 0][JUMP, 0][RETURN][VALUE, 2][JUMP, 3]
```
