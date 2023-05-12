---
title: COBOL Calculator
date: 2023-05-11 16:00:00
categories: [programming]
tags: [programming,cobol,blog]
---

# COBOL Calculator

A friend of mine mentioned self studying COBOL a few weeks ago and, after hearing them talk about it and how it's more procedural than object oriented, I thought it seemed fun to have a bash at it. Hitherto, I had only ever programmed in modern languages really so it seemed like a fun challenge to see if I could wrap my head around something archaic following more linear and ostensibly rigid conventions.

I'm not sure if I had ever even heard of COBOL before this, having no noteworthy formal computer science background, and I was surprised to learn that the language was, to apparently a significant extent, designed by a woman since I would have expected such a major foundation of general computing being made by a woman to be far more celebrated given how the field is endlessly bemoaned as excessively male.

The actual project itself is less interesting than the language since making a calculator is not exactly difficult but the constraints of working with COBOL and the limitations, at least as far as I could comprehend to use them, of many of its basic functions forced a bit of creativity. Simple things like being unable to search a string and replace "a" by "ab" because "ab" is longer than "a" or the inability to replace the nth character of a string directly were not hard to overcome but modern langauges often resolve these issues for you and don't demand you create your own little subroutine to do it.

In the end it was fairly surprising how effortless the language was to learn how how, in many ways, things haven't really changed they have just gotten less strict and the multitude of functions and libraries has exploded. I suppose programming is just programming and ultimately the computer is turning it all into machine code anyway.

## Preliminaries

Setting up the development environment was quite simple. GnuCOBOL has a variety of methods to choose from and the FAQ was more than comprehensive enough for my needs. I was just using Windows so Arnold Trembley's installer was my go-to. There were some headaches regarding path variables but a quick look at what set_env.cmd was actually doing made it clear how to resolve them. Specifically, while compilation was no issue calling subroutines was failing to resolve and it was just because COBOL was looking here:
```
set "COB_LIBRARY_PATH=%COB_MAIN_DIR%extras"
```
Which was not, to me, an intuitive place to put them.

There do exist IDE which presumably make it easier to write COBOL and certainly make it more readable but I couldn't be bothered setting them up. The compiler turns everything into C to then compile as C, based on my understanding.

### Links to GnuCOBOL and Arnold Trembley
https://gnucobol.sourceforge.io/

https://www.arnoldtrembley.com/GnuCOBOL.html

# Project

The task was to make a simple integer calculator capable of doing + - * / while also following the correct order of operations.


## Code

### Note
Unsurprisingly given its age, there seem to be a variety of styles regarding how to write COBOL. I have frankly no idea what the proper conventions are for naming variables or anything else but eventually settled on a style halfway between copying my friend, copying snippets of code I saw elsewhere, and what I personally found most readable. My impression was that modern COBOL forgoes the upper case for keywords but I liked the retro feel it gives the language and it seems silly to even talk about "modern COBOL".

### Snippet from main
```
IDENTIFICATION DIVISION.
PROGRAM-ID. Calculator.
AUTHOR. dakkou.

DATA DIVISION.
    WORKING-STORAGE SECTION.
    01  WS-input             PIC X(20).
    01  WS-operator          PIC X(1) VALUE "/".
    01  WS-tally             PIC 9(1) VALUE 0.

PROCEDURE DIVISION.
    DISPLAY "Input: ".
    ACCEPT WS-input.
    INSPECT WS-input
        REPLACING ALL "+"
        BY "A".
    CALL 'Signing' USING WS-input.

    INSPECT WS-input TALLYING WS-tally FOR ALL WS-operator.
    DISPLAY WS-tally " instances of /".
    PERFORM WS-tally TIMES
        CALL 'Pedmas' USING WS-input, WS-operator
    END-PERFORM.
    SET WS-operator TO "*".
    SET WS-tally TO 0
    ...
```
The program id might be Calculator but, for all intents and purposes, this is the main method: it's the entry point for the program.

One of the first things apparent about COBOL is how wordy it is: everything is practically in plain, albeit stilted, English. This comes from its history where it was designed for programmers as a high level language and frankly it does it well. While it takes a little while to get used to the odd syntax, once you have figured it out the documentation makes it very easy to follow.
For example, the INSPECT function, which was vital to this project for reading the string input and really was the cornerstone of the whole thing, is documented here https://www.ibm.com/docs/en/cobol-zos/6.2?topic=statements-inspect-statement and initially the flowcharts are somewhat intimidating and seemingly arcane but they are really very simple and readable after a few examples and illustrations to understand what identifiers, literals, phrases, etc all are. The keywords such as FIRST, BEFORE, ALL, etc are all entirely self explanatory. The initial hurdle, at least for me, was the peculiar syntax, the lack of bracketry, and the strange use of periods (which became more confusing when IF statements got involved).

Variables are also defined in an unusual way. They are given a PIC(ture) and are basically an array of that. For example, 9 is the numbers 0-9 or X is the characters a-Z and then there are (n) of these. COBOL could get a bit finicky when operating on variables of different lengths but really it was my own fault for no just being liberal with my lengths and trying to minimise them at the start. Even now, the given code will only work on inputs of at most 20 characters long which is not a particularly long sum. Another terrible mistake I made at the start was not signing my numbers to start with. I'm sure it immediately stands out that the first thing I did, after taking an input, is replace "+" by "A".

### Signing
```
DENTIFICATION DIVISION.
PROGRAM-ID. Signing.
AUTHOR. dakkou.

DATA DIVISION.
    WORKING-STORAGE SECTION.
    01  WS-input              PIC X(40).
    01  WS-partialInput       PIC X(40).
    01  WS-index01            PIC 9(2).
    01  WS-index02            PIC 9(2).
    LINKAGE SECTION.
    01  L-input               PIC X(20).

PROCEDURE DIVISION USING L-input.
    SET WS-index01 TO 1.
    SET WS-index02 TO 0.
    PERFORM FUNCTION LENGTH(FUNCTION TRIM(L-input)) TIMES
        IF WS-index01 EQUALS 1 AND L-input(1:1) NOT EQUAL "-" THEN
            STRING "+" INTO WS-input
        END-IF
        IF L-input(WS-index01:1) EQUALS "A"
            STRING WS-input DELIMITED BY SPACE "A+" INTO WS-input
        ELSE
            IF WS-index01 NOT EQUAL 1 AND L-input(WS-index01:1) EQUALS "-" AND L-input(WS-index02:1) NOT EQUAL "*" AND L-input(WS-index02:1) NOT EQUAL "/"
                STRING WS-input DELIMITED BY SPACE "A-" INTO WS-input
            ELSE
                STRING WS-input DELIMITED BY SPACE L-input(WS-index01:1) INTO WS-input
            END-IF
        END-IF
        ADD 1 TO WS-index01
        ADD 1 TO WS-index02
    END-PERFORM.
    SET L-input TO WS-input.
EXIT PROGRAM.
```
This simple subroutine just iterates over the input putting "+" in front of postiive numbers and "-" in front of negative numbers by reconstructing the input into WS-input. There is an obvious problem here regarding the potential for overflow errors since 1+1+1... becomes +1A+1A+1... and my calculator doesn't do anything about it but a simple check would be to test the length before setting the linked input, that is the variable defined in the parent program, to the working "input" here.

Signing was actually the last thing added to the program because, and this was totally unexpected when I started, subraction proved to be the most problematic operator creating issues like 10-3+1=6 rather than the correct 10-3+1=8. This was because the program operated on numbers in pairs rather than in some sort of holistic sense and the solution to just reduce addition and subtraction into signed addition was definitely the correct one. There was also the issue of multiplying and dividing by negative numbers. I think when I first started I was just too impetuous and just defined numbers as numbers rather than thinking about whether or not they even needed to be signed - other languages often do not make the distinction.

The majority of work went into the subroutine Pedmas (which at this time does not include the P or e).
### Pedmas
```
IDENTIFICATION DIVISION.
PROGRAM-ID. Pedmas.
AUTHOR. dakkou.

DATA DIVISION.
    WORKING-STORAGE SECTION.
    01  WS-input            PIC X(20).
    01  WS-working01        PIC X(20).
    01  WS-working02        PIC X(20).
    01  WS-operand01        PIC S9(4).
    01  WS-operand02        PIC S9(4).
    01  WS-numValue         PIC 9(4).
    01  WS-partialInputL    PIC X(20).
    01  WS-partialInputR    PIC X(20).
    LINKAGE SECTION.
    01  L-input             PIC X(20).
    01  L-operator          PIC X(1).

PROCEDURE DIVISION USING L-input, L-operator.
    SET WS-input TO L-input.

    INSPECT WS-input REPLACING FIRST L-operator BY SPACE.

    UNSTRING WS-input
        DELIMITED BY SPACE
        INTO WS-partialInputL, WS-partialInputR
    END-UNSTRING.
    
    SET WS-partialInputL TO FUNCTION REVERSE(FUNCTION TRIM(WS-partialInputL)).

    CALL 'NumberIsolation' USING WS-partialInputL, WS-working01.

    SET WS-partialInputL TO FUNCTION REVERSE(FUNCTION TRIM(WS-partialInputL)).
    SET WS-working01 TO FUNCTION REVERSE(WS-working01).
    
    CALL 'NumberIsolation' USING WS-partialInputR, WS-working02.

    SET WS-operand01 TO FUNCTION NUMVAL(WS-working01).
    SET WS-operand02 TO FUNCTION NUMVAL(WS-working02).

    CALL 'SelectFunction' USING WS-operand01, WS-operand02, L-operator.

    SET WS-operand02 TO 0.
    INSPECT WS-partialInputL TALLYING WS-operand02 FOR ALL SPACE.

    IF WS-operand02 IS EQUAL 20 THEN
        SET WS-operand02 TO 0
        INSPECT WS-partialInputR TALLYING WS-operand02 FOR ALL SPACE
    END-IF.

    IF WS-operand02 IS EQUAL 20 THEN
        SET L-input TO " ".
        IF WS-operand01 IS NEGATIVE THEN
            SET WS-numValue TO WS-operand01
            STRING "-" WS-numValue INTO L-input
    END-IF.

    SET L-input TO " ".
    IF WS-operand01 IS NEGATIVE THEN
        SET WS-numValue TO WS-operand01
        STRING WS-partialInputL DELIMITED BY SPACE "-" WS-numValue WS-partialInputR DELIMITED BY SPACE INTO L-input
        ELSE
            STRING WS-partialInputL DELIMITED BY SPACE WS-operand01 WS-partialInputR DELIMITED BY SPACE INTO L-input
    END-IF.
EXIT PROGRAM.
```
I still feel like there are too many variables in the subroutine and that there could probably be more intelligent reuse of variables for different functions but, for the sake of readability and simplicity, only WS-operand02 is reused for something other than its namesake, that being as a tally to check if the operations are completed.

NumberIsolation is a simple subroutine which just finds the next operator and cuts off the number. The necessity to REVERSE the strings is because the right partial input counts along whereas the left partial input counts backwards. SelectFunction performs whatever the operation is returning it as WS-operand01.

The end of the subroutine reconstitutes the input string so the parent program can loop effectively. There was some issue regarding negative operands and COBOL not putting them back into the string properly hence the entire existence of WS-numValue. This variable could likely be entirely eliminated and, if I go back to refine this and add the missing functionality, then I will do.

When ran the program looks something like this
```
Input:
4+8/-2*2-4*-3
1 instances of /
Calling Divide on +0008/-0002
Result is -0004
### Function is +4A-0004*2-4*-3
2 instances of *
Calling Multiplication on -0004*+0002
Result is -0008
Calling multiplication on -0004*-0003
Result is +0012
### Function is +4A-0008A+0012
2 instances of +
Calling addition on +0004A-0008
Result is -0004
Calling Addition on -0004A+0012
Result is +0008
Result is: 0008
```
The various steps are naturally explained as it goes along for debug purposes they obviously don't actually do anything.

### Extensions

While the program functions as a (very) basic calculator it is far from impressive. However, I felt comfortable closing the project because these extensions boil down to busywork moreso than actual programming or learning new features of COBOL.

Exponents would be very easy to add. In fact, adding any new simple function should be easy since it's just an extra loop, a new element of SelectFunction, and a subroutine for how to perform that function. For example, modulo could also very simply be added by practically copying division but just an extra line calculating the remainder rather by just multiplying the current quotient (since it is a floored integer) by the divisor and subtracting this from the initial dividend.
Parathenses should be a matter of

* checking that all ( are closed
* searching for the last (
* finding the next )
* finding the indices of those characters in the string
* unstringing the string either side of them
* performing the entire program on the encapuslated string
* repeating for as many ( as there are

Adding trigonometric functions would be a further extension which would require some actual restructuring since they are not a product of two operands and neither are they represented by a singular symbol. Spitballing some pseudocode,

* WS-trigFunction cycling like WS-operator does
* capturing the function within the parathenses of the function
* performing a normal calculation on that function
* solving the trig function
* returning that as a number

Which still is mostly recycling code but does introduce a new variable. I suppose it could check specifically for the functions rather than any variable but the use of a variable means that were arctan, sinh etc ever to be introduced that would be simpler since it would not require more restructing.

Again, none of this excites me particularly. The challenge was learning basic COBOL moreso than making a calculator: a calculator was just the given challenge. It took about three days of actual working over the course of maybe a week to get this far which was faster than I anticipated but after learning one programming language learning another is simple, even one which is sixty something years old.