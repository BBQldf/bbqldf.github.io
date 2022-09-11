---
layout:     post
title:     CGo学习-C返回值int类型无法和go的int类型比较
subtitle:   扩展学习
date:       2022-08-25
author:     ldf
header-img: img/post-bg-cgo01.png
catalog: true
tags:
    - CGo
    - 开发技巧
    - 跨语言编译
---


## 1、C返回值int类型无法和go的int类型比较


```
[root@VM-236-164-centos ~/qq_login_push]# go build
# git.woa.com/g_QQRTC/qq_login_push/loginclient
loginclient/query_get.go:51:9: invalid operation: res == QUERY_GET_SUCCESS (mismatched types _Ctype_int and int)
loginclient/query_get.go:67:28: invalid operation: res < QUERY_GET_SUCCESS (mismatched types _Ctype_int and int)
```

之前是：

```
//QUERYGETLEN 一次从loginClient中读取的用户信息条数
const QUERYGETLEN int = 100

//AVMAXLEN 从loginClient中读取的最大数据长度.LoginRecord长度在[40-130]之间
const AVMAXLEN int = QUERYGETLEN * 150

const (
	QUERY_GET_SUCCESS  = 10 //queryGet拉取数据成功
	QUERY_GET_EMPTY    = 1                  //queryGet拉取数据为空
	QUERY_GET_TOO_LONG = -411               //queryGet拉取数据超长
)
```



报错是：

```
//QUERYGETLEN 一次从loginClient中读取的用户信息条数
const QUERYGETLEN int = 100

//AVMAXLEN 从loginClient中读取的最大数据长度.LoginRecord长度在[40-130]之间
const AVMAXLEN int = QUERYGETLEN * 150

const (
	QUERY_GET_SUCCESS  = QUERYGETLEN //queryGet拉取数据成功
	QUERY_GET_EMPTY    = 1                  //queryGet拉取数据为空
	QUERY_GET_TOO_LONG = -411               //queryGet拉取数据超长
)
```

现在是：

```
//QUERYGETLEN 一次从loginClient中读取的用户信息条数
const QUERYGETLEN int = 100

//AVMAXLEN 从loginClient中读取的最大数据长度.LoginRecord长度在[40-130]之间
const AVMAXLEN int = QUERYGETLEN * 150

const (
	QUERY_GET_SUCCESS  = C.int(QUERYGETLEN) //queryGet拉取数据成功
	QUERY_GET_EMPTY    = 1                  //queryGet拉取数据为空
	QUERY_GET_TOO_LONG = -411               //queryGet拉取数据超长
)
```





## 2、Cgo中C代码返回值不合理导致数据丢失的问题

一个bug：

![](https://raw.githubusercontent.com/BBQldf/PicGotest/master/20220826012307.png)

利用C函数读取数据并拷贝的时候如果队列为空并不是在最大的条数（10条），会导致前面已经读取的数据被丢弃；比如，`LoginQuery_Get(record)1`如果读取了5条数据，没有读到第6条数据，这里直接返回(空为 -1)会导致前面5条数据丢失。

### 修改bug重新部署

C调用代码：

```c++
//AvLoginQueryGet 从共享内存中读取用户数据
int AvLoginQueryGet(char *avRecord, int avMaxLen,int *avRecordLen, int valueLen)
{
    for ( int i = 0; i < valueLen; i++){
        LoginRecord  record;
        int res = LoginQuery_Get(record);
        if(res == 0){
            auto recordStr = record.SerializeAsString();
            //query_get返回包大于指定最大长度，则返回-411
            if(recordStr.size() > avMaxLen){
                return -411;            
            }
            memcpy(avRecord,recordStr.c_str(),recordStr.size());
            avRecord += recordStr.size();
            *avRecordLen = recordStr.size();
            avRecordLen ++;
            avMaxLen -= recordStr.size();
        }else if (res == 1){
            return i;             //队列为空，返回读取到的数据条数
        }else{
            return res;     //拉取出错返回对应错误码
        }   
    }
    return valueLen;       //没有错误，返回预定义数据长度
}
```

go调用代码：

```go
// AvLoginQueryGet 获取一条上下线记录
func AvLoginQueryGet(ctx context.Context) ([]*login_record.LoginRecord, error) {
	valueAddress := make([]byte, AVMAXLEN)
	valueLen := make([]uint32, QUERYGETLEN)
	res := C.AvLoginQueryGet((*C.char)(unsafe.Pointer(&valueAddress[0])), C.int(AVMAXLEN),
		(*C.int)(unsafe.Pointer(&valueLen[0])), C.int(QUERYGETLEN))
	if res == QUERY_GET_SUCCESS {
		var beginLen uint32 = 0
		avRecordsRsp := make([]*login_record.LoginRecord, res)
		for index, _ := range avRecordsRsp {
			avRecordRsp := &login_record.LoginRecord{}
			if err := proto.Unmarshal(valueAddress[beginLen:beginLen+valueLen[index]], avRecordRsp); err != nil {
				metrics.IncrCounter("Cgo queryGet Unmarshal error", 1) //反序列化数据失败
				log.ErrorContextf(ctx, "Cgo queryGet Unmarshal error:%+v", err)
				return nil, err
			}
			metrics.IncrCounter("queryGet Unmarshal success", 1)
			avRecordsRsp[index] = avRecordRsp
			beginLen += valueLen[index]
		}
		metrics.IncrCounter("queryGet success", 1)
		return avRecordsRsp, nil
	} else if res >= 0 && res < QUERY_GET_SUCCESS { //出现队列为空的情况；照常把前面的数据反序列化
		var beginLen uint32 = 0
		avRecordsRsp := make([]*login_record.LoginRecord, res)
		for index, _ := range avRecordsRsp {
			avRecordRsp := &login_record.LoginRecord{}
			if err := proto.Unmarshal(valueAddress[beginLen:beginLen+valueLen[index]], avRecordRsp); err != nil {
				metrics.IncrCounter("Cgo queryGet Unmarshal error", 1) //反序列化数据失败
				log.ErrorContextf(ctx, "Cgo queryGet Unmarshal error:%+v", err)
				return nil, err
			}
			metrics.IncrCounter("queryGet Unmarshal success", 1)
			avRecordsRsp[index] = avRecordRsp
			beginLen += valueLen[index]
		}
		metrics.IncrCounter("queryGet Empty", 1) //队列数据拉取到空
		log.DebugContextf(ctx, "queryGet Empty.")
		time.Sleep(20 * time.Millisecond)
		return avRecordsRsp, nil
	} else if res == QUERY_GET_TOO_LONG {
		metrics.IncrCounter("queryGet too long", 1)
		log.DebugContextf(ctx, "queryGet too long.") //用户信息超长，告警
		return nil, errors.New("queryGet too long")
	} else {
		metrics.IncrCounter("queryGet error", 1)
		log.ErrorContextf(ctx, "queryGet error %+v", res) //拉取队列出错
		return nil, errors.New("queryGet error")
	}
}
```

