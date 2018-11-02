# GraphQL
https://graphql.github.io/learn/queries/

GrqphQL 은 query language.


쿼리를 json 형태로 던지면
```
{
  me {
    name
  }
}
```

아래와 같은 형태의 응답을 주나보다
```
{
  "me": {
    "name": "Luke Skywalker"
  }
}
```


.

.


약간의 구체화 된 예로 이런식으로 쿼리 던지면
```
{
  hero {
    name
    # Queries can have comments!
    friends {
      name
    }
  }
}
```

아래처럼 "data" 객체에 결과를 박아준다. 흔히 보던 json api result 느낌이지.
```
{
  "data": {
    "human": {
      "name": "Luke Skywalker",
      "height": 1.72
    }
  }
}
```

.

.

where 절 같은게 필요하면 아래와 같이 쿼리를 날린다. API 는 보통 call url 에 parameter 주렁주렁이지만 얘는 이렇게 함.
parameter 는 하나 겠지만 json 에 주렁주렁 하면 된다.
```
{
  human(id: "1000") {
    name
    height(unit: FOOT)
  }
}
```

그럼 요렇게 나옴
```
{
  "data": {
    "human": {
      "name": "Luke Skywalker",
      "height": 5.6430448
    }
  }
}
```


.

.


쿼리 요청시 같은 이름이 나올 수도 있는 경우, 아래 처럼 alias 를 이용해 요청 할 수 있음. () 안이 이해가 잘 안됨.
```
{
  empireHero: hero(episode: EMPIRE) {
    name
  }
  jediHero: hero(episode: JEDI) {
    name
  }
}
```

```
{
  "data": {
    "empireHero": {
      "name": "Luke Skywalker"
    },
    "jediHero": {
      "name": "R2-D2"
    }
  }
}
```


.

.

Fragments 라고 소개되어있는데 부분 비교 느낌이다.
```
{
  leftComparison: hero(episode: EMPIRE) {
    ...comparisonFields
  }
  rightComparison: hero(episode: JEDI) {
    ...comparisonFields
  }
}

fragment comparisonFields on Character {
  name
  appearsIn
  friends {
    name
  }
}
```

```
{
  "data": {
    "leftComparison": {
      "name": "Luke Skywalker",
      "appearsIn": [
        "NEWHOPE",
        "EMPIRE",
        "JEDI"
      ],
      "friends": [
        {
          "name": "Han Solo"
        },
        {
          "name": "Leia Organa"
        },
        {
          "name": "C-3PO"
        },
        {
          "name": "R2-D2"
        }
      ]
    },
    "rightComparison": {
      "name": "R2-D2",
      "appearsIn": [
        "NEWHOPE",
        "EMPIRE",
        "JEDI"
      ],
      "friends": [
        {
          "name": "Luke Skywalker"
        },
        {
          "name": "Han Solo"
        },
        {
          "name": "Leia Organa"
        }
      ]
    }
  }
}
```


















