## 1. Install prerequisite
### docker
```bash
sudo apt update
```
```bash
sudo apt install apt-transport-https ca-certificates curl software-properties-common
```
```bash
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo gpg --dearmor -o /usr/share/keyrings/docker-archive-keyring.gpg
```
```bash
echo "deb [arch=$(dpkg --print-architecture) signed-by=/usr/share/keyrings/docker-archive-keyring.gpg] https://download.docker.com/linux/ubuntu $(lsb_release -cs) stable" | sudo tee /etc/apt/sources.list.d/docker.list > /dev/null
```
```bash
sudo apt update
```
```bash
apt-cache policy docker-ce
```
```bash
sudo apt install docker-ce
```
```bash
sudo systemctl status docker
```
### redis-cli
```bash
sudo apt install redis-tools
```

## 2. Install elk + redis
+ bootstrap: `docker compose up -d`
+ elasticsearch: http://localhost:9200
+ kibana: http://localhost:5601

## 3. Restart service
> scenario: after config changed

+ stop: `docker compose stop logstash`
+ start: `docker compose start logstash`

## 4. Insert into redis
+ connect: `redis-cli -h 127.0.0.1 -p 6379 -a '12345678'`
+ insert: `rpush news ${json string}`

### data strcuture
```json
{
  "sn": 3,
  "title": "台積熊本廠開幕",
  "content": "台積電熊本廠即將在2月24日盛大開幕，二廠也拍板動工，選擇在日本設廠",
  "published": "2024-01-01T03:00:00+08:00",
  "sort": 2,
  "tags": [
    "2024",
    "台積電",
    "台灣"
  ]
}
```
## 5. "Discovery" of kibana
![Diagram outlining the hierarchical organization of the LangChain framework, displaying the interconnected parts across multiple layers.](images/discovery.png "LangChain Architecture Overview")

## 6. "Dev Tools" of kibana
![dev-tools.png](images/dev-tools.png)

```json
GET redis-news/_search
{
  "query": {
      "bool": {
        "should": [
          {
            "wildcard": {
              "tags.keyword": {
                "value": "*日本*"
              }
            }
          },
          {
            "wildcard": {
              "title.keyword": {
                "value": "*日本*"
              }
            }
          },
          {
            "wildcard": {
              "content.keyword": {
                "value": "*日本*"
              }
            }
          }
        ]
      }
    },
  "sort": [
    {
      "sort": {
        "order": "asc"
      }
    }
  ]
}
```

## 7. elasticsearch api
### request
```bash
curl --request GET \
  --url http://localhost:9200/redis-news/_search \
  --header 'Content-Type: application/json' \
  --data '{
  "query": {
      "bool": {
        "should": [
          {
            "wildcard": {
              "tags.keyword": {
                "value": "*日本*"
              }
            }
          },
          {
            "wildcard": {
              "title.keyword": {
                "value": "*日本*"
              }
            }
          },
          {
            "wildcard": {
              "content.keyword": {
                "value": "*日本*"
              }
            }
          }
        ]
      }
    },
  "sort": [
    {
      "sort": {
        "order": "asc"
      }
    }
  ]
}'
```
### response
![api-response.png](images/api-response.png)

+ `hits.total.value`: 資料總筆數
+ `hits.hits[#]._source.tags`: news tags
+ `hits.hits[#]._source.sn`: primary key
+ `hits.hits[#]._source.sort`: 等級
+ `hits.hits[#]._source.title`: news 標點
+ `hits.hits[#]._source.content`: news 內容
+ `hits.hits[#]._source.published`: news發布時間
+ `hits.hits[#]._source.['@timestamp']`: 資料異動時間

## 99. example data to redis
```json
{"sn":1,"title":"2024台灣總統大選","content":"當選人:AAA","published":"2024-02-22T20:23:00+08:00","sort":2,"tags":["總統","2024","台灣"]}
```
```json
{"sn":2,"title":"2024美國總統大選","content":"當選人:BBB","published":"2024-09-20T21:53:00+08:00","sort":4,"tags":["總統","2024","美國"]}
```
```json
{"sn":3,"title":"以色列直攻哈瑪斯大堡壘","content":"自從以色列與哈瑪斯去年10月7日交戰以來，這已是布林肯第5度親自走訪中東。他接著還將前往以色列和卡達。","published":"2021-05-30T03:00:00+08:00","sort":3,"tags":["以色列"]}
```
```json
{"sn":4,"title":"台灣多處遭重油污染","content":"綠島遭重油汙染，連螃蟹都被重油覆蓋全身","published":"2024-01-01T12:00:00+08:00","sort":5,"tags":["2024", "台灣"]}
```
```json
{"sn":5,"title":"元旦不平靜！日本西部遭7.6地震襲擊 海嘯警報發布","content":"路透社、CNN、BBC 報導，根據美國地質調查局（USGS），這起淺層地震震央位在石川縣穴水町東北約 42 公里處，深度僅 10 公里。","published":"2024-01-01T01:00:00+08:00","sort":2,"tags":["2024","日本"]}
```
```json
{"sn":6,"title":"逾6成日企擔憂台灣局勢","content":"國際顧問公司安侯建業26日公布針對日本企業調查「何為經濟安全最重大風險因素」，結果顯示逾6成日企指出「台灣局勢」居冠，其次依序為「中國增強貿易管制」、「美對中貿易管制加劇」。","published":"2024-01-01T02:00:00+08:00","sort":2,"tags":["台積電","日本"]}
```
```json
{"sn":7,"title":"台積熊本廠開幕","content":"台積電熊本廠即將在2月24日盛大開幕，二廠也拍板動工，選擇在日本設廠","published":"2024-01-01T03:00:00+08:00","sort":2,"tags":["台積電","2024","台灣"]}
```