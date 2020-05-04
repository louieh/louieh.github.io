---
title: "Beautiful Python refactor"
layout: post
date: 2020-04-19
headerImage: false
tag:
- note
- Python
star: true
category: blog
---



A kind of anti-pattern we should avoid is called ITM which means Initialize and Then Modify.

ex1:

```python
i = 0
for t in tr_elements[0]:
    i += 1
		# do something...
```

we initialize i and update it in the for loop, actually we can use enumerate

```python
for i, t in enumerate(tr_elements[0]):
		# do something...
```

ex2:

```python
col = []
for t in tr_elements[0]:
		name = t.text_content()
		col.append((name, []))
```

we initialize a list col and modify it in the loop, actually we can use list comprehension.

```python
col = [(t.text_content(), []) for t in tr_elements[0]]
```

the code will become more readable and more maintainable and just better in general.