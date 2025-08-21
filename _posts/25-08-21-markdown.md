---
layout: post
title: "마크다운 형식이랑 Html 정리"
categories:
  - +
tags:
  - markdown
  - html
last_modified_at: 2025-08-21
---

깃블로그에 좀 익숙해질 겸 마크다운과 html 치트시트 하나 만들려고 한다. 본문은 그냥 작성하면 된다. 

# Header one
제목 1은 # 제목

## Header two
제목 2은 ## 제목

### Header three
제목 3은 ### 제목

#### Header four

##### Header five

###### Header six
6까지 존재하지만, 그냥 3까지만 활용하는 게 좋을 거 같다. 

## Blockquotes

Single line blockquote:

> 한 줄 인용은 > 하나 치고 작성 

Multi line blockquote with a cite reference:

> 여러 줄도 그냥 > 하나 치고 길게 길게 길게 적으면 된다 

<cite>Steve Jobs</cite> --- Apple Worldwide Developers' Conference, 1997
{: .small}
마지막 누가 어디서 말했는지는 < cite> 사용해서 사람 감싸기 

## Tables
표는 내가 보기에 필요할 때 지피티 돌려서 만들어달라 하는 게 편하긴 하다. 

| Employee         | Salary |                                                              |
| --------         | ------ | ------------------------------------------------------------ |
| [John Doe](#)    | $1     | Because that's all Steve Jobs needed for a salary.           |
| [Jane Doe](#)    | $100K  | For all the blogging she does.                               |
| [Fred Bloggs](#) | $100M  | Pictures are worth a thousand words, right? So Jane × 1,000. |
| [Jane Bloggs](#) | $100B  | With hair like that?! Enough said.                           |

위 표에 1행은 (#) 넣어서 링크를 만들었네. 대괄호로 문자열 감싸고 그 옆에 소괄호로 링크 넣으면 첨부된다. 
아래는 행마다 정렬이 다르니까 참고 
+) 표 위아래 모두 한 줄씩 띄워야 한다. 

| Header1 | Header2 | Header3 |
|:--------|:-------:|--------:|
| cell1   | cell2   | cell3   |
| cell4   | cell5   | cell6   |
|-----------------------------|
| cell1   | cell2   | cell3   |
| cell4   | cell5   | cell6   |
|=============================|
| Foot1   | Foot2   | Foot3   |

## Definition Lists

Definition List Title
: Definition list division.

본문 아랫줄에 : 넣고 작성하면 들여쓰기가 된다. 

## Unordered Lists (Nested)

  * List item one
  * List item two
  * List item three
  * List item four
리스트는 그냥 참고하자. 

## Ordered List (Nested)

  1. List item one
      a. List item one
          1. List item one
          2. List item two
          3. List item three
          4. List item four
      b. List item two
      c. List item three
      d. List item four
  2. List item two
  3. List item three
  4. List item four

이것도 그냥 참고. a,b,c,d도 지원하나 해봤는데 안된다. 

## Address element

<address>
  1 Infinite Loop<br /> Cupertino, CA 95014<br /> United States
</address>

< address> 넣고 주소 작성하면 기울기를 제공해주는데 특별한 건 모르겠다. 

## Anchor element (aka. Link)

This is an example of a [link](http://apple.com "Apple").

오 소괄호 링크 옆에 큰따옴표로 문자열 작성하면 마우스 오버 텍스트 지원한다. 

## 문자 스타일?

 `word-wrap: break-word;` <strike>strikeout text</strike> _italicize_ <ins>inserted</ins> <kbd>keyboard text</kbd> **bold text** H<sub>2</sub>O E = MC<sup>2</sup>

  문자 스타일? 효과를 넣을 수 있다. 


## 코드 작성 

<pre>
.post-title {
    margin: 0 0 5px;
    font-weight: bold;
    font-size: 38px;
    line-height: 1.2;
    and here's a line of some really, really, really, really long text, just to see how the PRE element handles it and to find out how it overflows;
}
</pre>

< pre>로도 할 수 있는데 맠다 형식으로 작성하는 게 편한 거 같다. 아래 유튜브 추가하는 코드 첨부하면서 맠다 형식 넣었다. 

## Images

이미지 임베드 중요하지.. 근데 생각해보니까 이거 어느 폴더에 넣지? _screenshots에 넣어야 하나

![placeholder](https://placehold.it/800x400 "Large example image")
![placeholder](https://placehold.it/400x200 "Medium example image")
![placeholder](https://placehold.it/200x200 "Small example image")

## 유튜브 첨부 
```html
<!-- responsive iframe. The framesize reduces proportionately when viewing in mobile -->
<div class="video-container">
  <iframe class="embed-responsive-item" src="..."></iframe>
</div>
```

## 이외에 Html로 임베드할 수 있는 요소

<blockquote class="twitter-tweet" data-lang="en"><p lang="en" dir="ltr">Oh I dunno. It&#39;s probably been over 15 years since I smudged out a face with a pencil and kneaded eraser. <a href="https://twitter.com/hashtag/WIP?src=hash">#WIP</a> <a href="https://twitter.com/hashtag/Sktchy?src=hash">#Sktchy</a> <a href="https://t.co/PwqbMddyVK">pic.twitter.com/PwqbMddyVK</a></p>&mdash; Michael Rose (@mmistakes) <a href="https://twitter.com/mmistakes/status/826644109670612997">February 1, 2017</a></blockquote>
<script async src="//platform.twitter.com/widgets.js" charset="utf-8"></script>

<div class="message">
  Howdy! This is an example blog post that shows several types of HTML content supported in this theme.
</div>


## 맠다 형식과 Html 스타일 비교 

- **To bold text**, use `<strong>`.
- *To italicize text*, use `<em>`.
- Abbreviations, like <abbr title="HyperText Markup Langage">HTML</abbr> should use `<abbr>`, with an optional `title` attribute for the full phrase.
- Citations, like <cite>&mdash; Mark otto</cite>, should use `<cite>`.
- <del>Deleted</del> text should use `<del>` and <ins>inserted</ins> text should use `<ins>`.
- Superscript <sup>text</sup> uses `<sup>` and subscript <sub>text</sub> uses `<sub>`.


## 여러 코드 임베드 비교 

{% highlight js %}
// Example can be run directly in your JavaScript console

// Create a function that takes two arguments and returns the sum of those arguments
var adder = new Function("a", "b", "return a + b");

// Call the function
adder(2, 6);
// > 8
{% endhighlight %}

```css
#container {
  float: left;
  margin: 0 -240px 0 0;
  width: 100%;
}
```
{% highlight html linenos %}
{% raw %}<nav class="pagination" role="navigation">
  {% if page.previous %}
    <a href="{{ site.url }}{{ page.previous.url }}" class="btn" title="{{ page.previous.title }}">Previous article</a>
  {% endif %}
  {% if page.next %}
    <a href="{{ site.url }}{{ page.next.url }}" class="btn" title="{{ page.next.title }}">Next article</a>
  {% endif %}
</nav><!-- /.pagination -->{% endraw %}
{% endhighlight %}

여담인데 위 코드는 왜이렇게 어려워. 

GitHub Gist embeds can also be used:

```html
<script src="https://gist.github.com/mmistakes/77c68fbb07731a456805a7b473f47841.js"></script>
```

Which outputs as:

<script src="https://gist.github.com/mmistakes/77c68fbb07731a456805a7b473f47841.js"></script>

## 각주

Highlighting does not affect the meaning of the text itself; it is intended only for human readers.[^1]

[^1]: <http://en.wikipedia.org/wiki/Syntax_highlighting>


-----