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
REST 환경에서 몇몇의 요청은 서버에서 사이드 이펙트를 유발할수도 있지만,
데이터 수정시 GET 요청을 사용하지 않는게 컨벤션이다. graphql 도 이와 비슷한데..~

그러니까 아무튼 업데이트를 위한 키워드이다.


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

### 이 사이트에서는 이런식으로 queryQL 쿼리를 날려 볼 수 있다.
https://www.predic8.de/fruit-shop-graphql?query=mutation%20createSomething%20%7B%0A%20%20addCategory(id%3A%206%2C%20name%3A%20%22Green%20Fruits%22%2C%20products%3A%20%5B8%2C%202%2C%203%5D)%20%7B%0A%20%20%20%20name%0A%20%20%20%20products%20%7B%0A%20%20%20%20%20%20name%0A%20%20%20%20%7D%0A%20%20%7D%0A%7D%0A&operationName=createSomething




## Scheme and Types

### Type System
아래와 같이 그냥 암것도 선언안하고 기술할 수 있고
```
{
  hero {
    name
    appearsIn
  }
}
```

이렇게 hero 데이터에 name, appearsIn 에 타입을 정해서 기술 할수 있다.
```
{
  "data": {
    "hero": {
      "name": "R2-D2",
      "appearsIn": [
        "NEWHOPE",
        "EMPIRE",
        "JEDI"
      ]
    }
  }
}
```












