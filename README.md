# elasticsearch_sample



```
docker-compose build
起動には少し時間がかかる
docker-compose up -d

止める
docker-compose down
全部消す
docker-compose down --rmi all
```



```
ヘルスチェック
curl -X GET "localhost:9200/_cat/health?v&pretty"
ノードごとのステータス
curl -X GET "localhost:9200/_cat/nodes?v&pretty"

customer情報を管理する時

作成
curl -X PUT "localhost:9200/customer?pretty&pretty"

インデックス確認
curl -X GET "localhost:9200/_cat/indices?v&pretty"


ドキュメント

登録
curl -X PUT "localhost:9200/customer/_doc/1?pretty&pretty" -H 'Content-Type: application/json' -d'
{
  "name": "John Doe"
}
'

確認
curl -X GET "localhost:9200/customer/_doc/1?pretty&pretty"

更新
curl -X POST "localhost:9200/customer/_update/1?pretty&pretty" -H 'Content-Type: application/json' -d'
{
  "doc": { "name": "Jane Doe", "age": 20 }
}
'

削除
curl -X DELETE "localhost:9200/customer/_doc/1?pretty&pretty"


バッチ実行

curl -X POST "localhost:9200/customer/_bulk?pretty&pretty" -H 'Content-Type: application/json' -d'
{"index":{"_id":"1"}}
{"name": "John Doe" }
{"index":{"_id":"1"}}
{"name": "Jane Doe" }
'

インデックス削除
curl -X DELETE "localhost:9200/customer?pretty&pretty"


検索
curl -X PUT 'localhost:9200/bank' -H 'Content-Type: application/json' -d'
{
    "settings" : {
        "index" : {
            "number_of_shards" : 5,
            "number_of_replicas" : 1
        }
    }
}
'


サンプルデータ投入
wget https://raw.githubusercontent.com/elastic/elasticsearch/master/docs/src/test/resources/accounts.json

bankインデックスに投入
curl -H "Content-Type: application/json" -X POST "localhost:9200/bank/_bulk?pretty&refresh" --data-binary "@accounts.json"

balance順でソートされたものが11〜20件
curl -X GET "localhost:9200/bank/_search?pretty" -H 'Content-Type: application/json' -d'
{
  "query": { "match_all": {} },
  "_source": ["account_number", "balance"],
  "from": 10,
  "size": 10,
  "sort": { "balance": { "order": "desc" } }
}
'


curl -X GET "localhost:9200/bank/_search?pretty" -H 'Content-Type: application/json' -d'
{
  "query": {
    "bool" : {
      "must" : {
        "match" : { "address" : "road" }
      },
      "filter": {
        "range" : { "age" : { "gte" : 20, "lte" : 30 } }
      },
      "must_not" : {
        "match" : { "address" : "mill" }
      },
      "should" : [
        { "range" : { "balance" : { "gte" : 30000 } } },
        { "match" : { "gender" : "F" } }
      ]
    }
  }
}
'

```




# 参考

基礎
https://qiita.com/kiyokiyo_kzsby/items/344fb2e9aead158a5545

Function Score
スコアリングできる
https://qiita.com/yosu/items/b2d837ebe95049e9556e

