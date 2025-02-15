# 字符串

常用思路： 双指针：左右指针、快慢指针

## LeetCode 3 无重复字符的最长子串

左右指针，形成一个滑动窗口，碰到已出现的字符，就滑动左指针

```java
    public int lengthOfLongestSubstring(String s) {
        int length = s.length();
        int ans = 0;
        Map<Character, Integer> map = new HashMap<>();
        int left = 0;
        for(int i= 0; i < length; i++){
            // 如果已经包含当前字符,找到当前字符上次出现的位置
            if (map.containsKey(s.charAt(i))) {
                left = Math.max(map.get(s.charAt(i)) + 1, left);
            }
            // 更新当前字符最后出现的位置
            map.put(s.charAt(i), i);
            ans = Math.max(ans, i - left + 1);
        }
        return ans;
    }
```

## LeetCode 5 最长回文子串

暴力解法： 遍历所有子串，依次判断是否回文 dp解法：在上述暴力解法基础之上，将子串状态记录下来，防止重复判断。判断子串i,j依赖于i,j位置的字符是否匹配以及i+1,j-1子串的状态。base case是字符串长度为1的子串，均为回文。遍历则从长度为2的子串开始 中心扩展法： 从中间的字母向外扩展，与dp解法思路类似，只是从长度为1的子串开始，不需要再记录状态

```java
    // DP解法
    public String longestPalindrome3(String s) {
        if (s.length() <= 1) {
            return s;
        }

        boolean[][] dp = new boolean[s.length()][s.length()];

        // base case
        for (int i = 0; i < s.length(); i++) {
            dp[i][i] = true;
        }
        // 最终需要返回子串，所以用left和max分别表示子串起始位置和长度
        int left = 0, max = 1;
        for (int curLength = 2; curLength <= s.length(); curLength++) {
            for (int i = 0; i < s.length(); i++) {
                int j = i + curLength - 1;

                if (j >= s.length()) {
                    break;
                }

                if (s.charAt(i) == s.charAt(j)) {
                    if (curLength <= 3) {
                        dp[i][j] = true;
                    } else {
                        dp[i][j] = dp[i + 1][j - 1];
                    }
                    if (dp[i][j] && curLength > max) {
                        max = curLength;
                        left = i;
                    }
                } else {
                    dp[i][j] = false;
                }
            }
        }
        return s.substring(left, left + max);
    }
    /**
     * 中心扩展法，从中间的字母向外一一扩展，找最长回文子串。
     * 假设字符串是abba，那么i=1时，是从b和b开始对比的，如果字符串是aba，从b和自身开始对比的。
     */
    public String longestPalindrome(String s) {
        int len = s.length();
        if (len < 2)
            return s;

        for (int i = 0; i < len - 1; i++) {
            extendPalindrome(s, i, i); // 确保奇数长度的子串, try to extend
            // Palindrome as possible
            extendPalindrome(s, i, i + 1); //偶数长度的子串
        }
        return s.substring(lo, lo + maxLen);
    }

    private void extendPalindrome(String s, int left, int right) {
        while (left >= 0 && right < s.length() && s.charAt(left) == s.charAt(right)) {
            left--;
            right++;
        }
        // 因为上一个循环多扩展了两个位置，所以这里边界是right - left - 1
        if (maxLen < right - left - 1) {
            lo = left + 1;
            maxLen = right - left - 1;
        }
    }
```

### 最小覆盖子串

```java
    /**
     * 76. 最小覆盖子串
     * 给你一个字符串 s 、一个字符串 t 。返回 s 中涵盖 t 所有字符的最小子串。如果 s 中不存在涵盖 t 所有字符的子串，则返回空字符串 "" 。
     */
    public String minWindow(String s, String t) {
        if (s == null || s.length() == 0 || t == null || t.length() == 0) {
            return "";
        }
        Map<Character, Integer> need = new HashMap<>();
        //记录需要的字符的个数
        for (int i = 0; i < t.length(); i++) {
            need.put(t.charAt(i), need.getOrDefault(t.charAt(i), 0) + 1);
        }
        //l是当前左边界，r是当前右边界，size记录窗口大小，count是需求的字符个数，start是最小覆盖串开始的index
        int left = 0, right = 0, size = Integer.MAX_VALUE, count = t.length(), start = 0;
        //遍历所有字符
        while (right < s.length()) {
            char c = s.charAt(right);
            if (need.getOrDefault(c, 0) > 0) {//需要字符c
                count--;
            }
            need.put(c, need.getOrDefault(c, 0) - 1);//把右边的字符加入窗口
            if (count == 0) {//窗口中已经包含所有字符
                while (left < right && need.getOrDefault(s.charAt(left), 0) < 0) {
                    need.put(s.charAt(left), need.getOrDefault(s.charAt(left), 0) + 1);//释放右边移动出窗口的字符
                    left++;//指针右移
                }
                if (right - left + 1 < size) {//不能右移时候挑战最小窗口大小，更新最小窗口开始的start
                    size = right - left + 1;
                    start = left;//记录下最小值时候的开始位置，最后返回覆盖串时候会用到
                }
                //l向右移动后窗口肯定不能满足了 重新开始循环
                need.put(s.charAt(left), need.getOrDefault(s.charAt(left), 0) + 1);
                left++;
                count++;
            }
            right++;
        }
        return size == Integer.MAX_VALUE ? "" : s.substring(start, start + size);
    }
```

### 最长公共子序列

1143题：最长公共子序列，LCS\(Longest common subsequence\)问题

dp\[i\]\[j\]表示第一个字符串s1前i个字符与第二个字符串s2前j个字符公共子序列的长度

状态转移方程： 当s1\[i\] == s2\[j\]时，dp\[i\]\[j\] = dp\[i-1\]\[j-1\] + 1； 而s1\[i\] != s2\[j\]时，则不将i或j这个字符考虑进去，取dp\[i\]\[j-1\]和dp\[i-1\]\[j\]中较大者。

```java
    public int longestCommonSubsequence(String text1, String text2) {
        int m = text1.length();
        int n = text2.length();
        int[][] dp = new int[m + 1][n + 1];

        for (int i = 1; i <= m; i++) {
            char c1 = text1.charAt(i-1);
            for (int j = 1; j <= n; j++) {
                char c2 = text2.charAt(j-1);
                if (c1 == c2) {
                    dp[i][j] = dp[i-1][j-1] + 1;
                }else{
                    dp[i][j] = Math.max(dp[i][j-1], dp[i-1][j]);
                }
            }
        }
        return dp[m][n];
    }
```

变形题： 583.两个字符串的删除操作[https://leetcode-cn.com/problems/delete-operation-for-two-strings/](https://leetcode-cn.com/problems/delete-operation-for-two-strings/)

其思路即先求出两个字符串的最长公共子序列，然后计算两个字符串删除得到最长公共子序列需要多少步，即s1.length\(\) + s2.length\(\) - longestCommonSubsquence\(s1, s2\)。

