---
title: GO语言之Prometheus Exporter开发
date: 2020-11-25 00:52:59
tags: 
- Golang
- Prometheus
- Exporter
- 监控
categories: 
- Golang
- Prometheus
---



> ## 前言

蓝鲸监控通过 job 部署 prometheus 社区的 Exporter，对目标组件进行性能数据采集。接着 bkmetricbeat 从 Exporter 上周期拉取性能数据并通过数据通道上报。

### 自定义组件采集导入流程

蓝鲸监控当前支持使用 go 编写 Exporter

- 在社区找到适合自己的 Exporter 或者编写新的 Exporter

- 将源码编译成二进制文件

- 将编译的 Exporter 打成 zip 包

- 上传配置文件

<!-- more -->

## Exporter 开发

### Exporter 简介

- Exporter 本质上就是将收集的数据，转化为对应的⽂本格式，并提供 http 接口，供蓝鲸监控采集器 定期采集数据。

### Exporter 基础

- 指标介绍 Prometheus 中主要使⽤的四类指标类型，如下所示：

  - Counter (累加指标)
  - Gauge (测量指标)
  - Summary (概略图)
  - Histogram (直方图)

  最常使用的是 Gauge，Gauge 代表了采集的一个单个数据，这个数据可以增加也可以减少，比如 CPU 使用情况，内存使用量，硬盘当前的空间容量等。 Counter 一个累加指标数据，这个值随着时间只会逐渐的增加，比如程序完成的总任务数量，运行错误发生的总次数等，代表了持续增加的数据包或者传输字节累加值。

  > 【注】：所有指标的值仅支持 float64 类型。

- 文本格式以下面输出为例：

```bash
# metric:
sample_metric1 12.47
sample_metric2 {partition="c:"} 0.44
```

其中：

\#: 表示注释 sample_metric1 和 sample_metric2 表示指标名称

partition: 表示指标的作⽤维度，例如磁盘分区使⽤率，维度就是磁盘分区，即每个磁盘分区都有⼀个磁盘分区使⽤率的值

c: 表示维度的值，例如磁盘分区的 C 盘 / D 盘等 12.47 和 0.44 表示对应指标的值

### Exporter 开发

- 环境搭建：

  - Golang 安装
  - apt-get install git
  - wget https://dl.google.com/go/go1.10.7.linux-amd64.tar.gz
  - tar -C /usr/local -xzf go1.10.7.linux-amd64.tar.gz
  - export PATH=$PATH:/usr/local/go/bin
  - export GOPATH=`你的代码目录`

  > 不同系统安装介绍：https://github.com/astaxie/build-web-application-with-golang/blob/master/zh/01.1.md

- Prometheus Exporter 开发依赖库

  ⾸先引⼊ Prometheus 的依赖库

```bash
go get -v github.com/prometheus/client_golang/prometheus
```

- 开发示例

1. 导⼊依赖模块: 本例计划采集主机的内存和磁盘信息，因此引⼊以下依赖库

```bash
go get -v github.com/shirou/gopsutil
go get -v github.com/go-ole/go-ole
go get -v github.com/StackExchange/wmi
go get -v github.com/golang/protobuf/proto
go get -v golang.org/x/sys/unix
```

1. 新建⼀个 Exporter 项⽬： ⼀个 Exporter 只需要⼀个⽂件即可；在 GOPATH 下 src ⽬录下新建⼀个 test_exporter ⽬录和⼀个 test_exporter.go ⽂件: test_exporter.go ⽂件第⼀⾏必须写上 package main 可执⾏的命令必须始终使⽤ package main。

```go
import (
	"flag"
	"log"
	"net/http"

	"github.com/prometheus/client_golang/prometheus"
	"github.com/prometheus/client_golang/prometheus/promhttp"
	"github.com/shirou/gopsutil/disk"
	"github.com/shirou/gopsutil/mem"
)
```

1. 定义 Exporter 的版本（Version）、监听地址（listenAddress）、采集 url（metricPath）以及⾸⻚（landingPage）

```go
var (
	Version       = "1.0.0.dev"
	listenAddress = flag.String("web.listen-address", ":9601", "Address to listen on for web interface and telemetry.")
	metricPath    = flag.String("web.telemetry-path", "/metrics", "Path under which to expose metrics.")
	landingPage   = []byte("<html><head><title>Example Exporter" + Version + "</title></head><body><h1>Example Exporter" + Version + "</h1><p><ahref='" + *metricPath + "'>Metrics</a></p></body></html>")
)
```

1. 定义 Exporter 结构体

```go
type Exporter struct {
	error        prometheus.Gauge
	scrapeErrors *prometheus.CounterVec
}
```

1. 定义结构体实例化的函数 NewExporter

```go
func NewExporter() *Exporter {
	return &Exporter{}
}
```

1. Describe 函数，传递指标描述符到 channel，这个函数不⽤动，直接使⽤即可，⽤来⽣成采集指标的描述信息。

```go
func (e *Exporter) Describe(ch chan<- *prometheus.Desc) {
	metricCh := make(chan prometheus.Metric)
	doneCh := make(chan struct{})
	go func() {
		for m := range metricCh {
			ch <- m.Desc()
		}
		close(doneCh)
	}()
	e.Collect(metricCh)
	close(metricCh)
	<-doneCh
}
```

1. Collect 函数将执⾏抓取函数并返回数据，返回的数据传递到 channel 中，并且传递的同时绑定原先的指标描述符，以及指标的类型（Guage）；需要将所有的指标获取函数在这⾥写⼊。

```go
//collect 函数，采集数据的⼊⼝
func (e *Exporter) Collect(ch chan<- prometheus.Metric) {
	var err error
	// 每个指标值的采集逻辑，在对应的采集函数中
	if err = ScrapeMem(ch); err != nil {
		e.scrapeErrors.WithLabelValues("mem").Inc()
	}
	if err = ScrapeDisk(ch); err != nil {
		e.scrapeErrors.WithLabelValues("disk").Inc()
	}
}
```

1. 指标仅有单条数据，不带维度信息示例如下：

```go
func ScrapeMem(ch chan<- prometheus.Metric) error {
	// 指标获取逻辑，此处不做具体操作，仅仅赋值进⾏示例
	mem_info, _ := mem.VirtualMemory()
	// ⽣成采集的指标名
	metric_name := prometheus.BuildFQName("sys", "", "mem_usage")
	// ⽣成 NewDesc 类型的数据格式，该指标⽆维度，[] string {} 为空
	new_desc := prometheus.NewDesc(metric_name, "Gauge metric with mem_usage", []string{}, nil)
	// ⽣成具体的采集信息并写⼊ ch 通道
	metric_mes := prometheus.MustNewConstMetric(new_desc,
		prometheus.GaugeValue, mem_info.UsedPercent)
	ch <- metric_mes
	return nil
}
```

1. 指标有多条数据，带维度信息示例如下：

```go
func ScrapeDisk(ch chan<- prometheus.Metric) error {
	fs, _ := disk.Partitions(false)
	for _, val := range fs {
		d, _ := disk.Usage(val.Mountpoint)
		metric_name := prometheus.BuildFQName("sys", "", "disk_size")
		new_desc := prometheus.NewDesc(metric_name, "Gauge metric with disk_usage", []string{"mountpoint"}, nil)
		metric_mes := prometheus.MustNewConstMetric(new_desc,
			prometheus.GaugeValue, float64(d.UsedPercent), val.Mountpoint)
		ch <- metric_mes
	}
	return nil
}
```

1. 主函数

```go
func main() {
	// 解析定义的监听端⼝等信息
	flag.Parse()
	// ⽣成⼀个 Exporter 类型的对象，该 exporter 需具有 collect 和 Describe ⽅法
	exporter := NewExporter()
	// 将 exporter 注册⼊ prometheus，prometheus 将定期从 exporter 拉取数据
	prometheus.MustRegister(exporter)
	// 接收 http 请求时，触发 collect 函数，采集数据
	http.Handle(*metricPath, promhttp.Handler())
	http.HandleFunc("/", func(w http.ResponseWriter, r *http.Request) {
		w.Write(landingPage)
	})
	log.Fatal(http.ListenAndServe(*listenAddress, nil))
}
```

1. 编译 Exporter

```bash
go build test_exporter.go
./test_exporter
```

1. 运⾏起来后，访问 http://127.0.0.1:9601/metrics 即可验证

⾄此 Exporter 开发完成，其中 8，9 两步中的函数是重点，⽬前仅仅写了⼀些数据进⾏示例，其中的监控指标获取数据就是该部分的主要功能，需要编写对应逻辑获取指标的值。

## 制作⼀键导⼊包

### Exporter 编译

蓝鲸监控 Exporter 默认只⽀持 64 位机器运⾏ Exporter。

#### linux 系统

```bash
# 编译 windows exporter
env CGO_ENABLED=0 GOOS=windows GOARCH=amd64 go build -o ./exporterwindows.exe test_exporter
# test_exporter 为 GOPATH 下我们创建的⽬录名
# 编译 linux exporter
env CGO_ENABLED=0 GOOS=linux GOARCH=amd64 go build -o ./exporter-linux
test_exporter
```

#### windows 系统

```bash
# 编译 windows exporter
SET CGO_ENABLED=0
SET GOOS=windows
SET GOARCH=amd64
go build -o ./exporter-windows.exe test_exporter
# 编译 linux exporter
SET CGO_ENABLED=0
SET GOOS=linux
SET GOARCH=amd64
go build -o ./exporter-linux test_exporter
```



******

参考链接：
[蓝鲸监控 - Exporter 开发](https://bk.tencent.com/docs/document/5.1/19/600)