# BUBBLE SORT 버블정렬

**Bubble sort** :

- 두 인접한 원소를 검사하여 정렬하는 방법


 - 시간 복잡도 함수 O(n^2)
 - 코드가 단순해서 자주 사용

```python
ori_list = [5,3,8,1,7,2]

def bubble_sort(list):
    list_len = len(list)
    for i in range(list_len - 1):
        for j in range(list_len -i -1):
            if list[j] > list[j+1]:
                list[j], list[j+1] = list[j+1], list[j]
            print(list)       

bs= bubble_sort(ori_list)
print(bs)
```

