---
title: "如何在 Go 中发送表单请求"
date: 2018-03-06 16:51:25+0800
tags: [go]
---

通常我们与第三方交互使用的是 `json`，但偶尔也会遇到要求使用表单方式来提交数据，故今天我们就一起来学习下如何在 Go 中发送表单请求。

### 准备工作

首先我们有这样一段测试代码来接收 `POST` 请求，并返回其接收到的字段信息。

```golang
package main

import (
	"fmt"
	"log"
	"net/http"
)

func urlencodedHandler(w http.ResponseWriter, r *http.Request) {
	err := r.ParseForm()
	if err != nil {
		log.Printf("r.ParseForm(): %v", err)
		return
	}

	result := ""
	for k, v := range r.Form {
		result += fmt.Sprintf("%s:%v\n", k, v)
	}

	fmt.Fprintf(w, result)
}

func multipartHandler(w http.ResponseWriter, r *http.Request) {
	err := r.ParseMultipartForm(4 * 1024 * 1024)
	if err != nil {
		log.Printf("r.ParseForm(): %v", err)
		return
	}

	result := ""
	for k, v := range r.MultipartForm.Value {
		result += fmt.Sprintf("%s:%v\n", k, v)
	}

	for k, v := range r.MultipartForm.File {
		result += fmt.Sprintf("%s:%v\n", k, v)
	}

	fmt.Fprintf(w, result)
}

func main() {
	http.HandleFunc("/urlencoded", urlencodedHandler)
	http.HandleFunc("/multipart", multipartHandler)

	log.Fatal(http.ListenAndServe(":8080", nil))
}

```



### 发送 urlencoded 请求

`urlencoded` 主要用于纯文本请求，代码如下：

```
package main

import (
	"fmt"
	"io/ioutil"
	"net/http"
	"net/url"
	"strings"
)

func main() {
	payload := url.Values{}
	payload.Set("foo", "a")
	payload.Add("foo", "b")
	payload.Set("foo2", "c")

	req, err := http.NewRequest(http.MethodPost,
                            "http://localhost:8080/urlencoded",
                            strings.NewReader(payload.Encode()))
	if err != nil {
		return
	}

	req.Header.Add("Content-Type",
                  "application/x-www-form-urlencoded; param=value")

	resp, err := http.DefaultClient.Do(req)
	if err != nil {
		return
	}

	defer resp.Body.Close()

	data, _ := ioutil.ReadAll(resp.Body)

	fmt.Println(string(data))
}
```

运行 `go run urlencoded.go` 输出结果为：

```
foo2:[c]
foo:[a b]
```

### 发送 multipart 请求

`multipart` 主要用于发送文件上传的请求，代码如下：

```golang
package main

import (
	"bytes"
	"fmt"
	"io/ioutil"
	"log"
	"mime/multipart"
	"net/http"
)

func main() {
	buf := new(bytes.Buffer)

	writer := multipart.NewWriter(buf)
	writer.WriteField("foo", "a")
	writer.WriteField("foo", "b")

	part, err := writer.CreateFormFile("tmp.png", "tmp.png")
	if err != nil {
		return
	}

	fileData := []byte("hello,world")  // 此处内容可以来自本地文件读取或云存储
	part.Write(fileData)

	if err = writer.Close(); err != nil {
		return
	}

	req, err := http.NewRequest(http.MethodPost,
                              "http://localhost:8080/multipart",
                              buf)
	if err != nil {
		return
	}

	req.Header.Add("Content-Type", writer.FormDataContentType())

	resp, err := http.DefaultClient.Do(req)
	if err != nil {
		return
	}

	defer resp.Body.Close()

	data, _ := ioutil.ReadAll(resp.Body)

	fmt.Println(string(data))
}
```

运行 `go run multipart.go` 输出结果为：

```
foo:[a b]
tmp.png:[0xc420082410]
```
