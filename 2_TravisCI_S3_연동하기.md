# 2. TravisCI와 S3를 이용하여 정적 데이터 관리하기

최근에 많은 Github 저장소에서 유용한 정보들을 제공하고 있습니다.  
국내에는 다음과 같은 좋은 저장소들이 있는데요.

* [국내 개발 블로그 모음](https://github.com/sarojaba/awesome-devblog)
* [개발 관련 밋업, 세미나 모음](https://github.com/dev-meetup/dev-meetup.github.io)
* [기술 면접 모음](https://github.com/JaeYeopHan/Interview_Question_for_Beginner)
* [개발자 회고 모음](https://github.com/oaksong/developers-retrospective)
* [주니어 개발자 채용 정보](https://github.com/jojoldu/junior-recruit-scheduler)

본인이 이런 데이터를 모으고, 
이를 **Bot으로 안내해주는 서비스**를 만든다고 가정하시면 
어디에다 이 정보들을 관리할지가 고민일 수 있습니다.  
  
일반적인 웹 서비스처럼 생각한다면 데이터베이스에 저장해서 사용할수 있습니다.  
하지만 이처럼 **변경요소가 거의 없고, 조회만 대부분**인 상황에서 데이터베이스를 쓰기엔 비용이나 시간이 너무 아깝습니다.  
  
그래서 많은 저장소 운영자분들이  ```json``` 파일로 정보를 관리하십니다.    
예를 들어 제가 운영 중인 주니어 개발자 채용 정보의 경우 채용 정보를 ```db.json``` 으로 관리합니다.

![json](./images/2/json.png)

(예시)  
  
이렇게 **데이터를 json파일로 관리하고, 이 파일을 S3에 올려놓고 Bot에서 필요할때마다 호출**해서 사용하면 아주 저렴하고 간단하게 운영할 수 있게 됩니다.  
  
이번 시간엔 바로 이 과정을 진행하겠습니다.  

## 2-1. Travis CI & S3 연동하기

S3에 json파일을 올리는걸 사람이 수동으로 계속 할수는 없습니다.  
**json 파일을 고칠때마다 자동으로 S3에 업로드**되야만 편하겠죠?  
프로젝트의 파일을 고치고 git commit & push하면 다음 행위를 자동으로 하는걸 저희는 회사에서 자주 경험하고 있습니다.  
바로 **CI**입니다.  
  
젠킨스나 AWS Code Build, Travis CI 등 여러 CI툴을 통해 git push가 발생할때마다 build & test 가 자동으로 수행되는걸 많은 분들이 경험하고 계실거라 생각합니다.  
  
여기선 무료 CI 툴인 Travis CI를 통해 push가 발생하면 자동으로 json 파일을 S3에 올리도록 구성하겠습니다.  

### Travis CI에 저장소 등록

[Travis CI](https://travis-ci.org/)에 접속하셔서 저장소를 등록합니다.

![travis1](./images/2/travis1.png)

![travis2](./images/2/travis2.png)

활성화가 된 뒤에 다시 메인페이지로 가보시면 저장소가 등록된 것을 확인할 수 있습니다.

![travis3](./images/2/travis3.png)

현재 ```.travis.yml```이 없어 빌드를 진행할수 없습니다.  

```js
{
  "recruits": [
    {
      "team": "[배민아키텍처팀] 서버 개발자",
      "link": "http://bit.ly/2HL4FQs"
    },
    {
      "team": "[배민플랫폼개발실] 서버 개발자",
      "link": "http://bit.ly/2HL4FQs"
    },
    {
      "team": "[배민서비스개발실] 서버 개발자",
      "link": "http://bit.ly/2HL4FQs"
    },
    {
      "team": "웹 프론트엔드 개발자",
      "link": "http://bit.ly/2HL4FQs"
    }
  ]
}
```

## 2-2. AWS Lambda & S3 연동하기