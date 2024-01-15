---
layout: post
title: implement-str
date: 2024-01-15 01:28 +0900
category: [ algorithm, string ]
tags: [ Algorithm, Leetcode, String ]
img_path: /assets/images/
---

### [문제](https://leetcode.com/explore/learn/card/array-and-string/203/introduction-to-string/1161/)

![문제]({{site.url}}/assets/images/implementstrStr.png)

1. 이진 값이 들어 있는 a, b를 서로 더합니다. Long 범위를 넘는 케이스가 있어 BigInteger를 사용했습니다. 더한 숫자 값을 다시 각 원소별 리스트로 변환합니다.
2. 리스트에서 오른쪽부터 왼쪽으로 가면서 값이 2 이상인 경우 2로 나눈 나머지로 대체하고, 그 전 인덱스 값에 1을 더해 캐리를 흉내냅니다.
3. 첫번째에서 캐리가 발생하면 지금까지의 결과물에 1을 붙이고 리턴합니다.

```kotlin
fun strStr(
  haystack: String,
  needle: String,
): Int {
  var result = Int.MAX_VALUE
  val firstNeedleWord = needle[0]
  val needleLength = needle.length

  haystack.forEachIndexed { index, c ->
    if (c == firstNeedleWord && index + needleLength <= haystack.length) {
      if (haystack.slice(index..<index + needleLength) == needle) {
        result = Math.min(result, index)
      }
    }
  }

  if (result == Int.MAX_VALUE) {
    return -1
  }
  return result
}
