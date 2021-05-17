---
title:  "[Mongo] mongoimport 사용법"
date: 2021-05-03
excerpt: ""
tags: [til, mongo]
categories: [til/mongo]
---

특정 5만건 이상의 상품ID를 가지고 있는 document를 삭제해야 하는 경우가 생겼습니다. mongo에서는 `mongoimport` 커맨드로  `csv` 파일을 일괄 import 할 수 있는 기능이 있어 `mongoimport`로 특정 collection에 보정대상 ID를 넣고, 다시 target 컬렉션에서 해당 컬렉션을 참조하여 delete 한 과정을 기록합니다.


### 1. 엑셀파일 csv 파일로 저장

CSV는 몇 가지 필드를 쉼표(, )로 구분한 텍스트 데이터 및 텍스트 파일입니다. 확장자는 .csv이며 MIME 형식은 text/csv이다. comma-separated variables라고도 합니다.
[출처:위키피디아](https://ko.wikipedia.org/wiki/CSV_(%ED%8C%8C%EC%9D%BC_%ED%98%95%EC%8B%9D))

엑셀에서 다른이름으로 저장 > .csv 파일로 저장

### 2. csv 파일을 몽고 서버로 이동

![20210503_scp](/assets/20210503_scp.png)

`scp` 명령어를 이용해서 로컬에 있는 20210430_front_mongo_reimport.csv 파일을 `서버유저명@타깃서버IP`의 `:경로`로 파일을 전송합니다.


### 3-1. 일반 서버환경일 경우

``` console
/data01/program/mongodb/bin
```

몽고db가 설치되어 있는 경로에 /bin 을 보면 mongoimport 를 포함한 여러 실행파일을 볼 수 있다.

아래 문법에 따라 `monoimport`를 실행하면 된다.
``` console
mongoimport -d [DB_NAME] -c [COLLECTION_NAME] --type csv --file [IMPORT_FILE_NAME].csv --headerline
```

주의할점은, 엑셀 파일 내용에 컬럼명들이 있다면 --headerline을 붙여줘야 한다는 점이다.

참고
https://imgalib.wordpress.com/2015/06/25/mongodb-bulk-data-import-from-csv/

### 3-2. 도커 컨테이너에 올라와 있는 몽고서버일 경우

1. 먼저 도커 컨테이너에 target 파일을 이동시킨다. `cp` 옵션

``` console
#>docker exec -it <container-name> mongo
#>docker cp xxx.json <container-name-or-id>:/tmp/xxx.json
```

2. 몽고 컨테이너 이미지로 들어가서 `mongoimport` 커맨드를 수행한다.

``` console
#>docker exec <container-name-or-id> mongoimport -d <db-name> -c <c-name> --file /tmp/xxx.json
```

### 4. 컬렉션의 ID와 동일한 DOCUMENT 삭제


```json
var count = 0;
var ops = [];

conversionStage = {
   $addFields: {
      itemId: { $toString: "$ITEM_ID" }
   }
};

// bulk_temp : mongoimport로 import 한 컬렉션
db.bulk_temp.aggregate( [

  conversionStage

]).forEach(function(tgt) {
    ops.push({ "deleteOne": { "filter": { "_id": tgt.itemId }  } });

    if ( ops.length >= 1000 ) {
        db.getCollection('target_collection').bulkWrite(ops);
        ops = [];
    }
    count = count + 1;
});

if ( ops.length > 0 ) {
  db.getCollection('target_collection').bulkWrite(ops);
  ops = [];
}

print(count);
```

5만건 이상의 컬렉션을 삭제하기 때문에, `bulkWrite`로 수행한다.

### Trouble Shooting

- 만약 database의 비밀번호에 특수문자가 있다면, %인코딩을 해줘야 한다.

[%인코더 사이트](https://meyerweb.com/eric/tools/dencoder/)

- Authentication Failed

아무리 정보를 정확히 입력해도 아래 에러가 떴었다.
```
Failed: error connecting to db server: server returned error on SASL authentication step: Authentication failed
```

해결방법은 `--authenticationDatabase`를 추가하고, `--password '비밀번호'` 이렇게 비밀번호를 따옴표로 감싸서 실행하면 성공한다.


참고
https://stackoverflow.com/questions/49895447/i-want-to-execute-mongoimport-on-a-docker-container
https://stackoverflow.com/questions/50902635/mongodb-cant-import-a-csv-files

- csv 파일의 내용이 숫자라면, `mongoimport` 후에는 INT로 들어가 있다.

![20210503_mongo_bulk](https://i.imgur.com/mJlqJrs.png)

하지만 내가 지워야하는 컬렉션의 ID는 STRING 이었기 때문에 equal join 이 이뤄지지 않았다.

이 부분은 `$toString` 문법을 써서 컨버팅 후 join 한다.

``` json
zipConversionStage = {
   $addFields: {
      convertedString: { $toString: "$zipcode" }
   }
};
```

### 참고문서

[MongoDB Official Document](https://docs.mongodb.com/database-tools/mongoimport/)
