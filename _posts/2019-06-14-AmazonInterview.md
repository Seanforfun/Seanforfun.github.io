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

### Coding
* Two sum closest:
    1. Question 1
        ```Java
        public class TwoSumClosest {
        
            private int[] twoSumClosest(int[] nums, int target){
                /**
                 * O(nlgn + n)
                 */
                int[] res = new int[2];
                int diff = Integer.MAX_VALUE;
                Arrays.sort(nums);  //O(nlgn)
                int low = 0, high = nums.length - 1;
                while(low < high){  //O(n)
                    int curSum = nums[low] + nums[high];
                    if(curSum > target){
                        high--;
                    }else if(curSum == target){
                        return new int[]{nums[low], nums[high]};
                    }else if(target - curSum < diff){
                        diff = target - curSum;
                        res[0] = nums[low];
                        res[1] = nums[high];
                        low++;
                    }else low++;
                }
                return res;
            }
            public static void main(String[] args) {
                int[] nums = new int[]{90, 85, 75, 60, 120, 150, 125};
                TwoSumClosest algorithm = new TwoSumClosest();
                int[] res = algorithm.twoSumClosest(nums, 220);
                System.out.println("Res: " + res[0] + " " + res[1]);
            }
        }
        ```
    
    2. Question 2
        ```Java
        public class FindOptimalWeights {
            private static int[] findOptimalWeights(int[] weights, int target){
                int len = weights.length;
                Arrays.sort(weights);
                int slow = 0, fast = len - 1;
                int[] result =  new int[2];
                int diff = Integer.MAX_VALUE;
                while(slow < fast){
                    int curSum = weights[slow] + weights[fast];
                    if(curSum == target){
                        return new int[]{weights[slow], weights[fast]};
                    }else if(curSum > target) fast--;
                    else {  // curSum < target
                        if(target - curSum < diff){
                            diff = target - curSum;
                            result[0] = weights[slow];
                            result[1] = weights[fast];
                        }
                        slow++;
                    }
                }
                return result;
            }
            public static void main(String[] args) {
                int[] nums = new int[]{90, 85, 75, 60, 120, 150, 125};
                int[] res = findOptimalWeights(nums, 220);
                System.out.println("Result: " + res[0] + " " + res[1]);
            }
        }        
        ```

* Search 2-D Matrix
    1. [74. Search a 2D Matrix](https://github.com/Seanforfun/Algorithm-and-Leetcode/blob/master/leetcode/74.%20Search%20a%202D%20Matrix.md)
    ```Java
    class Solution {
        public boolean searchMatrix(int[][] matrix, int target) {
            if(matrix == null || matrix.length == 0 || matrix[0].length == 0) return false;
            int height = matrix.length, width = matrix[0].length;
            int x = 0, y = width - 1;
            while(x >= 0 && x < height && y >= 0 && y < width){
                if(matrix[x][y] == target) return true;
                else if(matrix[x][y] < target){
                    x++;
                }else{
                    y--;
                }
            }
            return false;
        }
    }
    ```

* Sliding Window Maximum
    1. [Sliding Window Maximum](https://github.com/Seanforfun/Algorithm-and-Leetcode/blob/master/leetcode/239.%20Sliding%20Window%20Maximum.md)
    ```Java
    class Solution {
        public int[] maxSlidingWindow(int[] nums, int k) {
            if(nums.length < k || nums.length == 0) return new int[0];
            int[] result = new int[nums.length - k + 1];
            Deque<Integer> deque = new ArrayDeque<>();
            int i = 0, index = 0;
            for(i = 0; i < nums.length; i++){
                if(i - k >= 0 && !deque.isEmpty() && deque.peek() == i - k) deque.pollFirst();
                int add = nums[i];
                while(!deque.isEmpty() && add >= nums[deque.getLast()]){
                    deque.pollLast();
                }
                deque.addLast(i);
                if(i - k + 1 >= 0) result[index++] = nums[deque.getFirst()];
            }
            return result;
        }
    }
    ```

* MaximumMinimumPath
    ```Java
    public class MaximumMinimumPath {
        public static int maxMinPath(int[][] grid){
            if(grid == null || grid.length == 0 || grid[0].length == 0) return 0;
            int height = grid.length, width = grid[0].length;
            int[][] dp = new int[height][width];
            dp[0][0] = grid[0][0];
            for(int i = 1; i < height; i++)
                dp[i][0] = Math.min(dp[i - 1][0], grid[i][0]);
            for(int i = 1; i < width; i++)
                dp[0][i] = Math.min(dp[0][i - 1], grid[0][i]);
            for(int i = 1; i < height; i++){
                for(int j = 1; j < width; j++){
                    // find the max possible in the minimum path from up and left.
                     dp[i][j] = Math.max(dp[i - 1][j], dp[i][j - 1]);
                     // if current value is smaller, it will refresh the minimum value.
                     dp[i][j] = Math.min(dp[i][j], grid[i][j]);
                }
            }
            return dp[height -  1][width - 1];
        }
        public static void main(String[] args) {            
            System.out.println(maxMinPath(grid));
        }
    }
    ```
* TreeAplitude find max diff in paths from root to leaf
    ```Java
    public class TreeAplitude {
        private static class TreeNode{
            int val;
            TreeNode left, right;
        }
        private int result = Integer.MIN_VALUE;
        public int findAplitude(TreeNode root){
            if(root == null) return 0;
            dfs(root, root.val, root.val);
            return result;
        }
        private void dfs(TreeNode node, int min, int max){
            if(node == null){
                this.result = Math.max(result, max - min);
            }else{
                int nextMin = Math.min(min, node.val);
                int nextMax = Math.max(max, node.val);
                dfs(node.left, nextMin, nextMax);
                dfs(node.right, nextMin, nextMax);
            }
        }
    }
    ```

* Arithmetic Slices
    1. [413. Arithmetic Slices](https://github.com/Seanforfun/Algorithm-and-Leetcode/blob/master/leetcode/413.%20Arithmetic%20Slices.md)
    ```Java
    class Solution {
        public int numberOfArithmeticSlices(int[] A) {
            if(A == null || A.length < 3) return 0;
            int res = 0;
            int slow = 0, fast = 1;
            int diff = A[fast] - A[slow];
            fast++; // fast = 2 now.
            for(; fast < A.length; fast++){
                if(A[fast] - A[fast - 1] == diff){
                    if(fast - slow + 1 >= 3){
                        res += fast - 2 - slow + 1;
                    }
                    continue;
                }else{   // diff changes
                    slow = fast - 1;
                    diff = A[fast] - A[fast - 1];
                }
            }
            return res;
        }
    }
    ```

* Four Integer, given 4 numbers, find max of abs(res[0] - res[1]) + abs(res[1] - res[2]) + abs(res[2] - res[3])
    ```Java
    public class FourIntegers {
        public static  int[] findMax(int A, int B, int C, int D){
            int[] res = new int[4];
            res[0] = A;
            res[1] = B;
            res[2] = C;
            res[4] = D;
            swap(res, 0, 1);
            swap(res, 2, 3);
            swap(res, 0, 3);
            return res;
        }
        private static void swap(int[] A, int a, int b){
            int temp = A[a];
            A[a] = A[b];
            A[b] = temp;
        }
    }
    ```
    
## Reference
1. [Amazon OA](https://wdxtub.com/interview/14520850399861.html)
2. [Amazon Online Assessment 1](https://aonecode.com/getArticle/215)
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

### Social Hiring
1. Cost of assembly.
	```Java
	public class AssembleComponents {
		private int minTimeAssemble(List<Integer> times){
			if(times == null) return -1;
			else if(times.size() == 1) return times.get(0);
			else if(times.size() == 0) return  0;
			PriorityQueue<Integer> pq = new PriorityQueue<>();
			for(Integer time : times){
				pq.offer(time);
			}
			int res = 0;
			while (pq.size() > 1){
				int first = pq.poll(), second = pq.poll();
				res += first + second;
				pq.offer(first + second);
			}
			return res;
		}

		public static void main(String[] args) {
			AssembleComponents assembleComponents = new AssembleComponents();
			List<Integer> list = new ArrayList<>();
			list.addAll(Arrays.asList(8,4,6,12));
			System.out.println(assembleComponents.minTimeAssemble(list));
		}
	}
	```

### Create a BST using arry and find the distance between 2 nodes.
	```Java
	public class BSTNodeDistance {
		private static class Node{
			int v;
			Node left, right;
			public Node(int v){
				this.v = v;
			}
		}
		private Node root;
		private Map<Integer, List<Node>> adj;
		private Map<Integer, Node> nodeMap;
		public int findMinimumDistanceInBST(int[] arr, int a, int b){
			if(arr == null || arr.length == 0) return 0;
			adj = new HashMap<>();
			nodeMap = new HashMap<>();
			this.root = new Node(arr[0]);
			adj.put(arr[0], new ArrayList<>());
			nodeMap.put(arr[0], root);
			for(int i = 1; i < arr.length; i++){
				insert(root, arr[i]);
			}
			int step = 0;
			Queue<Node> q = new LinkedList<>();
			q.offer(nodeMap.get(a));
			Set<Integer> visited = new HashSet<>();
			visited.add(root.v);
			while(!q.isEmpty()){
				int size = q.size();
				for(int sz = 0; sz < size; sz++){
					Node cur = q.poll();
					if(cur.v == b) return step;
					List<Node> neighbours = adj.get(cur.v);
					for(Node next : neighbours){
						if(visited.contains(next.v)) continue;
						visited.add(next.v);
						q.offer(next);
					}
				}
				step++;
			}
			return -1;
		}
		private void insert(Node node, int cur){
			if(cur > node.v){
				if(node.right == null){
					node.right = new Node(cur); // next is current node.
					List<Node> list = adj.getOrDefault(cur, new ArrayList<>());
					list.add(node);
					adj.put(cur, list);
					List<Node> parent = adj.getOrDefault(node.v, new ArrayList<>());
					parent.add(node.right);
					adj.put(node.v, parent);
					nodeMap.put(cur, node.right);
				}else insert(node.right, cur);
			}else{
				if(node.left == null){
					node.left = new Node(cur); // next is current node.
					List<Node> list = adj.getOrDefault(cur, new ArrayList<>());
					list.add(node);
					adj.put(cur, list);
					List<Node> parent = adj.getOrDefault(node.v, new ArrayList<>());
					parent.add(node.left);
					adj.put(node.v, parent);
					nodeMap.put(cur, node.left);
				}else insert(node.left, cur);
			}
		}

		public static void main(String[] args) {
			BSTNodeDistance distance = new BSTNodeDistance();
			System.out.println(distance.findMinimumDistanceInBST(new int[]{5,6,3,1,2,4}, 2, 4));
		}
	}
	```

### Find the minimum city connection cost. => MST
	```Java
	public class CityConnection {
		private int[] uf;
		public int getMinimumCostToConstruct(int numTotalAvailableCities,    //6
													int numTotalAvailableRoads, //3
													List<List<Integer>> roadsAvailable,     //[[1,4], [4,5], [2,3]]
													int numNewRoadsConstruct,   //4
													List<List<Integer>> costNewRoadsConstruct){ //[[1,2,5], [1,3,10], [1,6,2], [5,6,5]]
			if(numTotalAvailableCities == 0 || numTotalAvailableRoads >= numTotalAvailableCities) return 0;
			if(numNewRoadsConstruct == 0) return -1;
			this.uf = new int[numTotalAvailableCities + 1];
			for(int i = 1; i <= numTotalAvailableCities; i++){
				uf[i] = i;
			}
			for(List<Integer> pair: roadsAvailable){
				union(pair.get(0), pair.get(1));
			}
			PriorityQueue<List<Integer>> pq = new PriorityQueue<List<Integer>>((a, b)->{
				return a.get(2) - b.get(2);
			});
			for (List<Integer> road: costNewRoadsConstruct){
				pq.offer(road);
			}
			int result = 0;
			while (!pq.isEmpty()){
				List<Integer> road = pq.poll();
				int city1 = road.get(0), city2 = road.get(1), cost = road.get(2);
				if(!connected(city1, city2)){
					result += cost;
					union(city1, city2);
				}
			}
			int cur = find(1);
			for(int i = 2; i <= numTotalAvailableCities; i++){
				if(find(i) != cur) return -1;
			}
			return result;
		}
		private int find(int i){
			if(uf[i] != i){
				uf[i] = find(uf[i]);
			}
			return uf[i];
		}
		private void union(int i, int j){
			int p = find(i);
			int q = find(j);
			uf[p] = q;
		}
		private boolean connected(int i, int j){
			return find(i) == find(j);
		}
	}
	```

### Closest K points
	```Java
	public class DeliveryStrategy {
		public List<List<Integer>> closestXdestinations(int numDestinations, List<List<Integer>> allLoacations, int numDeliveries){
			if(allLoacations == null) return null;
			PriorityQueue<List<Integer>> pq = new PriorityQueue<List<Integer>>((a, b)->{
			   return a.get(0) * a.get(0) + a.get(1) * a.get(1) - b.get(0) * b.get(0) - b.get(1) * b.get(1);
			});
			for(List<Integer> location: allLoacations){
				pq.offer(location);
			}
			List<List<Integer>>  result = new ArrayList<>();
			while(numDeliveries-- > 0){
				result.add(pq.poll());
			}
			return result;
		}
	}
	```

### Find the minimum path to reach the end. BFS
	```Java
	public class MinimumDistance {
		private static final int[][] dir = new int[][]{{1, 0}, {-1, 0}, {0, 1}, {0, -1}};
		public int minimumDistance(int numRows, int numCols, List<List<Integer>> area){
			if(area.get(0).get(0) == 9) return 0;
			Queue<int[]> q = new LinkedList<>();
			Set<Integer> visited = new HashSet<>();
			visited.add(0);
			q.offer(new int[]{0, 0});
			int step = 0;
			while (!q.isEmpty()){
				int size = q.size();
				int tx = 0, ty = 0;
				for(int sz = 0; sz < size; sz++){
					int[] cur = q.poll();
					if(area.get(cur[0]).get(cur[1]) == 9) return step;
					for (int d = 0; d < 4; d++){
						tx = cur[0] + dir[d][0];
						ty = cur[1] + dir[d][1];
						if(tx >= 0 && tx < numRows && ty >= 0 && ty < numCols && area.get(tx).get(ty) != 0 && !visited.contains(tx * numRows + ty)){
							visited.add(tx * numRows + ty);
							q.offer(new int[]{tx, ty});
						}
					}
				}
				step++;
			}
			return -1;
		}
	}
	```

### Partitional Lables Greedy
	```Java
	public class PartitionLabels {
		public List<Integer> partitionLabels(String S) {
			int[] last = new int[26];
			char[] arr = S.toCharArray();
			for(int i = arr.length - 1; i >= 0; i--){
				if(last[arr[i] - 'a'] == 0) last[arr[i] - 'a'] = i;
			}
			List<Integer> res = new ArrayList<>();
			int slow = 0, fast = 0, max = last[arr[0] - 'a'];
			while(fast < arr.length){
				while(fast <= max){
					max = Math.max(last[arr[fast++] - 'a'], max);
				}
				res.add(fast - slow);
				slow = fast;
				if(slow < arr.length) max = last[arr[slow] - 'a'];
			}
			return res;
		}
	}
	```

### Reorder Logs
	```Java
	public class ReorderLogs {
		public String[] reorderLogFiles(String[] logs) {
			if(logs == null) return null;
			int len = logs.length;
			List<String> letterLogs = new ArrayList<>();
			List<String> digitLogs = new ArrayList<>();
			for(String log: logs){
				String[] tokens = log.split(" ");
				if(Character.isDigit(tokens[1].charAt(0))){
					digitLogs.add(log);
				}else letterLogs.add(log);
			}
			Collections.sort(letterLogs, (a, b)->{
				String subA = a.substring(a.indexOf(' ') + 1);
				String subB = b.substring(b.indexOf(' ') + 1);
				int cmp = subA.compareTo(subB);
				return cmp != 0 ? cmp : a.substring(0, a.indexOf(' ')).compareTo(b.substring(0, b.indexOf(' ')));
			});
			String[] res = new String[len];
			int count = 0;
			while(count < letterLogs.size()){
				res[count] = letterLogs.get(count);
				count++;
			}
			int temp = count;
			while(count < len){
				res[count] = digitLogs.get(count - temp);
				count++;
			}
			return res;
		}
	}
	```

### Two sum closest
	```Java
	public class TwoSumClosest {
		public List<List<Integer>> closestPath(int mileage, List<List<Integer>> forward, List<List<Integer>> backward){
			if(forward == null || backward == null || forward.size() == 0 || backward.size() == 0) return null;
			int diff = Integer.MAX_VALUE;
			List<List<Integer>> result = new ArrayList<>();
			for(int i = 0; i < forward.size(); i++){
				for(int j = 0; j < backward.size(); j++){
					int f = forward.get(i).get(1), b = backward.get(j).get(1);
					if(f + b >= mileage) continue;
					else if(mileage - f - b < diff){
						diff = mileage - f - b;
						result.clear();
						List<Integer> cur = new ArrayList<>(Arrays.asList(i + 1, j + 1));
						result.add(cur);
					}else if(mileage - f - b == diff){
						List<Integer> cur = new ArrayList<>(Arrays.asList(i + 1, j + 1));
						result.add(cur);
					}
				}
			}
			return result;
		}
	}
	```

## Reference
1. [Amazon OA](https://wdxtub.com/interview/14520850399861.html)
2. [Amazon Online Assessment 1](https://aonecode.com/getArticle/215)