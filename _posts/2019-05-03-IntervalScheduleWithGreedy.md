---
layout: post
title:  "Algorithm | Interval Schedule"
date:   2019-05-03 21:24
author: Botao Xiao
categories: Algorithm
comment: true
description:
---
### Introduction
When I am doing interval schedule related Leetcode questions, I always use sort and queue related methods, and when I am doing new questions, I didn't find a systematical way to solve this question.

When I searched this kind of new questions online, I found all of them are actually Greedy question. So I wrote this article to conclude this question.

### Meeting rooms
1. Find if there are overlaps between the intervals.
  * [252. Meeting Rooms](https://github.com/Seanforfun/Algorithm-and-Leetcode/blob/master/leetcode/252.%20Meeting%20Rooms.md)
  * Step 1: Sort all the intervals according to the start time.
  * Step 2: Starting from the second interval, we check if the beginning time is later than the previous end time. If false, it means there is a overlaps.
  ```Java
  class Solution {
      private static final Comparator<int[]> cmp = new Comparator<int[]>(){
          @Override
          public int compare(int[] a, int[] b){
              return a[0] - b[0];
          }
      };
      public boolean canAttendMeetings(int[][] intervals) {
          // Sort all intervals according to the starting time
          Arrays.sort(intervals, cmp);
          // Check for all of the intervals, check if there is overlap.
          for(int i = 1; i < intervals.length; i++){
              if(intervals[i][0] < intervals[i - 1][1])
                  return false;
          }
          return true;
      }
  }
  ```

2. Find least rooms for the meetings.
  * [253. Meeting Rooms II](https://github.com/Seanforfun/Algorithm-and-Leetcode/blob/master/leetcode/253.%20Meeting%20Rooms%20II.md)
  * This question is Greedy, greedy for the least rooms to be used, in another word, we want each room to be used as much as possible.
  * Step 1: Sort all of the meeting according to there starting time.(Since we want to make full use of the rooms, means we want to use the room as early as possible).
  * Step 2: We add all intervals to the priority queue, which orders all intervals according to all intervals ending time. We poll all the intervals that ends before current interval. It means we can re-use current room for current meeting interval.
  * Size of priority queue means the room we are using at current time.
  ```Java
  class Solution {
      public int minMeetingRooms(int[][] intervals) {
          if(intervals.length == 0) return 0;
          Arrays.sort(intervals, (a, b)->{
              return a[0] - b[0];
          });
          PriorityQueue<int[]> pq = new PriorityQueue<>((a, b)->{
              return a[1] - b[1];
          });
          int res = 1;
          for(int[] interval : intervals){
              int start = interval[0];
              pq.offer(interval);
              while(pq.peek()[1] <= start) pq.poll();
              res = Math.max(res, pq.size());
          }
          return res;
      }
  }
  ```

### Merge interval
1. Merge all intervals and try to remove all overlaps.
  * [56. Merge Intervals](https://github.com/Seanforfun/Algorithm-and-Leetcode/blob/master/leetcode/56.%20Merge%20Intervals.md)
  * Method 1: Only by sort, not understandable
    * Step 1: Sort all starts and ends.
    * Step 2: Merge each intervals
    ```Java
    class Solution {
        public int[][] merge(int[][] intervals) {
            if(intervals.length == 0) return new int[0][0];
            int len = intervals.length;
            int[] starts = new int[len];
            int[] ends = new int[len];
            for(int i = 0; i < len; i++){
                starts[i] = intervals[i][0];
                ends[i] = intervals[i][1];
            }
            Arrays.sort(starts);
            Arrays.sort(ends);
            List<int[]> res = new ArrayList<>();
            int start = starts[0];
            for(int i = 1; i < intervals.length; i++){
                if(starts[i] > ends[i - 1]){
                    res.add(new int[]{start, ends[i - 1]});
                    start = starts[i];
                }
            }
            res.add(new int[]{start, ends[intervals.length - 1]});
            int[][] result = new int[res.size()][2];
            for(int i = 0; i < res.size(); i++){
                result[i] = res.get(i);
            }
            return result;
        }
    }
    ```

  * Method 2:
    * Step 1: Sort all intervals by start time.
    * Step 2: Take start and end time, the start time is fixed every time and end time is always taking the maximum.
    ```Java
    class Solution {
        public int[][] merge(int[][] intervals) {
            if(intervals.length == 0) return new int[0][0];
            int len = intervals.length;
            Arrays.sort(intervals, (a, b)->{
                return a[0] - b[0];
            });
            int start = intervals[0][0], end = intervals[0][1];
            List<int[]> res = new ArrayList<>();
            for(int i = 1; i < intervals.length; i++){
                if(intervals[i][0] > end){
                    res.add(new int[]{start, end});
                    start = intervals[i][0];
                    end = intervals[i][1];
                }else{
                    end = Math.max(end, intervals[i][1]);
                }
            }
            res.add(new int[]{start, end});
            int[][] result = new int[res.size()][2];
            for(int i = 0; i < res.size(); i++){
                result[i] = res.get(i);
            }
            return result;
        }
    }
    ```

2. Inset intervals
  * [57. Insert Interval](https://github.com/Seanforfun/Algorithm-and-Leetcode/blob/master/leetcode/57.%20Insert%20Interval.md)
  * Ideas is same as merge interval, merge this inserted intervals.
  ```Java
  class Solution {
   	public int[][] insert(int[][] intervals, int[] newInterval) {
   		int len = intervals.length;
   		int[] starts = new int[len + 1];
   		int[] ends = new int[len + 1];
   		// Currently the starts and ends are sorted and non-overlap
   		for(int i = 0; i < len; i++){
   			starts[i] = intervals[i][0];
   			ends[i] = intervals[i][1];
   		}
   		starts[len] = newInterval[0];
   		ends[len] = newInterval[1];
   		Arrays.sort(starts);
   		Arrays.sort(ends);
   		List<int[]> temp = new ArrayList<>();
   		int start = starts[0];
   		for(int i = 1; i <= len; i++){
   			if(starts[i] > ends[i - 1]){
   				temp.add(new int[]{start, ends[i - 1]});
   				start = starts[i];
   			}
   		}
   		temp.add(new int[]{start, ends[len]});
   		int[][] res = new int[temp.size()][2];
   		for(int i = 0; i < temp.size(); i++){
   			res[i] = temp.get(i);
   		}
   		return res;
   	}
   }
  ```

### Pick as much intervals as possible. This question equals deleting least intervals to get a no-overlap array.
1. Delete least intervals to make non-overlap
  * [435. Non-overlapping Intervals](https://github.com/Seanforfun/Algorithm-and-Leetcode/blob/master/leetcode/435.%20Non-overlapping%20Intervals.md)
  * Algorithm: Greedy.
  * Greedy for earliest ending time, since we want to as many valid interval as possible, we want the previous interval ends as early as possible, so we can have a chance to enter the next interval.
  * Step 1: Sort the intervals according to the ending time.
  * Step 2: Check each term and find the number of valid intervals.
  ```Java
  class Solution {
      public int eraseOverlapIntervals(int[][] intervals) {
          int len = intervals.length;
          if(len == 0) return 0;
          Arrays.sort(intervals, (a, b)->{
              return a[1] - b[1];
          });
          int end = intervals[0][1];
          int count = 1;
          for(int i = 1; i < len; i++){
              if(intervals[i][0] >= end){
                  end = intervals[i][1];
                  count++;
              }
          }
          return len - count;
      }
  }
  ```

2. Find least number of intervals covers all range
  * [452. Minimum Number of Arrows to Burst Balloons](https://github.com/Seanforfun/Algorithm-and-Leetcode/blob/master/leetcode/452.%20Minimum%20Number%20of%20Arrows%20to%20Burst%20Balloons.md)
  * Same as the previous question, we want to have all areas covered with least intervals. It's like we get the largest number of intervals.
  ```Java
  class Solution {
      public int findMinArrowShots(int[][] points) {
          int len = points.length;
          if(len == 0) return 0;
          Arrays.sort(points, (a, b)->{
              return a[1] - b[1];
          });
          int end = points[0][1];
          int count = 1;
          for(int i = 1; i < len; i++){
              if(points[i][0] > end){
                  end = points[i][1];
                  count ++;
              }
          }
          return count;
      }
  }
  ```

### Find all overlaps
1. Find all overlaps.
    * [986. Interval List Intersections](https://github.com/Seanforfun/Algorithm-and-Leetcode/blob/master/leetcode/986.%20Interval%20List%20Intersections.md)
    * Method 1: Sort starts and ends time separately.
    * Step 1: Sort starts and ends arrays.
    * Step 2: Find all overlaps.
    ```Java
    class Solution {
        public int[][] intervalIntersection(int[][] A, int[][] B) {
            int len = A.length + B.length;
            int[] starts = new int[len];
            int[] ends = new int[len];
            for(int i = 0; i < A.length; i++){
                starts[i] = A[i][0];
                ends[i] = A[i][1];
            }
            for(int i = A.length; i < A.length + B.length; i++){
                starts[i] = B[i - A.length][0];
                ends[i] = B[i - A.length][1];
            }
            Arrays.sort(starts);
            Arrays.sort(ends);
            List<int[]> res = new ArrayList<>();
            for(int i = 1; i < starts.length; i++){
                if(starts[i] <= ends[i - 1]){
                    res.add(new int[]{starts[i], ends[i - 1]});
                }
            }
            int size = res.size();
            int[][] result = new int[size][2];
            for(int i = 0; i < size; i++){
                result[i] = res.get(i);
            }
            return result;
        }
    }
    ```
    
    * Method 2: Sort intervals together
    1. If start time equals, sort by end time.
    2. Find all overlaps.
    ```Java
    class Solution {
        public int[][] intervalIntersection(int[][] A, int[][] B) {
            int len = A.length + B.length;
            if(len == 0) return new int[0][2];
            int[][] arr = new int[len][2];
            for(int i = 0; i < A.length; i++)   arr[i] = A[i];
            for(int i = A.length; i < len; i++) arr[i] = B[i - A.length];
            Arrays.sort(arr, (a, b)->{
                return a[0] == b[0] ? a[1] - b[1]: a[0] - b[0];
            });
            int end = arr[0][1];
            List<int[]> res = new ArrayList<>();
            for(int i = 1; i < len; i++){
                if(arr[i][0] <= end){
                    res.add(new int[]{arr[i][0], Math.min(arr[i][1], end)});
                }
                if(end < arr[i][1])
                    end = arr[i][1];
            }
            int size = res.size();
            int[][] result = new int[size][2];
            for(int i = 0; i < size; i++) result[i] = res.get(i);
            return result;
        }
    }
    ```

2. Find all non-overlaps
    * [759. Employee Free Time](https://github.com/Seanforfun/Algorithm-and-Leetcode/blob/master/leetcode/759.%20Employee%20Free%20Time.md)
    * Step 1: We sort together with the start time, it start time is same, sort by end time.
    * Step 2: Idea is same as the previous question.
    ```Java
    class Solution {
        private static final Comparator<int[]> cmp = new Comparator<int[]>(){
            @Override
            public int compare(int[] a, int[] b){
                return a[0] == b[0] ? (a[1] - b[1]): (a[0] - b[0]);
            }
        };
        public int[][] employeeFreeTime(int[][][] schedule) {
            List<int[]> list = new ArrayList<>();
            for(int[][] person : schedule){
                for(int[] interval : person)
                    list.add(interval);
            }
            if(list.size() == 0) return new int[0][2];
            Collections.sort(list, cmp);
            List<int[]> res = new ArrayList<>();
            int end = list.get(0)[1];
            for(int i = 1; i < list.size(); i++){
                int[] cur = list.get(i);
                if(cur[0] > end){
                    res.add(new int[]{end, cur[0]});
                }
                end = Math.max(end, cur[1]);
            }
            return res.toArray(new int[0][2]);
        }
    }
    ```