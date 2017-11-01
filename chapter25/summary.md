## 25. Asynchronous Task Queues

Asynchronous task queue는 생성된 때와 다른 시간에 수행되는 작업이고, 같은 순서로 동작하지 않을 수도 있다. 

#### 용어 정의

- Broker: task 저장소. Django에서는 RabbitMQ나 Redis를 보통 사용한다.
- Producer: 나중에 실행될 대기열에 task를 추가하는 코드. 
- Worker: broker에서 task를 꺼내와서 실행하는 코드.
- Serverless: 일반적으로 AWS Lambda와 같은 서비스로부터 제공받는다. 서버측 로직은 우리가 작성하지만 전통적인 architecture와 달리 3rd party로부터 완전히 관리되고 있는 stateless compute container에서 event-trigger되어 실행된다.



### 25.1 Task Queue가 필요한가?

- 결과를 처리하는데 시간이 걸리는 작업: 아마도 사용해야 할 수 있다.
- 사용자가 결과를 즉시 확인해야하거나, 확인할 수 있어야 할 때: 사용하지 않는 것이 좋다.

| Issue                                    | Use Task Queue? |
| ---------------------------------------- | --------------- |
| 대량 이메일 발송                                | yes             |
| 파일들을 수정할 때 (이미지를 포함한)                    | yes             |
| 대량의 데이터를 API로부터 받아올 때                    | yes             |
| 대량의 record를 table에 insert 하거나 update 할 때 | yes             |
| 사용자의 프로필을 업데이트 할 때                       | no              |
| 블로그나 CMS에 글을 더할 때                        | no              |
| 시간 집약적인 계산을 수행할 때                        | yes             |
| webhook을 받거나 보낼 때                        | yes             |



### 25.2 Task Queue Software 택하기

| Software                  | Pros                                     | Cons                                     |
| ------------------------- | ---------------------------------------- | ---------------------------------------- |
| Celery                    | Django와 python 표준, 유연하고 기능이 많으며 High volume에서 잘 동작함 | 설치하기 어려우며, 기초적인 부분 외에는 러닝커브가 있음          |
| DjangoChannels            | 사실상 django 표준, 유연하고 사용하기 편하며, django에서 websocket을 사용할 수 있음 | Redis만 용해야 하며, retry mechanism이 없음       |
| AWSLambda                 | 유연, 확장 가능성, 설치가 쉬움                       | API call이 느릴 수 있으며, 추가로 logging system이 필요하고, 복잡도가 올라감 |
| Redis-Queue, Huey, others | 상대적으로 설치가 쉽고, Celery에 비해 적은 메모리를 사용함     | 기능이 별로 없고, 보통 Redis만 사용해야 하며 커뮤니티가 활성화 되어있지 않음 |
| django-background-tasks   | 아주 쉬운 설치, 쉬운 사용법, windows에서도 동작하며, 소규모 작업에 사용하기 좋음 | medium-to-high volume에서 사용하기 부적합         |



### 25.3 Best Practices for Task Queues

#### 25.3.1 View처럼 다루어라

view에서는 가능한 작은 양의 코드를 포함하는 것이 좋다고 추천하였는데, task에서도 똑같이 적용할 수 있다고 생각한다. task 안의 코드들을 function에 넣고, function을 helper module 안으로, 그리고 task에서 그 function을 호출해서 사용하는 것을 통해서 task안에 길고 복잡한 코드덩어리를 두는 것을 피할 수 있다. 

#### 25.3.2 Tasks Aren't Free

memory와 resource 문제는 언젠가 발생한다. 항상 resource 사용량을 최소화 하라. serverless의 경우에는 엄청난 청구서로 다가온다.

#### 25.3.3 Only pass JSON-Serializable Values to Task Functions

view처럼 오직 json serialize를 할 수 있는 value들(integer, float, string, list, tuple, dictionary)만 통과시켜야 한다.

1. ORM instance의 경우 race condition 문제가 발생할 수 있다. 대신 primary key를 사용하여 신선한 데이터를 가져온 뒤 사용하자.
2. 복잡한 object serialize하여 task로 넘기는데 시간과 메모리가 소비된다. task queue를 사용함으로써 얻는 이득과 상충한다. 
3. debuggig이 쉽다.
4. 사용중인 task queue에 따라 다르지만, 오직 JSON-Serializable 허용된다. 

#### 25.3.4 Write Tasks as Idempotent Whenever Possible

task queue는 (성공한 task라 하더라도) 재시도의 가능성이 있기 때문에 항상 같은 결과를 얻을 수 있도록 task를 작성해야 한다.

#### 25.3.5 중요한 데이터를 queue 안에 보관하지 마라

django channels를 제외하고 전부 retry mechanism이 있지만, 이마저도 실패할 수 있다. 

#### 25.3.6 어떻게 Task와 worker들을 모니터하는지 배워라

https://pypi.python.org/pypi/flower

#### 25.3.7 로깅!

보이지 않게 동작하는 것이기 때문에, 정확히 어떻게 되어가고 있는지 알기 어렵다. [Sentry](https://sentry.io)같은 툴이 굉장히 유용할 것이다. Serverless task의 경우 sentry는 옵션이 아니라 필수 요소인 것 같다.  

#### 25.3.8 Backlog를 모니터하기

#### 25.3.9 죽은 task를 주기적으로 지워라

#### 25.3.10 필요없는 결과를 무시하라

#### 25.3.11 큐의 Error Handling을 사용하라

taks가 fail하면 어떤 일이 일어날까? 네트워크 에러, 3rd party api 다운, 혹은 여러가지 원인에 의해 발생할 수 있는 일인데, 사용하는 task queue가 어떻게 그것들을 처리하는지 찾아보자. 

- Max retries for a task
- Retry delays

#### 25.3.12 사용하는 Queue software의 기능들을 익혀라




