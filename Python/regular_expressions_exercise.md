### 12.정규표현식 실습 문제풀기

1번문제:

```
m = re.findall('a\w\w\w', story)
m

------------------------------------------

강사님의 풀이:
result = re.findall(r'\ba\b{3}\b', story)
```

2번문제:
```
m = re.findall('\w+r', story)
m

------------------------------------------

강사님의 풀이:
result = re.findall(r'\w+r\b', story)
```

3번문제:
```
m = re.findall('\w*[abcde][abcde][abcde]\w*', story)
m

------------------------------------------

강사님의 풀이:
result = re.findall(r'\w*[abcde]{3}\w*', story)
```

4번문제:

```
강사님의 풀이:

def re_change(m):
	return '{}{}[{}]'.format(m.group('before').upper(), m.group('center'), m.group('after'))

p = re.compile(r(?P<before>\w+)(?P<center>\s*\s*)(?P<after>\w+)')
result = re.sub(p, re_change, story)

(lambda m: '{}{}[{}]'.format(m.group('before').upper(), m.group('center'), m.group('after'))
```