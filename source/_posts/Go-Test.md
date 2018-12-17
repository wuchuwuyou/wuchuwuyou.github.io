---
title: Go Test
date: 2018-12-17 22:44:34
tags: Go
categories: technology
published: true
keywords: GO,test,测试
---

# GO 测试
Go 程序测试分三类，功能测试(test)、基准测试(benchmark,也称性能测试)、以及事例测试(example)

测试文件命名规范为要测试的文件加上`_test`后缀，即如果被测试文件为`abc.go`则测试文件名为`abc_test.go`

每个测试文件必须至少包含一个测试函数

功能测试函数，其名称必须以Test为前缀，并且参数列表只应有一个` *testing.T`类型的参数声明，例如  `func TestHello(t *testing.T) {}`

性能测试函数，其名称必须以Benchmark为前缀，并且唯一参数的类型必须是 `*testing.B` 类型的，例如 `func BenchmarkHello(b *testing.B) {}`

示例测试函数，其名称必须以Example为前缀，但对函数的参数列表没有强制规定。
