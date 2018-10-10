<template>
  <div>
    <input type="file" name="" ref="file">
    <button @click="save">上传</button>
    <h2>{{fileTitleHash}}</h2>
    <h2>{{fileTitle}}</h2>
    <button @click="down">下载</button>
  </div>
</template>

<script>
  import ipfsAPI from 'ipfs-api';
  import download from 'downloadjs';
  import utf8ToBlob from 'typedarray-to-buffer';

export default {
  name: 'HelloWorld',
  data () {
    return {
      ipfs: null,
      fileTitleHash: 'Welcome to Your Vue.js App',
      fileTitle: null,
      fileHash: null
    }
  },
  mounted() {
    this.ipfs = ipfsAPI({host: 'localhost', port: '5001', protocol: 'http'});
  },
  methods: {
    Utf8ArrayToStr(array) {
      var out, i, len, c;
      var char2, char3;

      out = "";
      len = array.length;
      i = 0;
      while(i < len) {
      c = array[i++];
      switch(c >> 4)
        {
          case 0: case 1: case 2: case 3: case 4: case 5: case 6: case 7:
            // 0xxxxxxx
            out += String.fromCharCode(c);
            break;
          case 12: case 13:
            // 110x xxxx   10xx xxxx
            char2 = array[i++];
            out += String.fromCharCode(((c & 0x1F) << 6) | (char2 & 0x3F));
            break;
          case 14:
            // 1110 xxxx  10xx xxxx  10xx xxxx
            char2 = array[i++];
            char3 = array[i++];
            out += String.fromCharCode(((c & 0x0F) << 12) |
                           ((char2 & 0x3F) << 6) |
                           ((char3 & 0x3F) << 0));
            break;
          default:
            break;
        }
      }

      return out;
    }, 
    saveTextBlobOnIpfs (blob) {
      return new Promise((resolve, reject) => {
        const descBuffer = Buffer.from(blob, 'utf-8');
        this.ipfs.add(descBuffer).then((response) => {
          console.log(response)
          resolve(response[0].hash);
        }).catch((err) => {
          console.error(err)
          reject(err);
        });
      });
    },

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

    save(){
      console.log('上传');
      const file = this.$refs.file.files[0];
      let fileContentHash;

      let reader = new FileReader();
      reader.readAsArrayBuffer(file);
      reader.onloadend = (e) => {
        console.log(reader);
        this.saveImageOnIpfs(reader).then((hash) => {
          console.log(hash);
          fileContentHash = hash;
          this.fileHash = hash;
        }).then(() => {
          let fileName = file.name;
          const fileData = fileName + '&&' + fileContentHash
          console.log(fileData);
          this.saveTextBlobOnIpfs(fileData).then((hash) => {
            console.log(hash);
            this.fileTitleHash =  hash;
          });
        });
      }
    },

    down() {
      console.log('下载');

      this.downImageOnIpfs(this.fileTitleHash).then((stream) => {
        console.log(stream)
        const result = this.Utf8ArrayToStr(stream).split("&&");
        this.fileTitle = result[0];
        this.fileHash = result[1];

      }).then(() => {
        this.ipfs.cat(this.fileHash).then((streamFile) => {
          console.log(streamFile)
          const blob = new Blob([new Uint8Array(streamFile)]);
          console.log(blob)
          download(blob, this.fileTitle);
          // var foobar = new Uint8Array(streamFile);
          // var arrayBuffer = foobar.buffer; 
          // console.log(arrayBuffer)
          // const data = this.Utf8ArrayToStr(streamFile);
          // const arr = utf8ToBlob(data)
          // console.log(arr.readUInt16BE(0));
        })
      })
    }
  }
}
</script>

<!-- Add "scoped" attribute to limit CSS to this component only -->
<style scoped>
h1, h2 {
  font-weight: normal;
}
ul {
  list-style-type: none;
  padding: 0;
}
li {
  display: inline-block;
  margin: 0 10px;
}
a {
  color: #42b983;
}
</style>
