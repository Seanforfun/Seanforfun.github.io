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
    3. JLP LNT TVZ DFJ: Result: LNT, other are +2 +4.
    4. ABIJ DEHI MNQR STWX: Result: ABIJ, the difference between second and third character should be 3.
    5. ADP QTS HKR STE: Result: Not sure, ADP for its index are all perfect square and STE for difference between first and second should be 3.
    6. RHCAI OEST HNDA ADEH: Result: RHCAI, OEST->TOES, HNDA->HAND, ADEH->HEAD, rest are all body part.
    7. ADF MPR ILN EHJ: Result: MPR, it doesn't start with vowel.
    8. STV XYA KKT BDE: Result: KKT, contains 2 same character.
    9. 956 794 884 678: Result: 678, digits sum should be 20.
    10. 1,4,16 17,20,24 8,11,18 19,20,5: Result: 1,4,16, perfect square. Or 19,20,5 (20 - 19 is not 3 and 5 is not a multiplication of 3)
    11. AE5 DF6 HN14 KP2: Result: KP2, P != 2.
    12. HIK DGJ LPT SUW: HIK, difference between characters are not same.
    13. LKJI XYWV WVUT KJIH:

* Induction
    1. 2,3,7,8,13,14,?: Result: 20, difference between odd index value is +5 +6 +7.
    2. 0,1,1,2,4,8,? :Result: 16, sum of previous numbers
    3. 3, 6, 18, 108, ? : Result: 1944, multiplication of previous 2 numbers.
    4. 1,1,4,2,13,3,40,4,? : Result: 121, n = n -2 + 3^(n - 1).
    5. 3, 7, 13, 21, ?: Result: 31, +2, +4, +6...
    6. 5, 11, 19, 29, ? : Result: 41, +6, +8, +10, +12
    7. 0, 2, 6, 12, 20, ?: Result: 30, +2 +4...
    8. 5, 9, 16, 29, ? : Result: 54, n-1 * 2 - 1, n-1 * 2 - 2...
    9. 4, 12, 6, 18, 12, 36, 30, ? : Result: 90, pre * 3
    10. 1, 5, 7, ? :Result: 8 (1 + pow(2,2) = 5, 5 + pow(2,1) = 7, 7 + pow(2, 0) = 8)
    11. D, H, L, ? : Result: P, +4
    12. 10, 14, 23, 39, 64, ? :Result: 100, difference 1^2 ,2^2...
    13. 10, 74, 202, 394, ? : Result: 650, +64 +64*2 +64*3
    14. 2, 8, 5, 6, 8, ?, 11 :Result: 4, even indexed number -4.
    15. 16, 30, 46, 62, ? : Result: 82, +14 +16 +18...
    16. 1, 4, 27, 256, ? : Result: 2625 n^n
    17. 2, 5, 26, ? : Result: 677 pre^2 + 1

* Tips:
1. Get the letter <-> number table ready. It will save you a great amount of time!
2. When asking for an exception in the 4 strings given, convert the letters into numbers and find the pattern with the numbers.
3. Some times the odd indices follow a pattern and the even indices follow another pattern. Try looking at odds and evens separately.

## Reference
1. [Amazon OA](https://wdxtub.com/interview/14520850399861.html)
2. [Amazon Online Assessment 1](https://aonecode.com/getArticle/215)