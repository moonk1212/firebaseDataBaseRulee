# firebaseDataBaseRulee
파이어베이스 데이터베이스에 대해 정리한 내용(NoSql)

https://www.youtube.com/watch?v=WacqhiI-g_o&list=PLl-K7zZEsYLlP-k-RKFa7RyNPa9_wCH2s

RDBMS의 내용의 경우 최대한 간결하게 담았습니다.

### Rules Types

* .read: 데이터가 사용자들이 읽을 수 있도록 허용

* .write: 데이터가 쓰여딜 수 있게 허용

* .validate : 하위 속성이 있는지 여부와 데이터 유형이 올바로 형식화 된 값의 모양을 정의

* .indexOn : ordering 과 쿼리를 지원하기 위해 인덱스 할 자식을 지정

### RDBMS 와 NOSQL의 완전성과 유연성

* RDBMS 완전성 up, 유연성 down
* NOSQL 완전성 down, 유연성 up

### RDBMS의 테이블과 외부키

- 외부키로 서로의 테이블간 연결함

#### Nosql의 Json방식의 테이블 정의 

``` json
{
  "users" : {
    "1":{ /*1은 David의 기본키*/
      "name":"David"
    }
    "9" : {/*9은 Alice의 기본키*/
      "name":"Alice"
   }
  },
  "events": {
    "fm": {/*fm은 파이어베이스 밋업(MeetUp)이라는 이벤트입니다.*/
      "name" : "Firebase Meetup",
      "date" : 983275235320
    }
   },
     "eventAttendees": {
    "fm": {
      "1" : "David",
      "9" : "Alice"
    }
   }
}
```

```javascript
const db = firebase.database();
const events = db.child('events/fm');
const attendees = db.child('eventAttendees/fm');

events.on('value',snap=>{
//render data rto Html 
});

attendees.on('child_added',snap=>{
//append attendees to list 
});
```

### 쿼리 작성하는 방법

- 뷰에 맞춰서 데이터를 구조화하는 경향이 있음
- 이미 모든 필요한 부분들이 데이터베이스에 조직화된 상태로 준비

### RDBMS와 NOSQL의 검색 기능 만들기 비교

- 이거 잘되있다. 역시 킹갓 구글 ,,,
- 이런식으로 비교되니깐 머리에 쏙쏙 들어오네 ㅎ

```sql
select * from Events /*첫째 테이블에서 컬럼을 SELECt함*/
WHERE Name =="Firebase Meetuo"; /* where문을 써서 조건 둠*/
```
```javascript 
//부모키를 가리키는 레퍼런스를 생성
const db = firebase.database();
const events = db.child('events');

eventsRef.orderFunction().queryFunction();//순서를 정하는 orderFunction() 함수를 써서 제어, 좀더 세세한 제어는 queryFunction()-> 쿼리함수 사용
eventsRef.orderByKey().limitToFirst(10); //orderByKey() 순서를 정하는 함수,limitToFirst(10) 쿼리제한은 10개

/*orderBy~: 문자열로만 된 자식키에 대한 쿼리를 행할 수있음 =
orderByKey() : 개수를 제한하거나 기본적인 페이징 작업을 할 때 사용합니다.
orderByChild('child_property') : sql상에서의 where과 비슷 , 가령 이름,연령 등과 같은 자식 속성을 지정한 후 그 값에 대한 쿼리 짜기 가능 

orderByValue() : 모든 자식이 값을 기준으로 정렬됨
1. 자식키들은 SQL 데이터베이스에서의 기본키와 비슷(okㅎㅎ)
//범위 지정 
startAt('value')
endAt('value')

//where 문과 비슷
equalTo('child_key')
//상위 10줄
limitToFirst(10)
//하위 10줄
limitToLast(10)

모두 결합 가능 (워후 유용하겠군)

*/
```
```javascript
const db = firebase.database();
const events = db.child('events');
const query = events.orderByChild('name')
                    .equalTo('Firebase Meetup')
                    .limitToFirst(1);
//이런식으로 결합가능 
//문장이해 해보자 
```
```javascript
import * as firebase from 'firebase';
/*
{
"users":{
  "1": {
  "name": "David",
  "email": "deve@gmail.com",
  "age":99,
  "location":"SF",
  "age_location":"99_SF"
  },
 "9": {
  "name": "Alice",
  "email": "alice@gmail.com",
  "age":28
  "location":"Berlin",
  "age_location":"28_Berln"
  }
  
}
*/
```
## once와 on의 차이점
- once는 데이터 한번만 
- on은 지속적으로

## 비정규화와 정규화
-정규화는 중복된 데이터를 줄이는 것 :RDBMS의 가장 큰 특징
- 비정규화는 데이터를 중복시키는 것: firebase의 특징
  - 즉 쿼리를 줄이기 위해 데이터를 중복 시킴 
  - 대체적으로 뷰에 따라서 데이터의 구조를 만든다.
  아래의 예시 참고
  ```json
  "locationEvents":{
  "SF": {
    "fm":{
      "name":"Firebase Meetup",
      "timestamp": "12333",
      "location" :"SF"
      },
    },
    "BER":{
    "fbs":{
      "name":"Firebase Dev",
      "timestamp" : "1635",
      "location" :"BER"
    }
   }
  }
  ```
  -locationEvent를 사용하여 SF,BER을 키로 생각하고 location정보에 SF와 BER저장
  그래서 밑에 처럼 사용한다. 
  ```javascript
  const ref= firebase.database()l,ref();
  const berEvents = ref.child('locationEvents/BER');
  berEvents.on('child_added',snap => {
  });
  ```
## 데이터의 일관성

- 비정규화를 하면 데이터의 중복을 줘서 복잡한 쿼리를 짜지 않아도 됨'
- 하지만 문제가 생김 데이터가 바뀌었을 때 데이터의 일관성이 사라질 수도 있음
- 이것을 해주는 것이 다중 경로 갱신

## 다중 경로 갱신

- 비정규화와 일관성을 연결하는 것
- lookuptable을 만들면됨
```json
"userEvents": {

"$uid":{
  "$eventId":true
  }
}

```
-위의 코드 내용이 아래의 코드 내용

```json
"userEvents": {
  "1":{
  "fm":true,
  "other":true,
  "coolEvent":true,
  }
}
```
