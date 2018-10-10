# 使用vue.js利用ipfs实现文件的上传下载

星际文件系统（InterPlanetary File System，缩写IPFS）是一个旨在创建持久且分布式存储和共享文件的网络传输协议。

不同于中心化的存储。

ipfs官网提供有ipfs-api，让web端也能够调用ipfs的api

```npm install ipfs-api```

说一下基础的ipfs配置步骤：
## 1.安装ipfs本地节点
去官网下载：https://dist.ipfs.io/#go-ipfs

打开控制台

```ipfs init```  

它会在你的```C:\Users\Administrator\.ipfs```创建一个ipfs节点文件
我们在config文件中先配置一下api接口的跨域



	"API": {
    "HTTPHeaders": {
      "Access-Control-Allow-Credentials": [
        "true"
      ],
      "Access-Control-Allow-Headers": [
        "Authorization",
        "X-Requested-With",
        "Range"
      ],
      "Access-Control-Allow-Methods": [
        "GET",
        "POST",
        "OPTIONS",
        "PUT"
      ],
      "Access-Control-Allow-Origin": [
        "*"
      ],
      "Server": [
        "go-ipfs/0.4.17"
      ]
    }
	}

随后命令行中启动ipfs

```ipfs daemon```

## 2.开始vue项目

vue先安装一下依赖包

```npm install```

```npm run dev```

引入ipfs

```import ipfsAPI from 'ipfs-api'```

#### 获取文件



	saveImageOnIpfs (reader) {
      return new Promise((resolve, reject) => {
        const buffer = Buffer.from(reader.result);
        this.ipfs.add(buffer).then((response) => {
          console.log(response)
          resolve(response[0].hash);
        }).catch((err) => {
          console.error(err)
          reject(err);
        })
      })
    },

### 下载文件

	downImageOnIpfs (hash) {
      return new Promise((resolve, reject) => {
        // const buffer = Buffer.from(reader.result);
        this.ipfs.cat(hash).then((response) => {
          console.log(response)
          resolve(response);
        }).catch((err) => {
          console.error(err)
          reject(err);
        })
      })
    },





我使用的下载插件是downloadjs

随后使用

	 const blob = new Blob([new Uint8Array(streamFile)]);
     console.log(blob)
     download(blob, this.fileTitle);

因为文件存储都是以uint8array格式，因此获取到后首先使用转化为blob格式，随后使用download下载





