# elasticsearch_sample


### UbuntuにaptでElasticsearch

[ubuntuにaptでelasticsearchを入れる](https://self-development.info/elasticsearch%E3%81%AE%E7%B0%A1%E5%8D%98%E3%82%A4%E3%83%B3%E3%82%B9%E3%83%88%E3%83%BC%E3%83%AB%E3%80%90ubuntu-20-04%E3%80%91/)



### DockerでElasticsearch

2020年あたりまで動いてたが、2022年試したら動かなかった。    
dockerや何かのバージョンがエラーにさせてる。

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

customerインデックス作成
curl -X PUT "localhost:9200/customer?pretty&pretty"

全インデックス確認
curl -X GET "localhost:9200/_cat/indices?v&pretty"


ドキュメント

customerの1を登録
curl -X PUT "localhost:9200/customer/_doc/1?pretty&pretty" -H 'Content-Type: application/json' -d'
{
  "name": "John Doe"
}
'

1を確認
curl -X GET "localhost:9200/customer/_doc/1?pretty&pretty"

1を更新
curl -X POST "localhost:9200/customer/_update/1?pretty&pretty" -H 'Content-Type: application/json' -d'
{
  "doc": { "name": "Jane Doe", "age": 20 }
}
'

1を削除
curl -X DELETE "localhost:9200/customer/_doc/1?pretty&pretty"


バッチ実行

curl -X POST "localhost:9200/customer/_bulk?pretty&pretty" -H 'Content-Type: application/json' -d'
{"index":{"_id":"1"}}
{"name": "John Doe" }
{"index":{"_id":"1"}}
{"name": "Jane Doe" }
'

customerインデックス削除
curl -X DELETE "localhost:9200/customer?pretty&pretty"


サンプルデータ投入
wget https://raw.githubusercontent.com/elastic/elasticsearch/master/docs/src/test/resources/accounts.json

bankインデックスに投入

```
curl -H "Content-Type: application/json" -XPOST 'localhost:9200/bank/account/_bulk?pretty&refresh' --data-binary "@accounts.json"
```


bank検索
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




```
ドキュメントのスコアを自分で定義する
curl -X GET 'http://localhost:9200/bank/_search?pretty=true' -H 'Content-Type: application/json' -d '
{
  "query": {
    "function_score": {
      "query": {
        "match_all": {}
      },
      "boost_mode": "replace",
      "script_score" : {
        "script" : "doc[\"account_number\"].value * doc[\"balance\"].value"
      }
    }
  }
}
'


```

### Elasticsearch 8.0 を docker-compose

結論動かないんだけど、なんでか不明のまま。    
securityの設定は参考になる。


[Elasticsearch 8.0 を docker-compose](https://zenn.dev/fujimotoshinji/scraps/4fb4616976ee00)


### 公式

公式命だと思うけど、見ても解決できず。    

[ES公式](https://www.elastic.co/guide/en/elasticsearch/reference/7.17/docker.html)
    
[ESのPHPライブラリ](https://github.com/elastic/elasticsearch-php/tree/7.11)


# Laravelに繋げる時

composer require laravel/scout
composer require elasticsearch/elasticsearch

php artisan vendor:publish --provider="Laravel\Scout\ScoutServiceProvider"


```:config/scout.php
  'elasticsearch' => [
      'index' => env('ELASTICSEARCH_INDEX', 'scout'),
      'hosts' => [
          env('ELASTICSEARCH_HOST', 'http://localhost'),
      ],
```


php artisan make:provider ElasticsearchServiceProvider


```
<?php

namespace App\Providers;

use App\Scout\ElasticsearchEngine;
use Elasticsearch\ClientBuilder;
use Illuminate\Support\ServiceProvider;
use Laravel\Scout\EngineManager;

class ElasticsearchServiceProvider extends ServiceProvider
{
    /**
     * Register services.
     *
     * @return void
     */
    public function register()
    {
        resolve(EngineManager::class)->extend('elasticsearch', function ($app) {
            return new ElasticsearchEngine(
                config('scout.elasticsearch.index'),
                ClientBuilder::create()
                             ->setHosts(config('scout.elasticsearch.hosts'))
                             ->build()
                );
        });
    }

    /**
     * Bootstrap services.
     *
     * @return void
     */
    public function boot()
    {
        //
    }
}

```


```:config/app.php
App\Providers\ElasticsearchServiceProvider::class,
```


```:app/Scout/ElasticSearchEngine.php

```


# 参考

基礎
https://qiita.com/kiyokiyo_kzsby/items/344fb2e9aead158a5545

Function Score
スコアリングできる
https://qiita.com/yosu/items/b2d837ebe95049e9556e

