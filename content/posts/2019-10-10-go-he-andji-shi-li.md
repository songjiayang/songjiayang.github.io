---
title: "Go 和 Android 集成实战"
date: 2019-10-10 16:24:35+0800
tags: [go]
---

![cover.png](/images/goandroid/cover.png)

看到这个标题，你可能会问，为什么要在 Android 中运行 Go，直接使用 Java 不挺好吗？

是的，如果你有现成很强大的 Java 团队，这没有问题，但并不是所有团队都是如此。而且我在这里想强调的是 Android 与 Go 的集合，即在 Android 程序中使用 Go 而不是完全用 Go 来写 Android 程序。

我能想到在 Android 中用 Go 的一些原因：

- 团队熟悉 Go, 对 Java/Android 了解不多。
- 已经有现成的 Go 核心代码，比如开源类库: libp2p，turn/stun 类库等。
- 自己服务的 SDK 其核心逻辑复杂，繁琐，涉及大量网络或并发的操作。

能够在 Android 上使用 Go 代码，得益于 Go 强大的交叉编译能力，那该如何在 Android 上使用我们的 Go 库呢，接下来我将通过一个简单的示例来讲解。

## 实例教程

本例是在 Android 程序中使用 Go 编译的一个简单动态库来实现对网站测速的简单例子。

思路：

- Go 交叉编译为 Android 平台支持的 so 文件。
- 在 Android 中使用 JNA 调用该 so 文件。

依赖：

- [Go](https://golang.org/dl)
- [NDK r20](https://developer.android.com/ndk/downloads#stable-downloads)
- [JNA 5.4.0](https://github.com/java-native-access/jna)

说明: 演示环境为 Mac。

### 编写 Go 测试代码

- 编写 speedtester 的核心代码，实现对任意网站访问速度的检测：

```golang
package speedtester

import (
	"net/http"
	"time"
)

func Perform(url string) (int, error) {
	req, err := http.NewRequest(http.MethodGet, url, nil)
	if err != nil {
		return 0, err
	}

	startAt := time.Now()

	resp, err := http.DefaultClient.Do(req)
	cost := time.Now().Sub(startAt)
	if err != nil {
		return 0, err
	}
	defer resp.Body.Close()

	return int(cost / time.Millisecond), nil
}
```

- 编写 CGO 代码，暴露一个 `Perform` API 函数：

```go
package main

import "C"

import (
	"github.com/songjiayang/go-android/go/speedtester"
)

//export Perform
func Perform(url *C.char) C.int {
	cost, err := speedtester.Perform(C.GoString(url))
	if err != nil {
		return -1
	}
	return C.int(cost)
}

func main() {}
```

### 交叉编译动态链接库

前面一片文章 [如何在 Ubuntu 上交叉编译 ARM 架构的 CGO 程序](http://localhost:4000/posts/ru-he-kua-ping-tai-bian-yi-armde-cgocheng-xu) 中我已提到过如何交叉编译 CGO 代码，只不过当时交叉编译的平台是 Linux，现在我们使用类似的方式来编译 Android 版本。

交叉编译 Android 版本的动态库不仅需要指定 GCC 还要指定 NDK_TOOLCHAIN，所以我们先下载对应的 NDK 。

- Step1: 下载 ndk r20

```
wget https://dl.google.com/android/repository/android-ndk-r20-darwin-x86_64.zip
unzip android-ndk-r20-darwin-x86_64.zip
```

解压缩后，可以看到 ndk 目录为：

![ndk.png](/images/goandroid/ndk.png)

- Step2: 编译 toolchain

我们可以使用 ndk 自带的 make-standalone-toolchain.sh 脚本，编译特定的 toolchain，使用命令如下：

```
./android-ndk-r20/build/tools/make-standalone-toolchain.sh \
--toolchain=aarch64-linux-android-4.9 --platform=android-29 \
--install-dir=~/Library/toolchain --force
```

参数说明：

1. toolchain 参数表示对应 Android 的 ARCH，arm32 使用 arm-linux-androideabi-4.9，arm64 使用 aarch64-linux-android-4.9。
2. platform 参数表示对应 Android API的版本，25 对应 Android7.1.1，26 对应 Android8.0，27 对应Android8.1，28 对应 Android9.0，29 对应 Android10.0。
3. install-dir 参数表示编译的目标 toolchain 存放位置，后面交叉编译 Go 代码会用到。

- Step3: 执行交叉编译

使用刚刚编译好的 toolchain 来交叉编译：

```
CGO_ENABLED=1 GOOS=android NDK_TOOLCHAIN=~/Library/toolchain \ 
GOARCH=arm64 CC=~/Library/toolchain/bin/aarch64-linux-android-gcc \
go build -buildmode=c-shared -o libspeedtester.so api.go
```

此时会在目录下生成一个 `libspeedtester.so` 文件，通过 file 命令查看其信息如下：

```
libspeedtester.so: ELF 64-bit LSB shared object, ARM aarch64, version 1 (SYSV), dynamically linked, not stripped
```

### Android 代码集成

- Step1: 使用 Android Studio 新建一个工程，名叫 goandroid 的工程。

![app.png](/images/goandroid/app.png)

- Step2: 添加 jna 依赖

在 `build.gradle` 文件中的 `dependencies` 中添加依赖：

```
implementation 'net.java.dev.jna:jna:5.4.0'
```

![app02.png](/images/goandroid/app02.png)

- Step3: 添加 jna Android 平台依赖

在 app/libs 目录下新建一个叫做 arm64-v8a 的目录，目录下存放 `libjnidispatch.so` 以及我们的动态库 `libspeedtester.so` 文件，其次还要修改 `build.gradle` 文件，添加 `sourceSets`:

![app03.png](/images/goandroid/app03.png)

说明： 

1. `libjnidispatch` 文件需要根据不同的CPU架构来选择，可以点击连接 [jna/dist](https://github.com/java-native-access/jna/tree/master/dist) 下载对应平台的 jar 包，解压缩 jar 包后，可以提取 libjnidispatch 文件。
2. 手机不同架构，所新建的目录名称不同，比如我这里 arm64 的手机为 arm64-v8a，具体视情况而定。

- Step4: 新加一个 `SpeedTester` 接口来使用我们的 `libspeedtester.so` 库

```java
package com.example.goandroid;

import com.sun.jna.Library;
import com.sun.jna.Native;

public interface SpeedTester extends Library {
    SpeedTester INSTANCE = Native.load("speedtester", SpeedTester.class);

    int Perform(String url);
}
```

- Step5: 修改程序首页，调用 `SpeedTester`

修改 `activity_main.xml` 文件添加如下内容：

```
<?xml version="1.0" encoding="utf-8"?>
<LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:app="http://schemas.android.com/apk/res-auto"
    xmlns:tools="http://schemas.android.com/tools"
    android:layout_width="match_parent"
    android:orientation="vertical"
    android:layout_height="match_parent"
    tools:context=".MainActivity">

    <EditText
        android:id="@+id/urlInput"
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:hint="请输入要测试 URL"
        android:layout_marginLeft="10dp"
        android:inputType="text"
        />

    <Button
        android:id="@+id/onTest"
        android:layout_width="150dp"
        android:layout_height="45dp"
        android:text="点击测速" />

</LinearLayout>
```

修改 `MainActivity` 代码，使用 `SpeedTester` 进行测速。

```java
package com.example.goandroid;

import android.os.Bundle;
import android.util.Log;
import android.view.View;
import android.widget.EditText;

import androidx.appcompat.app.AppCompatActivity;

public class MainActivity extends AppCompatActivity {

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);

        // register id
        findViewById(R.id.onTest).setOnClickListener(new View.OnClickListener(){
            @Override
            public void onClick(View v) {
                EditText registerIdText =(EditText)findViewById(R.id.urlInput);
                String url = registerIdText.getText().toString();
                Log.d("GOAndroid","start speed testing with url: " + url);
                int cost = SpeedTester.INSTANCE.Perform(url);
                Log.d("GOAndroid", "finish speed testing with result: " + cost + "ms");
                return;
            };
        });

    }
}
```

- Step6: 修改 `AndroidManifest.xml`，开启网络权限

```
<uses-permission android:name="android.permission.INTERNET" />
```
- Step7: 真机模拟运行

程序首页：

![app04.png](/images/goandroid/app04.png)

输入 `http://jd.com`，点击测试，可以在 Logcat 中看到输出结果：

![app05.png](/images/goandroid/app05.png)

可以看到，在 Logcat 日志中已经打印输出了测试 http://jd.com 网站的速度了，说明在 Android 真机中调用我们的 Go 代码已经成功。

## 总结

今天我们通过一个 Android 程序中调用 Go 交叉编译的动态链接库的简单示例来演示了在 Android 中如何使用 Go，其过程大致为：

- 选用 NDK 的 make-standalone-toolchain.sh 编译本机环境的 toolchain。
- 使用编译出来的 toolchain 交叉编译 Android 系统的动态库。
- 在 Android 中使用 jna 来使用该动态库。

代码参考 [songjiayang/go-android](https://github.com/songjiayang/go-android) 。