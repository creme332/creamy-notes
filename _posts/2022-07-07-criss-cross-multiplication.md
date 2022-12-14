---
title: Understanding the criss-cross multiplication algorithm
categories : [Maths, Programming]
tags :  [algorithms, c++]
description: A concise explanation of the Criss-Cross multiplication algorithm and its implementation in C++.
math : true
comments: true
img_path: /assets/vedic/
---
  
Back in 2019, I stumbled upon the Criss-Cross multiplication algorithm on [Youtube](https://www.youtube.com/watch?v=JhGzbN5YuPo). It comes from Vedic mathematics, a compendium of tricks for increasing the speed of mathematical calculations. The distinguishing feature of this algorithm is the fact that it can be used for mental calculations and it is much faster than the grade-school multiplication algorithm we all learnt.
 
The implementation of the Criss-Cross multiplication algorithm in C++ can be done in less than [50 lines of code](https://github.com/creme332/big-integer-vedic-multiplication-algorithm/blob/main/Vedic%20Algorithm/VedicMultiplicationAlgorithm.cpp) and it manages to multiply 1000-digit numbers (in string format) almost instantly. Moreover, when multiplying numbers having less than 1000 digits, it is nearly twice as fast as the [Karatsuba algorithm](https://en.wikipedia.org/wiki/Karatsuba_algorithm#:~:text=The%20Karatsuba%20algorithm%20was%20the,faster%2C%20for%20sufficiently%20large%20n.), one of the most popular fast multiplication algorithms in computer science.
 
# Algorithm

There is a sequence of patterns to follow when carrying out the single digit products.
 
1. For each pattern, compute the sum of all the required single digit products and add the carry from the previous step, if any, to this sum .
2. Prepend the units digit of the sum to our answer so far and carry over the other digits to the next step.
3. Move to next pattern.
 
## Pattern for 2-digit multiplication

> In the following GIFs, the dots represent the digits and each line represent the product of 2 digits.
{: .prompt-info }

<img src="2x2%20multiplication.gif" height ="300" width="400" alt = "2x2 criss-cross multiplication pattern">

<img src="2x2 multiplication example.gif" height ="300" width="400" alt = "2x2 criss-cross multiplication example">

<details>
  <summary>Detailed explanation for $29 \times 12$</summary>
  <li>
Step 1 : Multiply the digits in the right-most column (\( 9\times2 \)) to obtain \( 18 \).</li>
   <li>

Step 1.1 : Write down only unit digit (8 in this case) and carry over the digit in the tens place (1 in this case). The answer so far is \( 8 \) and the carry is \( 1 \).</li>

<li>
Step 2 : There are two single-digit multiplications (\( 2\times2 \) and \( 9\times1 \)) to carry out. Sum these products to obtain \( 13 \) and add the carry (which is 1) from the previous step. The final sum is \( 14\).</li>

  <li>
Step 2.1 : Write down only unit digit (4 in this case) and carry over the digit in the tens place (1 in this case). The answer so far is \( 48 \) and the carry is 1.</li>

  <li>
Step 3 : Multiply the digits in the left-most column (\( 2\times1 \)) to obtain 2. Add carry from previous step to current sum to obtain 3. </li>

  <li>
Step 3.1 : Since there are no more steps, prepend the current sum to our answer. Final answer is \( 348 \). </li>
</details>
  
## Pattern for 3-digit multiplication

<img src="3x3%20multiplication.gif" height ="300" width="400" alt = "3x3 criss-cross multiplication pattern">

<img src="3x3%20example.gif" height ="300" width="400" alt = "3x3 criss-cross multiplication example">
 
## Pattern for 4-digit multiplication

<img src="4x4%20multiplication.gif" height ="300" width="400" alt = "4x4 criss-cross multiplication pattern">

## Pattern for 5-digit multiplication

<img src="5x5%20multiplication.gif" height ="300" width="400" alt = "5x5 criss-cross multiplication pattern">

## Things to note

> Make use of symmetry to remember the sequence of patterns. For example, notice that the last $n/2$ patterns are simply a reflection in the vertical axis of the first $n/2$ patterns.
{: .prompt-tip }

> When the 2 numbers being multiplied have different lengths, simply prepend zeros to the smaller number to make it the same length as the larger number. For example, the Criss-Cross multiplication of $512323$ and $32$ is the same as the Criss-Cross multiplication of $512323$ and $000032$. In such a case, you will notice that there will be fewer single-digit products to carry out.
{: .prompt-tip }
 
# Proof

Even though I have not found a formal proof for the correctness of the Criss-Cross algorithm, we can get some insights into why it works by comparing it side-by-side with the grade-school algorithm : 

<img src="Proof.gif" height ="300" width="400" alt = "A gif showing similarity between criss cross multiplcation and grade school multiplication">

> We are doing the exact calculations as the grade-school multiplication algorithm. The difference lies in the order in which these calculations are carried out : the Criss-Cross algorithm uses a more efficient order which allows us to work with smaller numbers and gradually build the answer at each step.
{: .prompt-info }

# Code
```c++
std::string vedic(std::string a, std::string b) {
    if (a == "0" || b == "0")return "0"; 
    if (b.length() > a.length()) { std::string t = b; b = a; a = t; }

    const long long asize = static_cast<long long>(a.size());
    const long long bsize = static_cast<long long>(b.size());
    long long totalsteps = asize + bsize - 1;
    std::string ans = "";
    long long max = asize - 1, min = bsize - 1; 
    long long sum = 0, carry = 0;
    long long lines = 0; //number of multiplications at each step

    //Each step finds 1 digit of the answer
    for (long long step = 1;step <= totalsteps; step++) {
        sum = 0; //reset sum
        if (step <= asize) { lines = step; }
        else { lines = asize - (step - asize); }
        if (min < 0) {
            min = 0;
            max--;
            if (step < asize) {
                lines -= step - bsize;
            }
            else {
                lines -= asize - bsize;
            }
        }
        long long i = max, j = min;
        for (long long k = 0;k < lines;k++) {
            sum += (a[i] - '0') * (b[j] - '0');
            i--; j++;
        }
        sum += carry;
        carry = sum / 10;
        ans = std::to_string(sum % 10) + ans;
        min--;
    }
    if (carry != 0) ans = std::to_string(carry) + ans; 
    return ans;
}
```
> Time complexity : $\mathbb O (n^2)$
>
>Space complexity : $\mathbb O (1) $
{: .prompt-info }


## Explanation
```c++
ll i = max, j = min;

for (ll k = 0; k < lines; k++) {
    sum += (a[i] - '0') * (b[j] - '0');
    i--; j++;
}
```

$min$ and $max$ keep track of the region within which the cross-multiplication will take place.
 
 <img src="3x3%20pointer.gif" height ="300" width="400" alt = "2x2 criss-cross multiplication pointer explanation gif">
 
> $ i $ :  pointer used to iterate over $a$ within the range $[min, max]$.
> 
> $ j $ : pointer used to iterate over $b${: .filepath } within the range $[min, max]$.
{: .prompt-info }

 
### Case 1 : Both $a$ and  $b$ have $n$ digits
This is the simplest case. The total number of steps required is $2n-1$. (There are $n$ patterns +  $n$ reflected patterns. However, the $n$-th pattern is counted twice so we minus $1$.)
 
The number of single digit product (or the number of lines drawn) at each step can be found as follows :
```c++
if (currentstep <= n) {lines = currentstep;}
else {lines = n - (currentstep - n);} // or 2*n - currentstep
```
 
| Step number | Number of single digit product required |
| :---        |    :----:   |
|$ 1 $     | $1 $      |
|$ 2  $ |$ 2   $     |
|$ 3 $  | $3 $       |
|$ 4  $ | $4  $      |
|$ ... $  |$ ... $       |
|$ n $  | $n    $    |
| $n+1$  | $n-1$      |
| $n+2  $ | $n-2 $       |
| $n+3$   | $n-3  $      |
| $... $  | $...$        |
|$ 2n-1 $  |$ 1 $       |
 
At each step, we will compute the sum of all the 1-digit multiplications required. From this sum we will obtain the carry for the next step and a digit of our answer.

### Pseudocode for Case 1
```
Initialise the total number of steps
Initialise min and max to n-1
 
For each step
        Determine the number of 1-digit multiplication to be carried out
        Initialise sum to 0
        When the step number becomes > n (or when min becomes negative), set min to 0 and decrement max.
       
        For each 1-digit multiplication
                Increment sum
                Modify counters to move to next 1-digit multiplication
       
        Add previous carry to sum
        Update carry and answer
        Decrement min
 
If carry is non-zero, concatenate it to the left of the answer.
Return answer
```
 
### Case 2 : $b$ has fewer digits than $a$
This case could be eliminated if we simply make both $a$ and $b$ the same length initially by adding leading zeros to $b$. However, this approach is time-consuming because there will be unnecessary and will introduce leading zeros in our final answer which must be removed.
 
To bypass this problem, we simply reduce the total number of steps and the number of single digit products at each step to eliminate cases where the 1-digit multiplication involves a leading zero.
 
The total number of steps is now found using :

$$ \boxed{\text{totalsteps = length(a) + length(b) - 1}}$$

For each step, we first find the expected number of 1-digit multiplication if $a$ and $b$ were the same length :
```c++
if (step <= asize) {
      lines = step;
}
else {
    lines= asize - (step- asize); // do not simplify as 2*asize might overflow
}
```
 
When all the digits of $b$ have been traversed, $min$ becomes negative. Previously when $a$ and $b$ were the same length, we exited when $min$ was negative. However, when they differ in length, we cannot exit. Now, each time $min$ reaches the start of $b$, we decrement $max$ and keep $min$ at index 0 of $b$.
```c++
if (min < 0) {
    min = 0;
    max--;
}
```
Then we reduce the number of 1-digit multiplication as follows :
 
```c++
if (min < 0) {
    min = 0;
    max--;
    if (step < asize) {
        lines -= step - bsize;
    }
    else {
        lines -= asize - bsize;
    }
}
```
<img src="2x2 multiplication example.gif" height ="300" width="400" alt = "2x2 criss-cross multiplication example">
 
The rest of the code for Case 2 is the same as that in Case 1.
 
# Comparison with Karatsuba
  
From the table in Case 1, we can deduce that the total number of single digit products carried out by the Criss-Cross algorithm is $$ n^2 + n - 1 $$.  In comparison, the total number of single digit products for the Karatsuba algorithm is $$ n^{\log_2 3} $$.

<img src="desmos-graph.png" height ="300" width="400" alt = "graph comparing time complexity of criss cross and karatsuba">
 
In theory, as the numbers being multiplied tend to infinity, the Karatsuba algorithm will perform better. However, for relatively small numbers, the Karatsuba algorithm actually performs worse.

In practice, I decided to compare both algorithms using a basic benchmark method (my results should be taken with a grain of salt). The code used for comparing  the Criss-Cross algorithm and the Karatsuba algorithm can be found [here](https://github.com/creme332/big-integer-vedic-multiplication-algorithm/blob/main/Testing/tester.cpp).

| Number of digits in each number |Number of test cases | Avg time for vedic() | Avg time of karatsuba() | % difference |
| :---             |:----:                 |:----:                |:----:                                   |:----:        |
| 100              | 1000                  | 6033                  | 343458                                  |        98.2  |
| 500              | 100                   |  2263                 | 131762                                |     98.2           |
| 1000             | 200                   |  172990               | 5794911                                  | 97.0            |
 
> The unit of time used is microseconds.
 
#  Final thoughts

## Why use the Criss-Cross algorithm
 
> Much simpler to use with pen and paper than the Karatsuba algorithm and the grade-school algorithm.
{: .prompt-tip }

> Can be used for mental calculations.
{: .prompt-tip }

> The risk of integer overflow is small as it carries out the sum of single-digit products.
{: .prompt-tip }

## Why not use the Criss-Cross algorithm
 
> Not suitable for multiplying numbers tending to infinity.
{: .prompt-danger }
