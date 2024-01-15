---
layout: post
title: add binary
date: 2024-01-13 16:31 +0900
img_path: /assets/images/
category: [ algorithm, string ]
tags: [ Algorithm, Leetcode, String ]
---

### [Add Binary](https://leetcode.com/explore/learn/card/array-and-string/203/introduction-to-string/1160)

![문제]({{site.url}}/assets/images/addbinary.png)

1. 이진 값이 들어 있는 a, b를 서로 더합니다. Long 범위를 넘는 케이스가 있어 BigInteger를 사용했습니다. 더한 숫자 값을 다시 각 원소별 리스트로 변환합니다.
2. 리스트에서 오른쪽부터 왼쪽으로 가면서 값이 2 이상인 경우 2로 나눈 나머지로 대체하고, 그 전 인덱스 값에 1을 더해 캐리를 흉내냅니다.
3. 첫번째에서 캐리가 발생하면 지금까지의 결과물에 1을 붙이고 리턴합니다.

```kotlin
fun addBinary(
  a: String,
  b: String,
): String {
  val sum = (a.toBigInteger() + b.toBigInteger()).toString().map { it.digitToInt() }.toMutableList()
  for (i in sum.size - 1 downTo 0) {
    if (sum[i] >= 2) {
      sum[i] = sum[i] % 2
      if (i == 0) {
        return "1${sum.joinToString("")}"
      }
      sum[i - 1] = sum[i - 1] + 1
    }
  }

  return sum.joinToString("")
}
```


