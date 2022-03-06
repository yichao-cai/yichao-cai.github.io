---
layout: post
title: "Letter decoder"
date:   2022-02-21 16:00:00 +0800
tags: algorithm
---

# Question description

Given the mapping a = 1, b = 2, ... z = 26, and an encoded message,
count the number of ways it can be decoded. For example, the message
\'111\' would give 3 combinations, since it could be decoded as \'aaa\',
\'ka\', and \'ak\'. You can assume that the messages are decodable. For
example, \'001\' is not allowed.

<!--more-->

# Solution

## Dynamic programming

If we consider a string of length i, the number of ways that can be
decoded can be written in a formula (let\'s consider starting from the
end of the string and going backwards):

f(0, i) = f(0, i-2) \* f(i-2, i) + f(0, i-1) \* f(i-1, i)

where function f(x, y) returns the number of possible ways to decode a
string from position x to position y.

The reason why we consider i-2 and i-1 position is that a two digit
string can have two ways of decoding. One is to decode individual digit,
while the other is to decode the two digits together.

For the edge cases, we can consider two scenarios:

1.  When a string only have 1 digit, there is only one way of decoding.
    For example, \'9\' is decoded as \'i\'.

2.  When a string have 2 digits, there are two ways of decoding:

    -   If the second digit is \'0\', then there would be only one way
        of decoding. For example, \'10\' is decoded as \'j\'.
    -   If the first digit is \>= 2, then there would be only onw way of
        decoding. For example, \'36\' is decoded as \'cf\'.
    -   For the rest of the cases, there are 2 ways of decoding. For
        example, \'11\' can be decoded as \'aa\' or \'k\'.

## Code to return decoded combinations

We can have the following codes to return the results. The decoder
function is to take care of the results when reaching edge cases, while
the messageDecoder function is to do the dynamic programming steps.

``` python
def decoder(msg):
    # Edge cases of length 1 or 2
    if(msg[0] == "0"):
        return 0
    elif(len(msg) == 1):
        return 1
    elif(len(msg) == 2):
        if(msg[1] == "0"):
            return 1
        elif(int(msg) > 26):
            return 1
        else:
            return 2

def messageDecoder(msg):
    if(len(msg) <=2):
        return decoder(msg)
    else:
        return (messageDecoder(msg[:-2]) * decoder(msg[-2:])) + (messageDecoder(msg[:-1]) * decoder(msg[-1:]))

messageDecoder("101")  # Returns 1
messageDecoder("111")  # Returns 4
messageDecoder("999")  # Returns 2
messageDecoder("1111")  # Returns 8
messageDecoder("1234")  # Returns 6
```

If we look at the results, we can notice that the number of possible
decoding ways is not as we expected. The reason being that the code only
consider the combination of substring of length 1 or 2, but does not
remember the intermediate stage, where duplicated results are returned.

For example, for string \'111\', it would be divided into two
combinations of letters: \'1\' and \'11\', \'11\' and \'1\'. These two
combinations would both return 2 possible decoding ways. In this case,
the decoded string \'aaa\' would be considered twice in the two separate
function calls. Therefore, the final results is not 3 but 4.

## Code to return decoded strings

We can further fix this unexpected issue by returning decoded string
instead of decoded ways.

``` python
import itertools
import string

# Dictionary to map string to letter.
letterMap = dict(zip([str(i) for i in range(1,27)],
                     [i for i in string.ascii_lowercase]))

def decoder2(msg):
    # Edge cases of length 1 or 2
    # Return decoded letter instead.
    if(msg[0] == "0"):
        return []
    elif(len(msg) == 1):
        return [letterMap[msg]]
    elif(len(msg) == 2):
        if(msg[1] == "0"):
            return [letterMap[msg]]
        elif(int(msg) > 26):
            return ["".join([letterMap[i] for i in msg])]
        else:
            return ["".join([letterMap[i] for i in msg]), letterMap[msg]]


def messageDecoder2Recur(msg):
    if(len(msg) <=2):
        return decoder2(msg)
    else:
        # Concatenate returned letters and append in the returned list.
        ret = []
        ret.append([''.join(i) for i in list(itertools.product(messageDecoder2Recur(msg[:-2]), decoder2(msg[-2:])))])
        ret.append([''.join(i) for i in list(itertools.product(messageDecoder2Recur(msg[:-1]), decoder2(msg[-1:])))])
        # Return unique decoded letters.
        return list(set(list(itertools.chain(*ret))))

def messageDecoder2(msg):
    # Helper function to get the number of uniquely decoded letters.
    return(len(messageDecoder2Recur(msg)))

messageDecoder2("101")  # Returns 1
messageDecoder2("111")  # Returns 3
messageDecoder2("999")  # Returns 1
messageDecoder2("1111")  # Returns 5
messageDecoder2("1234")  # Returns 3
messageDecoder2("27")  # Returns 1
```

This way we creat a nested string list of uncertain layers. Thus the
final merging step may be time consuming and also take up more memory.

## Code to return decoded strings (improved)

We can create a list of the same length as the input string. Store the
number of possible decode ways at each location. For example,
`decodeNum[i]` is the number of possible decode ways for string
`msg[i:]`.

``` python
def messageDecoder3(msg):
    decodeNum = [0] * len(msg)

    # Move pointer forward.
    for i in range(len(msg)-1, -1, -1):
        # Process the last digit.
        if(i == len(msg) -1):
            if(int(msg[i]) == 0):
                decodeNum[i] = 0
            else:
                decodeNum[i] = 1
        else:
            if(int(msg[i]) == 0):
                decodeNum[i] = 0
            elif(int(msg[i]) > 3 or int(msg[i:i+2]) > 26):
                decodeNum[i] = decodeNum[i+1]
            elif(int(msg[i]) in [1, 2] and decodeNum[i+1] == 0):
                try:            
                    decodeNum[i] = decodeNum[i+2]
                except IndexError:
                    # If at the second last digit, decodeNum[i+2] would lead to index out of range.
                    # In such cases, the only possibility is 1 because the last two digits is '10' or '20'.
                    decodeNum[i] = 1
            else:
                try:
                    decodeNum[i] = decodeNum[i+1] + decodeNum[i+2]
                except IndexError:
                    # If at the second last digit, decodeNum[i+2] would lead to index out of range.
                    # In such cases, there is one more possible way to decode the digits. i.e. for '11', we have 2 ways.
                    decodeNum[i] = decodeNum[i+1] + 1

    return(decodeNum[0])
```

In this way we reduce the memory usage and it runs faster since we
don\'t need to do the list flattening and string concatenate steps. You
can rewrite the try except clauses into a new elif clause for situation
where `i == len(msg) - 2`, but that way would make the conditions look
repetitive.

## LeetCode link

A same question is described in LeetCode as well
([link](https://leetcode.com/problems/decode-ways/)).
