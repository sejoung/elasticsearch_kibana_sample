
아래 내용은 두번째 kibana 수업을 진행하기 위한 선행 작업이다.

```
1-1 인덱스 매핑 (kibana)

PUT /s3-order-list
{
  "mappings": {
    "doc": {
      "properties": {
        "items": {
          "type": "nested"
        }
      }
    }
  }
}

1-2 s3.conf 생성 (터미널)
vi s3.conf
input {
    s3 {
        region => "ap-northeast-2"
        bucket => "elastic-day-20180425"
        prefix => "data/s3"
        codec => "json_lines"
#        access_key_id => ""
#        secret_access_key => ""
    }
}
filter {
}
output {
    elasticsearch {
        hosts => [ "<Elasticsearch EndPoint>" ]
        ssl => true
        user => "elastic"
        password => "<password>"
        index => "s3-order-list"
        manage_template => false
    }
}

1-3 logstash 실행 (터미널)
bin/logstash -f s3.conf

1-4 인덱스 생성 확인 (kibana)
GET _cat/indices

```

아래 내용은 실시간 데이터를 수집할수 있는 샘플 내용이다.

```

2-1 kinesis.conf 생성 (터미널)
vi kinesis.conf
input {
    kinesis {
        kinesis_stream_name => "elastic-day-20180425-kinesis"
        application_name => "elastic-day-20180425-<각자의 자리번호>"
        region => "ap-northeast-2"
        codec => "json"
    }
}
filter {
    date {
        match => [ "datetime", "UNIX_MS" ]
        timezone => "Asia/Seoul"
        target => "@timestamp"
    }
}
output {
    elasticsearch {
        hosts => [ "<Elasticsearch EndPoint>" ]
        ssl => true
        user => "elastic"
        password => "<password>"
        index => "kinesis-iot-%{+YYYY-MM-dd}"
        manage_template => false
    }
}

2-2 logstash 실행 (터미널)
bin/logstash -f kinesis.conf

2-3 인덱스 생성 확인 (kibana)
GET _cat/indices

```

