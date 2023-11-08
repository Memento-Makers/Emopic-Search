# Emopic-Search
Emopic-Search 저장소에는 Emopic 프로젝트에 사용되는 검색서버를 구축하기 위한 파일들이 있습니다. 


## Table Of content

- [기술 스택](#기술-스택)
- [실행 하기](#실행하기)
- [참고사이트](#참고-사이트)
- [기여 하기](#기여하기)

## 기술 스택
![elasticsearch](https://img.shields.io/badge/elasticsearch-8.9.0-005571)
![kibana](https://img.shields.io/badge/kibana-8.9.0-005571)
![logstash](https://img.shields.io/badge/logstash-8.9.0-005571)


### 배포
![docker](https://img.shields.io/badge/docker-blue)
![docker-compose](https://img.shields.io/badge/docker_compose-3.8-blue)

## 실행하기

- 서버를 시작하기전에 서비스 DB의 데이터를 동기화 하기 위해 사용하고 있는 서비스의 DB Connector가 필요합니다. 
- Emopic 프로젝트는 서비스 DB가 Mysql이라 Mysql Connector를 사용해 데이터를 동기화 시켰습니다.

### Compose docker 

- 필요한 설정 파일는 docker-compose의 volume을 이용해 컨테이너 안으로 넣습니다. 

#### 1. 레포지토리 클론 
```shell 
git clone https://github.com/Memento-Makers/Emopic-Search.git
```

#### 2. DB Connetor 다운로드 및 logstash 설정
- DB Connetor를 volume으로 컨테이너에 넣을 수 있도록 다운로드 받은 파일 경로를 docker-compose.yml -> logstash에 적어주세요
- Logstash에서 서비스 DB에서 데이터를 받을 수 있도록 Connector 및 DB 설정을 해주세요.  

#### 3. 개발 환경 실행
```shell
docker compose up -d
``` 

#### 4. elastic search 설정

- nori 확장 플러그인 사용을 위해 다음 명령어를 수행한 뒤 elastic search를 재부팅 해야 합니다. 
```shell
#container 터미널에 접속합니다. 
docker exec -it <elasticsearch_conainer_id> /bin/bash
#nori 확장 플러그인 설치
bin/elasticsearch-plugin install analysis-nori
#설치 완료 후 container 터미널에서 빠져나와 재부팅 명령어를 실행합니다. 
docker restart <elasticsearch_conainer_id>
```
- nori 사용자 정의 사전이나 동의어 사전을 만드신다면 elasticsearch/config 안에 만들고 사용하는 것을 추천합니다. (파일 이름이나 사전의 경로가 달라질 경우 docker-compose.yml 에서 elasticsearch의 volume 설정 부분을 수정해주세요.)

### 5. index 생성
- kibana 주소 : <yourdomain or localhost>:5601
- [명령어 입력하는 콘솔](https://www.elastic.co/guide/en/kibana/current/console-kibana.html)
- 생성 명령어<details>
    ```
    PUT nori_photo
    {
      "settings": {
        "index": {
          "analysis": {
            "tokenizer": {
              "my_nori_tokenizer": {
                "type": "nori_tokenizer",
                "user_dictionary":"user_dictionary.txt"
                "decompound_mode": "mixed",
                "discard_punctuation": "false",
              }
            },
            "filter": { // 띄어쓰기 filter
              "stopwords": {
                "type": "stop",
                "stopwords": [" "]
              },
              "synonym": {// 동의어 filter 
                "type": "synonym",
                "synonyms_path": "synonyms.txt"
              }
            },
            "analyzer": {
              "my_nori_analyzer": {
                "type": "custom",
                "tokenizer": "my_nori_tokenizer",
                "filter": ["lowercase", "stop", "trim", "stopwords", "nori_part_of_speech"]
              },
              "my_nori_synonym_analyzer":{
                "type": "custom",
                "tokenizer": "my_nori_tokenizer",
                "filter": ["lowercase", "stop", "trim", "stopwords", "nori_part_of_speech","synonym"]
              }
            }
          }
        }
      },
      "mappings" : {
        "properties" : {
          "caption": {
            "type" : "text",
            //데이터 입력시 사용
            "analyzer": "my_nori_analyzer",
            //검색시 사용
            "search_analyzer": "my_nori_synonym_analyzer"
          },
          "class_list":{
            "type" : "text",
            "analyzer": "my_nori_analyzer",
            "search_analyzer": "my_nori_synonym_analyzer"
          },
          "snapped_at":{
            "type": "date",
            "format": "strict_date_optional_time||yyyyMMdd||yyyyMM||year||strict_year_month"
          }
        }
      }
    }

```
</details>
<div markdown="1">

## 참고 사이트
- docker-elk stack
    - https://github.com/deviantony/docker-elk
- nori
    - https://www.elastic.co/guide/en/elasticsearch/plugins/current/analysis-nori-tokenizer.html
    - https://esbook.kimjmin.net/06-text-analysis/6.7-stemming/6.7.2-nori

## [기여하기](docs/contribute.md)
