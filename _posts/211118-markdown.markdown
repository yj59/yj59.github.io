---
title: "Markup: HTML Tags and Formatting"
header:
  teaser: "assets/images/markup-syntax-highlighting-teaser.jpg"
categories:
  - Markup
tags:
  - content
  - css
  - formatting
  - html
  - markup
toc: true
toc_sticky: true
---

# 한영 출력 확인



## 1. 행간/자간



가나다라마바사아자차카타파하

가나다라마

바사아자

차카타

파하



abcdefghijklmnopqrstuvwxyz

abcdefghijk

lmnopqr

stuvw

xyz



## 2. 글머리



# H1

## H2

### H3

#### H4

##### H5

###### H6



## 3. 인용구

>가나다라마바사아자차카타파하
>
>abcdefghijklmnopqrstuvwxyz



## 4. 글머리부호

+ 가
  + 나
    + 다
      + 라
        + 마
          + 바
            + 사



## 5. 코드

```C
#include <stdio.h>

int main() {

	printf("hello, world!");
	
	return 0;
}
```



## 6. 참조

구글 : <https://google.com/>



## 7. 문장

기울기 : *가나다라마바사*

굵기 : **가나다라마바사**

찍 : ~~가나다라마바사~~



## 8. 선

***

---



## 9. 수식

수식 적용 시 머리말 use_math: true 추가

1. **여러 줄 입력**

$$
y = ax+b
$$

y = ax+b


$$
\sigma^{2}_{Batch}\leftarrow \frac {1}{m}\sum_{i=1}^m (x_{i}-\mu_{Batch})^{2}\qquad//분산
$$

![v](C:/git-log/yj59.github.io/_posts/img/2021-11-18-markdown-test/v.png)


2. **한 줄 입력**


$y = ax+b$

$\sigma^{2}_{Batch}\leftarrow \frac {1}{m}\sum_{i=1}^m (x_{i}-\mu_{Batch})^{2}\qquad$
