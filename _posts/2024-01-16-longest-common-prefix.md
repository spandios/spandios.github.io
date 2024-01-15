---
layout: post
title: longest-common-prefix
date: 2024-01-16 00:03 +0900
---

### [LongestCommonPrefix](https://leetcode.com/explore/learn/card/array-and-string/203/introduction-to-string/1162/)

![문제]({{site.url}}/assets/images/longestcommonprefix.png)

<br> 

1. 사전순으로 문자열이 정렬하면 중간은 반드시 같은 문자열이 있다는 것을 알 수 있기 때문에 맨 첫번째와 마지막 인덱스의 값만 서로 공통된 값이 있는지 체크하면 됩니다.

<br>

```kotlin
fun longestCommonPrefix(strs: Array<String>): String {
  if (strs.size == 1) {
    return strs[0]
  }
  var commonPreFixIndex = 0
  strs.sort()

  while (true) {
    val firstStrChar = strs[0].getOrNull(commonPreFixIndex) ?: break
    val lastStrChar = strs[strs.size - 1].getOrNull(commonPreFixIndex) ?: break

    if (firstStrChar == lastStrChar) {
      commonPreFixIndex++
    } else {
      break
    }
  }

  if (commonPreFixIndex == 0) {
    return ""
  }

  return strs[0].substring(0, commonPreFixIndex)
}


```
