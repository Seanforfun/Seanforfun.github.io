---
layout: post
title:  "Interview | Amazon Interview Questions"
date:   2019-06-14 10:33
author: Botao Xiao
categories: Interview
comment: true
description: In this article, I tried to find some Amazon interview questions online and try to make solutions to all of them.
---

## OA1
### Debugging
* 7 questions in 21 mins.
* Possible Bugs:
    1. Count Occurrences: Missing i++ at the end of while loop.
    2. Print Pattern: Missing { } to wrap up the two lines of code in a for-loop, causing the second line to be excluded from the for-loop.
        ```Java
        for(int i = 0; i < 100; i++)    // In this case, only a() will repeat, miss pattern {}
            a();
            b();
        ```
    3. Insert Sort Descending Order: change `>` (greater than) to `<` (less than), in this question, the order of the array is reversed.
    4. Select Sort: change `arr[min] > arr[x]` to arr[min] > arr[y].
    5. Reverse Array: change arr[len - 1] to arr[len - i - 1] and remove len += 1 at the end of the loop.
    6. Manchester Array(Not clear): change result=(a[i]==a[i-1]) to result=(a[i]!=a[i-1]), same for ret[0].
    7. Check Grade: Return ABCD according to marks, change both of || to &&.
    8. Replace Even/Odd Values: i <= len and j <= len change into < 6.

* Tips for debugging
    1. Sorting functions: Check the order of sorting, check the if-else statement.
    2. TLE problems: Check the break loop statements.
    3. For-loop: Check the {}.

### Reasoning
#### Pattern Search
* Implication
    1. A:B -> C:?
    2. QPS:TSV -> IHK:? Result: LKN, every character + 3
    3. 46:64 -> 82:? Result 1. 28, sum for each character is 10. Result 2. 64 - 46 = 18, 100
    4. EAGLE:FZHKF -> THANKS:?  Result: UGBMLR, odd number +1 and even number -1.
    5. FASTER:HCUVGT -> SLOWER:? Result: UNQYGT, all values +2.
    6. 985:874 -> 763:? Result: 652 each number -1.
    7. 865:532 -> 976:? Result: 643 each number -1.
    8. ADBC : EHFG -> ILJK:? Result: MPNO, 4(like 1234) in a group and reorder to 1423.
    9. JOHN:LSNV -> MARK:? Result: OEXP, +2 +4 +6 +8.
    10. COMPUTER:PMOCRETU -> TELEVISION:? Result: VELETNOISI, reverse first half and second half.
    11. A17R:D12P -> G7N:? Result: J2L, the first character increase by 3(+3), the middle number drops 5, the last letter = first letter + middle number, G + 3 = J, 7 - 5 = 2, last = J + 2 = L.
    12. COMPUTER:GKQLYPIN -> SENATE:? Result: WARWXA, odd +4, even -4.
    13. KPQR:LRTV -> DGHY:? Result: EIKC, +1 +2 +3 +4
    14. ACFJ:CEHL -> PRUY:? Result: RTWA, +2
    15. VAILANT:UBKJZOS -> TRANSCEND:? Result: SSCLRDGLC, in term of 4 letters -1 +1 +2 -2, last letter is -1.
    16. 27:24 -> 64:? Result: 60, 24 = 3 * 3 * 3 - 3 = 27 - 3, 64 - 4 = 4 * 4 * 4 - 4 = 60.
    17. MQD:KRK -> SWM:? Result QTX , MQD is 13, 17, 4, rotate KRK to RKK(18, 11, 11), operation is +5 -6 +7.
    18. ASSERTIVENESS-> SENSSAEVISTRE : MULTINATIONAL -> ? Result: ANOLUMITALNIT, fine the mapping of reorder.
    19. DELHI:CDAGH->MUMBAI:? Result: CJCAZH, for each index number minus 1. for example (23 - > 12)

* Find Exception (Given A B C D, find which one is different from the other three.)
    1. BGL DIN MRW HLR: HLR, difference between first and second is 5 except for HLR.
        ```
        BGL         DIN          MRW        HLR
        2 7 12      4 9 14      13 18 23    8 12 18
        ```
    2. PRS TVX FIK LME
    3. JLP LNT TVZ DFJ:

## Reference
1. [Amazon OA](https://wdxtub.com/interview/14520850399861.html)
2. [Amazon Online Assessment 1](https://aonecode.com/getArticle/215)