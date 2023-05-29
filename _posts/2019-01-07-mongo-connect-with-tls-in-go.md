---
title: "如何在 Go 中使用 TLS 连接 MongoDB"
date: 2019-01-07 14:38:44+0800
tags: [go, MongoDB]
---

通常我们的数据库都配置为内网访问，但由于业务部署架构的不同，有时也需要通过公网访问 MongoDB 数据库，此时为了防止被端口扫描和脱库，MongoDB 需要配置为 TLS 访问，那在 Go 中应该如何实现呢？

### 依赖

- 配置了 TLS 公网访问的 MongoDB 实例
- Go 的 MongoDB 驱动 [globalsign/mgo](github.com/globalsign/mgo)

### Go 实现代码：

```golang
package model

import (
	"crypto/tls"
	"crypto/x509"
	"errors"
	"github.com/globalsign/mgo"
	"io/ioutil"
	"log"
	"net"
)

func main() {
	dsn := "mongodb://user:password@host/database"

	dialInfo, err := mgo.ParseURL(dsn)
	if err != nil {
		log.Panic(err)
	}

	// read pemfile data
	pemData, err := ioutil.ReadFile("./pemfile")
	if err != nil {
		log.Panic(err)
	}

	roots := x509.NewCertPool()
	if !roots.AppendCertsFromPEM(pemData) {
		log.Panic(errors.New("failed to parse root certificate"))
	}

	// set tls config
	tlsConfig := &tls.Config{
		RootCAs:            roots,
		InsecureSkipVerify: true,
	}

	// update dialserver with tls Dial
	dialInfo.DialServer = func(addr *mgo.ServerAddr) (net.Conn, error) {
		conn, err := tls.Dial("tcp", addr.String(), tlsConfig)
		if err != nil {
			log.Println(err)
		}
		return conn, err
	}

	session, err := mgo.DialWithInfo(dialInfo)
	if err != nil {
		log.Panic(err.Error())
	}
	// db operation with session
}
```

通过以上代码，我们就能通过公网连接 tls 的 MongoDB 实例，当连接上后，其数据库的操作和内网连接一致。