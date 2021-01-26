---
title: golang查询elasticsearch初体验
date: 2021-01-26 17:23:42
tags: golang elastictich
categories: k8s
thumbnail: https://ss1.bdstatic.com/70cFvXSh_Q1YnxGkpoWK1HF6hhy/it/u=463240096,4017976753&fm=26&gp=0.jpg
---

# 库选择
golang查询es目前有两个库，一个是第三方的[github.com/olivere/elastic](https://github.com/olivere/elastic)，一个是官方的[github.com/elastic/go-elasticsearch](https://github.com/elastic/go-elasticsearch),根据以往被坑的经验，果断选择使用官方库

# 查询语法
这个只能看文档了，推荐官方文档，还是比较好懂的[https://www.elastic.co/guide/cn/elasticsearch/guide/current/query-dsl-intro.html](https://www.elastic.co/guide/cn/elasticsearch/guide/current/query-dsl-intro.html)
研究了半天最后写的表达式如下
```json
{
  "query": {
    "match": {
      "log": {
        "query": "qweasd1 10.0.11.129 注册了系统",
        "operator": "and"
      }
    }
  }
}
```
放到kibana中测试一下，符合预期
![WX20210126-181745](https://tvax4.sinaimg.cn/large/9f8a45fbly1gn19ln2gmsj21z413bk29.jpg)

# 编写代码
```go
func search(c *gin.Context) {
	var (
		r map[string]interface{}
	)
	type SearchInfo struct {
		Word string `json:"word"`
	}
	var info SearchInfo
	if err := c.ShouldBindJSON(&info); err != nil {
		c.JSON(http.StatusOK, gin.H{"code": 1001, "message": err.Error()})
		return
	}
	var buf bytes.Buffer
	query := map[string]interface{}{
		"query": map[string]interface{}{
			"match_phrase": map[string]interface{}{
				"log":      info.Word,
				"operator": "and",
			},
		},
	}
	if err := json.NewEncoder(&buf).Encode(query); err != nil {
		log.Printf("Error encoding query: %s", err)
	}
	res, err := es.Search(
		es.Search.WithContext(context.Background()),
		es.Search.WithIndex("logs"),
		es.Search.WithBody(&buf),
		es.Search.WithTrackTotalHits(true),
		es.Search.WithPretty(),
	)
	if err != nil {
		log.Printf("Error getting response: %s", err)
		c.JSON(http.StatusOK, gin.H{"code": 1003, "message": err.Error()})
		return
	}
	defer res.Body.Close()

	if res.IsError() {
		var e map[string]interface{}
		if err := json.NewDecoder(res.Body).Decode(&e); err != nil {
			log.Fatalf("Error parsing the response body: %s", err)
		} else {
			// Print the response status and error information.
			log.Printf("[%s] %s: %s",
				res.Status(),
				e["error"].(map[string]interface{})["type"],
				e["error"].(map[string]interface{})["reason"],
			)
		}
	}
	if err := json.NewDecoder(res.Body).Decode(&r); err != nil {
		log.Printf("Error parsing the response body: %s", err)
	}
	// Print the response status, number of results, and request duration.
	log.Printf(
		"[%s] %d hits; took: %dms",
		res.Status(),
		int(r["hits"].(map[string]interface{})["total"].(map[string]interface{})["value"].(float64)),
		int(r["took"].(float64)),
	)
	// Print the ID and document source for each hit.
	for _, hit := range r["hits"].(map[string]interface{})["hits"].([]interface{}) {
		log.Printf(" * ID=%s, %s", hit.(map[string]interface{})["_id"], hit.(map[string]interface{})["_source"])
	}

	log.Println(strings.Repeat("=", 37))
	c.JSON(http.StatusOK, gin.H{"code": 0, "message": "OK"})
}
```

## 坑来了
```
2021/01/26 18:03:45 [400 Bad Request] parsing_exception: [match_phrase] query doesn't support multiple fields, found [log] and [operator]
```
这个库这种语法貌似不支持，换
```go
query := map[string]interface{}{
		"query": map[string]interface{}{
			"bool": map[string]interface{}{
				"must": []interface{}{
					map[string]interface{}{
						"match_phrase": map[string]string{
							"log": info.IP,
						},
					},
					map[string]interface{}{
						"match_phrase": map[string]string{
							"log": info.UserName,
						},
					},
					map[string]interface{}{
						"match_phrase": map[string]string{
							"log": info.Content,
						},
					},
				},
			},
		},
	}
```
每当这个时候，就无比怀念PHP的好

# 结果
```
2021/01/26 18:52:48 [200 OK] 1 hits; took: 362ms
2021/01/26 18:52:48  * ID=swyLPHcBVf2j-S8W5N2N, map[@timestamp:2021-01-26T02:35:08.178869714+00:00 docker:map[container_id:58e018a3bb44d1788aa1f8c7ae29708e224555cce714b18d30bc7ac3d56fe32c] kubernetes:map[container_image:cp-user-server:1.0.3 container_image_id:docker://sha256:015cc14bdab277d818775fb9fa1094f2d4056c65a91e355a8519713c7b2d9234 container_name:user host:hp labels:map[app:user pod-template-hash:868b789497] master_url:https://10.96.0.1:443/api namespace_id:598874aa-157d-46ad-a04a-2a069765a395 namespace_name:default pod_id:9b0242e2-e4fb-419a-a6aa-be6c0c6b3970 pod_name:user-868b789497-gsbsn] log:2021-01-26 10:35:08.178183 qweasd1 10.0.11.129 注册了系统
 stream:stdout tag:kubernetes.var.log.containers.user-868b789497-gsbsn_default_user-58e018a3bb44d1788aa1f8c7ae29708e224555cce714b18d30bc7ac3d56fe32c.log]
2021/01/26 18:52:48 =====================================
```
看结果成功取得了想要的数据，符合预期