# 알고리즘 에러 핸들링 정리

## 타입 불일치 에러

### `incompatible types: cannot infer type arguments for HashSet<>`

**원인**
변수 타입을 잘못 지정하여 발생하는 문제. `graph`가 `int[][]` 타입인데 `HashSet<>`을 할당하려 할 때 발생.

**에러 메시지**
```
error: incompatible types: cannot infer type arguments for HashSet<>
graph[i] = new HashSet<>();

reason: no instance(s) of type variable(s) E exist so that HashSet<E> conforms to int[]
where E is a type-variable:
```

**해설**
- `HashSet<>` 안에 어떤 타입이 들어갈지 추론할 수 없음
- `int[][]` 배열에 `HashSet`을 넣으려 했기 때문에 타입 충돌 발생

**해결 방법**
- `graph`의 타입을 목적에 맞게 수정: `int[][]` → `HashSet<Integer>[]`
- 또는 `HashSet<>` 대신 `int[]`에 맞는 자료구조 사용
