---
layout: post
title:  "Interview | Amazon Interview Questions"
date:   2019-06-14 10:33
author: Botao Xiao
categories: Interview
comment: true
description: In this article, I tried to find some Amazon interview questions online and try to make solutions to all of them.
---
# Amazon Interview Questions

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

### Logic
1. If northwest becomes east, northeast becomes south, and so on, what does southeast become? 
    * West: rotate 135 degree clockwise.

2. Lily can't find her home, she is 25 yards southwest of her home, then she walked 20 yards toward north, where is her home from her now? 
    * 15 yards, East. NEED TO RECONSIDER.

3. Facing north, Rufus walked 15 meters to the left. Then made an about-turn and walked 30 meters. Where is he now? 
    * (-15, -30) to the original point. Or (15, 0) ?

4. Jack walked 4 miles south-east, 8 miles west and then 4 miles northwest. Which direction he stands from the original spot? 
    * West

5. Facing North, Jack walked 20 miles to the left, 10 miles to the right and then 30 miles to the left. Which direction is he from the original spot?
    * North-West
    
6. Jack walked 5 yards South, 4 yards West, 7 yards South, 4 yards East and then 5 yards North. Where is he from the original spot? 
    * (0, -7)

7. 66 people are in a 3-level building. The second level has more people than any of the other levels. It is known that one of the levels has 21 people in it and the second floor has 2 people more than the first floor. Question: How many people is in the second floor. 
    * 23, from 1 to 3 is: 21, 23, 22

8. A sister is N years younger than her brother. Brother was born 1988. What you can learn from the given information.
    * She is born in year i988 + N and we can find her age, and there are at least 2 childs in her family.

9. Which two of the conditions tells the rank of Jack in the class. The two conditions are 1. 38 people are in the class. 2. 19 people rank behind Jack.
10. Which conditions tell which day Jack bought the car on. 1. 10/16=<the day < 10/19; 2. 10/17 <the day < 10/20
    * the day = 18.
11. A better sales person would be the one who is able to explain the features of his/her product in a simple manner.
12. Would you be able to tell how many balls are on the table knowing that if 7 balls got taken there will be no less than 23 balls left, and if 6 balls got added there will be no more than 20 balls on the table. 
    * No
13. Candidates for this business traversal are M1, M2, M3, M4, M5 and W1, W2, W3 (5 men and 3 women). The travel demands 3 men and 1 woman. M1 and M3 cannot go on the same trip. M4 and W2 cannot go on the same trip. If it’s been decided to send M2, M3 and W2 on this trip, who else you may send with them? 
    * M5
14. A - B denotes A plus B. A # B denotes A times B. A/B denotes A greater than or equal to B. A ? B denotes A less than B. Given expression: (V # X) / (V – X), X ? Y and Z/Y, which translates to V*X >= V+X, X < Y and Z >= Y, which of the two following expressions is true?

#### Indian Company
It has been proven by research that in India, a company which purchases saturation radio advertising will get maximum brand recognition.
1. A high degree of brand recognition will help a company win a higher share of the market.
2. Radio has wide listenership and companies intending to increase their awareness, should advertise it. 
3. For maximum brand recognition, a company need not spend on media channels other than radio publicizing. 
4. Brand recognition in India is more heavily dependent on where the brand advertises than the quality of its offering.
* Choose 2.

#### Fridge Sales
Which conditions were needed to know how many fridges were sold this year.
1. The number sold this year is 3 times of that of last year.
2. 40 were sold last year. 
* Choose both.

#### Environmental-friendly Enterprise
Decide whether to invite a company to Environmental Protection Conference.
The entrance bar:
1. have ECC Environmental Clearance Certificate
2. have at least 3 solar products
3. none of their products made from synthetic polymers
4. headquarter in Texas
5. have grade certified unit of its product
6. do not have legal dispute related to land or forest pending against them

#### PM Hire
A company is hiring PM. Hiring bar:
1. have CS major undergraduate
2. have MBA degree
3. GPA > 3.0 undergraduate
4. Report to HR if a candidate has no MBA degree but has 5 years of work experience
5. Report to HR if a candidate was not majored in CS but has 3 years of CS experience

A candidate studies mechanical engineering in university. GPA 4.0. Did not go to MBA. Worked 5 years in Google as a mechanical engineer and 3 years as a software engineer. What to do with the candidate?
1. Hire
2. Dismiss
3. Insufficient conditions
4. Report to HR

* Choose 4, meet number 5.

#### Hire
Now have some additional hiring rules on candidates:
1. have master’s degree. GPA must be A.
2. have 2 years or more work experience
3. if1 is not met then report to director

A candidate has 3 years of work experience, majored in CS in university and has master’s degree and MBA. His GPA undergraduate is A-. What to do with the candidate?
* Report to director.

#### Hire
Now again given more rules:
1. Master in commerce and at least B in GPA / have CPA, report to M director if this is not met
2. 25 > Age > 20
3. Fluent in English and Spanish
4. Pay a $125 deposit, report to chairman if this is no met
5. Promise to work 5 years for the company
* 1 and 4 happened in the example given.from

#### Delivery Charges

The following are the details of the procedure of deciding delivery charges for goods bought from ABC company. The customers:
1. are divided into two categories: those who have sales region code of 10 or above into one category and those with a code less than 10 into another
2. must have bought goods worth $500 or more in the previous month.
3. must not have dealership of any other similar company.
4. must not have availed bulk discount before
5. must have been provided a special discount of 5% or less than that in the previous dealings
6. must have been regularly ordering for more than 3 years
however,
1. if the customer fulfills all the conditions except (2), and if the sales region code is less than 10, delivery charges of $10 would be levied. Delivery charges of $8 would be levied for a code more than 10
2. if the candidate fulfills all the conditions except (3), and if the sales region code is less than 10, deliver charges of $5 would be levied. Deliver charges of $12 would be levied for a code more than 10.
3. If the customer does not fulfill 2 or more of the conditions stated above, then he/she would have to pay delivery charges of $30 irrespective of the sales region.

Jacob is a customer whose sales region code is 14. He had bought goods worth $150 from ABC company in June. He does not have dealership of any other similar company. He has never been provided any bulk discount or special discount.
1. He need not pay any delivery charges
2. He would have to pay $30 as delivery charges
3. has to pay $10
4. has to pay $8
5. data insufficient
* Choose 5, we don't know if he meet 6.

#### Delivery fee
Emma is a customer whose sales region code is 08. She has been regularly ordering goods from ABC company for more than 4 years. She has also purchases goods worth $150 in the previous month. She has never been provided with any bulk discount, but has been given a special discount of 2%. However, she has dealership of some other similar company.
1. She need not pay any deliver charges
2. She would have to pay $30 as delivery charges
3. She has to pay $10
4. She has to pay $12.
5. Data insufficient
* choose 2.

#### Coordinators
There are four coordinators named Lily, Cathy,Mary and Nina. Each coordinator is at a different corner of the rectangle meeting hall. A coffee vending machine is situated at one of the corners and a restroom at another corner of the meeting hall. Lily and Cathy are at either sides of the white board, which is situated at the center of the side which is opposite to the side at whose corners the coffee vending machine and the restroom are located. Coordinator Mary is not at the corner where the restroom is located.
Which of the following cannot be true?
1. Lily is not on the side of the hall where the white board is placed
2. Nina is adjacent to the restroom at one corner
3. Cathy is at the corner, adjacent to the coffee vending machine.from aonecode.com
4. Mary is adjacent to the coffee vending machine, at one corner of the hall
5. Lily is at the corner, adjacent to the coffee machine
* Choose 1, L and C must be at both side of the whiteboard.

#### Manufacturer
A manufacture company has 8 products and 4 divisions. Four divisions are lead by Alan, Betty, Cathy, Diana. The 8 products are: mixer, iron, water pump, geyser, juicer, blender, grinder, and heater. Each division produces 2 products, no 2 divisions produces the same product. Diana’s division produced Geyser, Cathy’s division produces water pump. Mixer and iron are produced by division lead by Alan and Betty respectively. The division that produces mixer doesn’t produce blender.

Four questions:
1. if the division that produces mixer doesn’t produce juicer, which of the following statement is true? (did not catch the statements)
2. if Alan produces mixer and heater, what does Betty produce. (iron)
4. if the division that produces mixer also produces juicer,how many ways are there for product pairs? (3! = 6)

#### Round Table
A round table sits 8 people A, B, C, D, E, F, G, H. F is two seats to the right of C. A and E sit by the sides of G. B and H are right facing each other.
- who is facing D ?
- G
- which two people sit in front of each other? (multiple choices)
- choose D and G
- who sits next to D ?
- C
- A and B is not adjacent. F is facing A. What is a possible counter clockwise sequence of people sitting at the table.
- AHCDFBEG

Sales Planning

Sales drives in big organizations, many a times, fall flat on the face. A research showed that an average buyer remembers only 20% of the things discussed during a sales call. The saddest part is that the sales team doesn’t get to choose what those 20% of things would be. The world today is cluttered with information and thus it is essential that the sales team represents their product/service in the best possible manner. It is like answering questions that children ask. Expect and out of context questions and reply to each one of them, patiently, in a way that the customers understand the intricacies. You can use technical terms to explain your product and its features. No doubt, it will be an accurate methodology but certainly not the right one. Simplify your message and see how well your client remembers you and your presentation when you meet him to finally close the deal.
1. A regular buyer would remember more than 20% of the details after a sales meeting
2. A customer is as gullible as a child and hence may ask many questions.
3. A better sales person would be the one who is able to explain the features of his/her product in a simple manner.
4. If you simplify your message, the customer would remember your entire presentation.
* Choose 3

#### Developer Recruit
An IT company has decided to recruit software developers. Conditions for selections of a candidate are as follows:
1. Should have at least a bachelor’s degree in engineering
2. Should have scored at least 60% marks in his/her bachelor’s degree and 80% marks in 12th grade.from aonecode.com
3. Must have at least 1 year’s work experience
4. Should be willing to sign a bond of 2 years
5. Should not be more than 28 years and not less than 21 years of age as on 01.02.2012from aonecode.com
However,
1. Candidates who fulfill all conditions except (1), but have obtained 75% in their bachelor’s degree (any computer applications degree like BCA) and have at least 3 years of work experience, may be referred to the Director
2. Who fulfill all conditions except (4), but are willing to pay an amount of $1000 as security deposit should be referred to the President
3. Who fulfill all conditions except (3), but are IT engineers may be referred to Deputy General Manager.from aonecode.com
Alexander is an IT engineer with 65% marks in his bachelor’s degree and 88% marks in 12th grade. He completed his bachelor’s degree in engineer, in 2007 and immediately started working in a private firm. He is not ready to sign a bond but doesn’t mind paying a sum of $1000 as security deposit. He was 26 years old as on 01.01.2012.from aonecode.com
1. He should not be recruited
2. He should be recruited
3. He should be referred to the President
4. He should be referred to the Deputy General Manager
5. Data insufficient
* Choose 3

#### Ionization Energy
Ionization energy decreases with the increasing size of metal atom. Out of cesium, lithium, potassium and sodium, which will have the lowest ionization energy?
Statements: I) Lithium has the smallest size. II) The size of potassium and cesium is greater than that of lithium.
1. I alone is sufficient for answering the problem question
2. II alone is sufficient for answering the problem question
3. Both together are sufficient for answering the problem question
4. Both together are not sufficient for answering the problem question
5. Either of statement is sufficient for answering the problem question

## Reference
1. [Amazon OA](https://wdxtub.com/interview/14520850399861.html)
2. [Amazon Online Assessment 1](https://aonecode.com/getArticle/215)