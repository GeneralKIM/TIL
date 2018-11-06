# GraphQL
https://graphql.github.io/learn/queries/

GrqphQL 은 query language.


## Fields
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

## Arguments
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

## alias
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

## Fragments
각 부분별 응답이 같은 구조로 반복될 경우 해당 구조를 재사용할 용도로 사용하는것 같다.
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


.

.

## Operation name
이제 까지 쿼리 예제에서 query 키워드와 Operation name 이 빠진채 호출되었음.
이 2개 키워드는 암묵키워드로 생략이 가능했나봄. 그러나 실제로 사용할때는 이 값을 명시하는것이 쓸때도 명확하겠죠
```
query HeroNameAndFriends {
  hero {
    name
    friends {
      name
    }
  }
}
```

```
{
  "data": {
    "hero": {
      "name": "R2-D2",
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



## Variables
보통 쿼리 요청을 할때 어지간하면 바인딩 변수와 같이 몇몇개의 값을 받아 실행할텐데 그런 부분을 대응하는 기능.
```
query HeroNameAndFriends($episode: Episode = JEDI) {
  hero(episode: $episode) {
    name
    friends {
      name
    }
  }
}
```
위 쿼리의 변수는 $episode: Episode 로 명시됨. 이 쿼리를 쓸때 episode 부분을 바인딩해야겠지.
프로그래밍 언어처럼 '= JEDI' 를 명시하여 초기화 default 값을 정할 수 있음
```
{
  "episode": "JEDI"
}
```
요래 변수 세팅해주고 콜 때리면
```
{
  "data": {
    "hero": {
      "name": "R2-D2",
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


## Directives
프로그래밍 처럼 쿼리에 조건문을 넣을 수 있다.
```
query Hero($episode: Episode, $withFriends: Boolean!) {
  hero(episode: $episode) {
    name
    friends @include(if: $withFriends) {
      name
    }
  }
}


query Hero($episode: Episode, $withFriends: Boolean!) {
  hero(episode: $episode) {
    name
    friends @skip(if: $withFriends) {
      name
    }
  }
}
```
위 : @include(if: $withFriends) $withFriends 값이 true 면 friends.name 를 가져오도록 하나보다.
아래 : @skip(if: $withFriends) $withFriends 값이 true 면 friends.name 를 스킵하도록 하나보다.

변수를 줘야 하므로 변수를 세팅하구
```
{
  "episode": "JEDI",
  "withFriends": false
}
```
콜하면
```
{
  "data": {
    "hero": {
      "name": "R2-D2"
    }
  }
}
```


## Mutations
대부분의 graphQL 의 논의점은 데이터 패칭에 맞춰져 있지만,
모든 완성된 데이터 플랫폼에서는 서버사이드 데이터도 잘 수정하는 방법이 필요로한다.
REST 환경에서 어떤 요청이 어떤 사이드 이팩트로

```
mutation CreateReviewForEpisode($ep: Episode!, $review: ReviewInput!) {
  createReview(episode: $ep, review: $review) {
    stars
    commentary
  }
}
```
변수
```
{
  "ep": "JEDI",
  "review": {
    "stars": 5,
    "commentary": "This is a great movie!"
  }
}
```

결과
```
{
  "data": {
    "createReview": {
      "stars": 5,
      "commentary": "This is a great movie!"
    }
  }
}
```


















