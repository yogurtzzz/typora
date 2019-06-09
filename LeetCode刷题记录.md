## LeetCode刷题记录

### 1.无重复字符的最长子串

1. 暴力法

   ```java
   public int lengthOfLongestSubstring(String s) {
           int size = s.length();
           int maxLen = 0;
           List<Character> list = new ArrayList<>();
           int start = 0;
           for (int i = 0; i < size ;){
               if (list.size() == 0 || !list.contains(s.charAt(i))){
                   list.add(s.charAt(i));
                   if (i == size - 1 ){
                       maxLen = list.size() > maxLen ? list.size() : maxLen;
                   }
                   ++i;
               }else{
                   maxLen = list.size() > maxLen ? list.size() : maxLen;
                   list.clear();
                   i = ++start;
               }
           }
           return maxLen;
       }
   ```

   

2. 滑动窗口法    O(2n)

   ```java
   public int lengthOfLongestSubstring(String s) {
           int n = s.length();
           Set<Character> set = new HashSet<>();
           int ans = 0, i = 0, j = 0;
           while (i < n && j < n) {
               // try to extend the range [i, j]
               if (!set.contains(s.charAt(j))){
                   set.add(s.charAt(j++));
                   ans = Math.max(ans, j - i);
               }
               else {
                   set.remove(s.charAt(i++));
               }
           }
           return ans;
       }
   //下面是优化后的
   public int lengthOfLongestSubstring(String s) {
           int[] times = new int [256];
           char[] charseq = s.toCharArray();
           int ans = 0,j = 0;
           for (int i = 0; i < charseq.length;){
               if (times[charseq[i]] == 0){
                   //一次还未出现
                   times[charseq[i++]]++;
               }else{
                   //已经出现过，那么从左边开始减
                   times[charseq[j++]]--;
               }
               ans = Math.max(i - j,ans);
           }
           return ans;
       }
   ```

   

3. 优化的滑动窗口法   O(n)

   ```java
   public int lengthOfLongestSubstring(String s) {
           Map<Character,Integer> map = new HashMap<>();
           int ans = 0,i = 0,j = 0;
           for (; j < s.length();j++){
               if (!map.containsKey(s.charAt(j))){
                   map.put(s.charAt(j),j);
               }else{
                   i = Math.max(map.get(s.charAt(j)) + 1,i);
                   map.put(s.charAt(j),j);
               }
               ans = Math.max(j - i + 1,ans);
           }
           return ans;
       }
   //下面是优化后的
   public int lengthOfLongestSubstring(String s) {
           char[] charseq = s.toCharArray();
           int[] index = new int[257];
           int ans = 0,j = 0;
           for (int i = 0; i < charseq.length;){
               if (index[charseq[i]] == 0){
                   //一次还未出现
                   //记录出现的位置
                   index[charseq[i]] = ++i;
               }else{
                   //已经出现过
                   j = Math.max(index[charseq[i]],j);
                   index[charseq[i]] = ++i;
               }
               ans = Math.max(i - j,ans);
           }
           return ans;
       }
   ```

   