---
title: Go语言请求蓝鲸ESB示例
date: 2020-09-04 01:46:20
tags: 
- Golang
- 网络编程
- 请求
- 蓝鲸ESB
categories: 
- Golang
- 网络编程
---

> ## 前言

之前学了Go语言网络编程，包括HTTP编程、Socket编程，现在我们使用Go实现蓝鲸API接口调用。

总所周知，Python的requests、urllib2都是http请求、爬虫的利器，那么Go语言该怎样做？

话不多说，直接上示例代码。

## 示例代码

```go
import (
	"bytes"
	"encoding/json"
	"fmt"
	"io/ioutil"
	"net/http"
)

// 定义了一个结构体
type BkResp struct {
	Code       int                    `json:"code"`
	Permission string                 `json:"permission"`
	Result     bool                   `json:"result"`
	RequestId  string                 `json:"request_id"`
	Message    string                 `json:"message"`
	Data       map[string]interface{} `json:"data"`
}

func main() {
	client := http.Client{}
	url := "http://paas.zw.com/api/c/compapi/v2/cc/search_business/"
	m1 := make(map[string]string)
	m1["bk_app_code"] = "bk_monitor"
	m1["bk_app_secret"] = "7c919c6c-3e07-42b3-b8ea-cc56b6a367d0"
	m1["bk_username"] = "admin"
	// json序列化
	params, err1 := json.Marshal(m1)
	if err1 != nil {
		fmt.Println(err1.Error())
		return
	}
	req_data := bytes.NewReader(params)
	request, err2 := http.NewRequest("POST", url, req_data)
	if err2 != nil {
		fmt.Println(err2.Error())
		return
	}
	// 设置请求头
	request.Header.Set("Content-Type", "application/json;charset=UTF-8")
	resp, err3 := client.Do(request)
	if err3 != nil {
		fmt.Println(err3.Error())
		return
	}
	respBytes, err4 := ioutil.ReadAll(resp.Body)
	if err4 != nil {
		fmt.Println(err4.Error())
		return
	}
	//fmt.Println(string(respBytes))
	var resp_data BkResp
	// 对响应体进行反序列化
	err5 := json.Unmarshal([]byte(string(respBytes)), &resp_data)
	if err5 != nil {
		fmt.Println(err5.Error())
		return
	}
	bk_info_list := resp_data.Data["info"]
	//fmt.Println(bk_info_list)
	//fmt.Println(reflect.TypeOf(bk_info_list))
	//for _, v := range bk_info_list.([]interface{}) {
	for _, v := range bk_info_list.([]interface{}) {
		fmt.Println(v)
		//for k1, v1 := range v.(map[string]interface{}) {
		// fmt.Println(k1)
		// fmt.Println(v1)
		//}
	}
}
```

