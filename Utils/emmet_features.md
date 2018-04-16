<!-- $theme: default -->

Atom 에디터에서 emmet 약어
(emmet syntax)
===

---

# Child: >
### ex) `div>ul>li` 입력후 tab키를 누르면
```
<div>
  <ul>
    <li></li>
  </ul>
</div>
```

### 상속되는 요소들을 한번에 쓸 수 있음

---

# Sibling: +

### ex) `div+p+bq` 입력하면

```
<div></div>
<p></p>
<blockquote></blockquote>
```

### 연속된 여러 요소를 쉽게 쓸수있음  

---

# Climb-up: ^ 
### ex) `div+div>p>span+em+bp` 입력하면
```
<div></div>
<div>
  <p>
    <span></span>
    <em></em>
    <blockquote></blockquote>
  </p>
</div>
```
### 이렇게 나온다 
---

### 여기서 마지막 +를 ^로 바꿔서
### `div+div>p>span+em^bq`로 바꿔 입력하면
```
<div></div>
<div>
  <p>
    <span></span>
    <em></em>
  </p>
  <blockquote></blockquote>
</div>
```
### bq가 위에와 달리 p의 밖으로 빠져나오게 됨
### `^` 는 상속의 윗단계로 올라가게 한다
---

# Multiplication: *
### ex) `ul>li*5` 입력하면
```
<ul>
  <li></li>
  <li></li>
  <li></li>
  <li></li>
  <li></li>
</ul>
```	

---

# Grouping: ()
### ex) `div>(header>ul>li*2>a)+footer>p` 입력하면
```
<div>
  <header>
    <ul>
      <li><a href=""></a></li>
      <li><a href=""></a></li>
    </ul>
  </header>
  <footer>
    <p></p>
  </footer>
</div>
```

---

### ex) `(div>dl>(dt+dd)*3)+footer>p` 입력하면
```
<div>
  <dl>
    <dt></dt>
    <dd></dd>
    <dt></dt>
    <dd></dd>
    <dt></dt>
    <dd></dd>
  </dl>
</div>
<footer>
  <p></p>
</footer>
```

---

# ID and CLASS
### ex) `div#header+div.page+div#footer.class1.class2.class3`입력하면
```
<div id="header"></div>
<div class="page"></div>
<div id="footer" class="class1 class2 class3"></div>
```
### `#`는 id값이 되고 / `.`는 class값이 된다 

---

# Custom attributes
### `td[title="Hello world!" colspan=3]` 입력하면
```
<td title="Hello world!" colspan="3"></td>
```
`[]`를 통해 값을 넣어줄수있다

---

# Item numbering: $
### `ul>li.item$*5` 입력하면
```
<ul>
  <li class="item1"></li>
  <li class="item2"></li>
  <li class="item3"></li>
  <li class="item4"></li>
  <li class="item5"></li>
</ul>
```
### `$`부분이 차례대로 숫자가 매겨진다

---
`ul>li.item$$$*5`
```
<ul>
  <li class="item001"></li>
  <li class="item002"></li>
  <li class="item003"></li>
  <li class="item004"></li>
  <li class="item005"></li>
</ul>
```

---

# Changing numbering base and direction
### `ul>li.item$@-*5` 입력하면
```
<ul>
  <li class="item5"></li>
  <li class="item4"></li>
  <li class="item3"></li>
  <li class="item2"></li>
  <li class="item1"></li>
</ul>
```
### `@-` 입력해서 숫자가 역순

---

### `ul>li.item$@3*5` 입력하면
```
<ul>
  <li class="item3"></li>
  <li class="item4"></li>
  <li class="item5"></li>
  <li class="item6"></li>
  <li class="item7"></li>
</ul>
```
### `@3` 입력해서 3부터 숫자를 매긴다

---

### `ul>li.item$@-3*5` 입력하면
```
<ul>
  <li class="item7"></li>
  <li class="item6"></li>
  <li class="item5"></li>
  <li class="item4"></li>
  <li class="item3"></li>
</ul>	
```
### `@-3`입력해서 역순으로 3부터 숫자를 매긴다

---

# Text: {}
### `a{Click me}` 입력하면
```
<a href="">Click me</a>
```
### `{}` 텍스트 부분이 입력됨

---

'a{click}+b{here}'
``` 
<a href="">click</a><b>here</b>
```

`a>{click}+b{here}`
```
<a href="">click<b>here</b></a>
```


`p>{Click }+a{here}+{ to continue}`
```
<p>Click <a href="">here</a> to continue</p>
```

`p{Click }+a{here}+{ to continue}`
```
<p>Click </p>
<a href="">here</a> to continue
```



















