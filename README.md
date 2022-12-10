<p align='center'>
    <img src='./logo.png' width='200px' height='80px'/>
</p>
https://github.com/gwuhaolin/livego
测试通过步骤：
1 在手机上启动服务端，手机IP为192.168.0.109
./livego --flv_dir=./data --level=debug
2,在电脑端运行HTTP建立房间xiazemin：APP名为live
http://192.168.0.109:8090/control/get?room=xiazemin
网页显示
{"status":200,"data":"rfBd56ti2SMtYvSgD5xAV0YU99zampta7Z7S575KLkIZ9PYk"}


3 手机端运行：可以看到app名还是app没变化，把房间名改成编码了
HP-340-G3:~/Music$ ffmpeg -re -i 33bb.mp4 -c copy -f flv rtmp://192.168.0.109:1935/live/rfBd56ti2SMtYvSgD5xAV0YU99zampta7Z7S575KLkIZ9PYk
4,在播放端即可用rtmp://192.168.0.109:1935/live/xiazemin
http://192.168.0.109:7002/live/xiazemin.m3u8
http://192.168.0.109:7001/live/xiazemin.flv

[中文](./README_cn.md)

[![Test](https://github.com/gwuhaolin/livego/workflows/Test/badge.svg)](https://github.com/gwuhaolin/livego/actions?query=workflow%3ATest)
[![Release](https://github.com/gwuhaolin/livego/workflows/Release/badge.svg)](https://github.com/gwuhaolin/livego/actions?query=workflow%3ARelease)

Simple and efficient live broadcast server:
- Very simple to install and use;
- Pure Golang, high performance, and cross-platform;
- Supports commonly used transmission protocols, file formats, and encoding formats;

#### Supported transport protocols
- RTMP
- AMF
- HLS
- HTTP-FLV

#### Supported container formats
- FLV
- TS

#### Supported encoding formats
- H264
- AAC
- MP3

## Installation
After directly downloading the compiled [binary file](https://github.com/gwuhaolin/livego/releases), execute it on the command line.

#### Boot from Docker
Run `docker run -p 1935:1935 -p 7001:7001 -p 7002:7002 -p 8090:8090 -d gwuhaolin/livego` to start

#### Compile from source
1. Download the source code `git clone https://github.com/gwuhaolin/livego.git`
2. Go to the livego directory and execute `go build` or `make build`

## Use
1. Start the service: execute the livego binary file or `make run` to start the livego service;
2. Get a channelkey(used for push the video stream) from `http://localhost:8090/control/get?room=movie` and copy data like your channelkey.
3. Upstream push: Push the video stream to `rtmp://localhost:1935/{appname}/{channelkey}` through the` RTMP` protocol(default appname is `live`), for example, use `ffmpeg -re -i demo.flv -c copy -f flv rtmp://localhost:1935/{appname}/{channelkey}` push([download demo flv](https://s3plus.meituan.net/v1/mss_7e425c4d9dcb4bb4918bbfa2779e6de1/mpack/default/demo.flv));
4. Downstream playback: The following three playback protocols are supported, and the playback address is as follows:
    - `RTMP`:`rtmp://localhost:1935/{appname}/movie`
    - `FLV`:`http://127.0.0.1:7001/{appname}/movie.flv`
    - `HLS`:`http://127.0.0.1:7002/{appname}/movie.m3u8`
5. Use hls via https: generate ssl certificate(server.key, server.crt files), place them in directory with executable file, change "use_hls_https" option in livego.yaml to true (false by default)

all options: 
```bash
./livego  -h
Usage of ./livego:
      --api_addr string       HTTP manage interface server listen address (default ":8090")
      --config_file string    configure filename (default "livego.yaml")
      --flv_dir string        output flv file at flvDir/APP/KEY_TIME.flv (default "tmp")
      --gop_num int           gop num (default 1)
      --hls_addr string       HLS server listen address (default ":7002")
      --hls_keep_after_end    Maintains the HLS after the stream ends
      --httpflv_addr string   HTTP-FLV server listen address (default ":7001")
      --level string          Log level (default "info")
      --read_timeout int      read time out (default 10)
      --rtmp_addr string      RTMP server listen address
```

### [Use with flv.js](https://github.com/gwuhaolin/blog/issues/3)

Interested in Golang? Please see [Golang Chinese Learning Materials Summary](http://go.wuhaolin.cn/)


https://cloud.tencent.com/developer/article/2065186

golang源码阅读：livego直播系统
发布于2022-08-03 13:54:42阅读 3560
在分析源码之前，先搭建一个直播系统：

直播服务器

https://github.com/gwuhaolin/livego

播放站点

https://github.com/Bilibili/flv.js/

推流

https://github.com/obsproject/obs-studio

首先启动直播服务器

./livego --flv_dir=./data --level=debug
复制
1，在启动livego服务后默认会监听以下端口：

8090端口：用于控制台，通过HTTP请求可查看与控制直播房间的推拉流

1935端口：用于RTMP推拉流，目前貌似只能通过RTMP方式推流

7001端口：用于FLV拉流

7002端口：用于HLS拉流

2，创建直播房间：

请求：http://你的服务器地址:8090/control/get?room=房间名字（房间名字你自己自定义）

成功响应：{“status”:200,“data”:一段与房间名对应的MD5秘钥}

http://127.0.0.1:8090/control/get?room=xiazemin
{
status: 200,
data: "rfBd56ti2SMtYvSgD5xAV0YU99zampta7Z7S575KLkIZ9PYk"
}
复制
3，启动推流服务器

配置文件livego.yaml里可以看到默认的app名字，串流密码输入room名字，这里设置成xiazemin

server:
- appname: live
复制

4，启动网页服务器，拉取视频内容

<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>课程直播</title>
</head>
<body>
    <script src="flv.min.js"></script>
    <video id="videoElement" autoplay controls loop preload muted></video>
    <script>
        if (flvjs.isSupported()) {
            var videoElement = document.getElementById('videoElement');
            var flvPlayer = flvjs.createPlayer({
                type: 'flv',
                isLive: true,
                url: 'http://127.0.0.1:7001/live/rfBd56ti2SMtYvSgD5xAV0YU99zampta7Z7S575KLkIZ9PYk.flv'
            });
            flvPlayer.attachMediaElement(videoElement);
            flvPlayer.load();
            flvPlayer.play();
            document.body.addEventListener('mousedown', function(){
    var vdo = $("video")[0]; //jquery
    vdo.muted = false;
}, false);  
        }
</script>
</body>
</html>
复制
下载我们依赖的flv播放库

 https://cdn.bootcdn.net/ajax/libs/flv.js/1.6.1/flv.min.js
复制
启动一个静态服务器，返回上述网页

package main

import (
  "fmt"
  "net/http"
)

//http://newbt.net/ms/vdisk/show_bbs.php?id=56A46991BF52F1A048DB5F0350D74D01&page=0
func handler(w http.ResponseWriter, r *http.Request) {
  w.Header().Add("Access-Control-Allow-Origin", "*")
  w.Header().Add("Access-Control-Allow-Methods", "GET, DELETE, HEAD, OPTIONS,POST,PUT,PATCH")
  w.Header().Add("Access-Control-Allow-Headers", "x-requested-with,content-type")
  w.Header().Add("Access-Control-Allow-Credentials", "true")
  http.FileServer(http.Dir("./")).ServeHTTP(w, r)
  return
}

//https://stackoverflow.com/questions/69435888/how-to-ensure-cors-response-header-values-are-valid-when-querying-mongodb-in-n

func main() {
  http.HandleFunc("/", handler)
  if err := http.ListenAndServe(":8089", nil); err != nil {
    fmt.Println(err)
  }
}
复制
至此直播服务器搭建完毕。

livego还提供了其他推拉流管理的接口和统计信息相关的接口

http://127.0.0.1:8090/control/pull?&oper=start&app=live&name=xiazemin&url=rtmp://127.0.0.1:1935/live/xiazemin

{
status: 400,
data: "<h1>push url start rtmp://127.0.0.1:1935/live/xiazemin ok</h1></br>"
}
复制
http://127.0.0.1:8090/stat/livestat

{
status: 200,
data: {
publishers: [
{
key: "live/mPjVIUp97tQjNHJcaOAGeDZRvcMdGIASmHsVKxASAgqjn9FS",
url: "rtmp://localhost:1935/live/mPjVIUp97tQjNHJcaOAGeDZRvcMdGIASmHsVKxASAgqjn9FS",
stream_id: 1,
video_total_bytes: 1940374,
video_speed: 1465,
audio_total_bytes: 206931,
audio_speed: 161
}
],
players: null
}
}
复制
http://127.0.0.1:8090/control/push?&oper=start&app=live&name=xiazemin&url=rtmp://127.0.0.1:1935/live/xiazemin

{
status: 200,
data: "<h1>push url start rtmp://127.0.0.1:1935/live/xiazemin ok</h1></br>"
}
复制
http://127.0.0.1:8090/control/pull?&oper=stop&app=live&name=xiazemin&url=rtmp://127.0.0.1:1935/live/xiazemin
{
status: 400,
data: "<h1>push url stop rtmp://127.0.0.1:1935/live/xiazemin ok</h1></br>"
}
复制
        下面我们开始分析源码，整体包括三个部分，推流，拉流，推拉流管理。我从livego的入口开始main.go的main函数：创建了RTMP stream，然后粉笔起了拉流服务hls和httpflv，然后起了控制台服务器，最后起了推流服务器：

      rtmpServer = rtmp.NewRtmpServer(stream, nil)

      rtmpServer = rtmp.NewRtmpServer(stream, hlsServer)
      rtmpServer.Serve(rtmpListen)
复制
    func startHls() *hls.Server 
      hlsServer := hls.NewServer()
      go func() {
        hlsServer.Serve(hlsListen)
复制
func startHTTPFlv(stream *rtmp.RtmpStream) 

      hdlServer := httpflv.NewServer(stream)
      go func() {
        hdlServer.Serve(flvListen)
复制
func startAPI(stream *rtmp.RtmpStream) 

      opServer := api.NewServer(stream, rtmpAddr)
      go func() {
        opServer.Serve(opListen)
复制
  stream := rtmp.NewRtmpStream()
    hlsServer = startHls()
    startHTTPFlv(stream)
    startAPI(stream)
    startRtmp(stream, hlsServer)
复制
推流是基于rtmp协议的，底层是基于tcp协议的，在循环中监听连接，每个连接起一个协程：

 protocol/rtmp/rtmp.go 

func (s *Server) Serve(listener net.Listener) (err error) 
      netconn, err = listener.Accept()
      conn := core.NewConn(netconn, 4*1024)
      go s.handleConn(conn)  
复制
func (s *Server) handleConn(conn *core.Conn) error 
      connServer := core.NewConnServer(conn)
      appname, name, _ := connServer.GetInfo()
      key, err := configure.RoomKeys.GetKey(name)
      channel, err := configure.RoomKeys.GetChannel(name)
       pushlist, ret := configure.GetStaticPushUrlList(appname); 
      s.handler.HandleWriter(flvWriter.GetWriter(reader.Info()))
复制
本质上做的工作就是把数据从输入channel copy到输出channel，数据存储在sync.Map里面：

 protocol/rtmp/stream.go 

type RtmpStream struct {
  streams *sync.Map //key
}
复制
func (rs *RtmpStream) HandleReader(r av.ReadCloser) 

        i, ok := rs.streams.Load(info.Key)
        ns := NewStream()
        rs.streams.Store(info.Key, ns)
        stream.AddReader(r)    
复制
func (rs *RtmpStream) HandleWriter(w av.WriteCloser)

        item, ok := rs.streams.Load(info.Key)
        rs.streams.Store(info.Key, s)
        s.AddWriter(w)    
复制
func (rs *RtmpStream) CheckAlive()

        rs.streams.Delete(key)
复制
	HLS（Http Live Streaming）传输内容包括两部分：一是M3U8描述文件，二是TS媒体文件。TS媒体文件中的视频必须是H264编码，音频必须是AAC或MP3编码。protocol/hls/hls.go 

type Server struct {
  listener net.Listener
  conns    *sync.Map
}  
复制
func (server *Server) Serve(listener net.Listener) error

      mux.HandleFunc("/", func(w http.ResponseWriter, r *http.Request) {
    server.handle(w, r)
  })
复制
func (server *Server) handle(w http.ResponseWriter, r *http.Request) 

      path.Base(r.URL.Path) == "crossdomain.xml" 
      switch path.Ext(r.URL.Path)
        case ".m3u8":
          key, _ := server.parseM3u8(r.URL.Path)
          conn := server.getConn(key)
          tsCache := conn.GetCacheInc()
          body, err := tsCache.GenM3U8PlayList()
          w.Header().Set("Access-Control-Allow-Origin", "*")
          w.Write(body)
        case ".ts":
          key, _ := server.parseTs(r.URL.Path)
          conn := server.getConn(key)
          tsCache := conn.GetCacheInc()
          item, err := tsCache.GetItem(r.URL.Path)
复制
	我们前面搭建的直播系统是基于http flv 协议的：protocol/httpflv/server.go 

type Server struct {
  handler av.Handler
}
复制
func (server *Server) Serve(l net.Listener) error

        mux.HandleFunc("/", func(w http.ResponseWriter, r *http.Request) {
    server.handleConn(w, r)
  })
        mux.HandleFunc("/streams", func(w http.ResponseWriter, r *http.Request) {
    server.getStream(w, r)
  })
复制
func (server *Server) handleConn(w http.ResponseWriter, r *http.Request) 

      path := strings.TrimSuffix(strings.TrimLeft(u, "/"), ".flv")
      paths := strings.SplitN(path, "/", 2)
      msgs := server.getStreams(w, r)
        for _, item := range msgs.Publishers {
      if item.Key == path {
        include = true
      writer := NewFLVWriter(paths[0], paths[1], url, w)
      server.handler.HandleWriter(writer)
复制
func (server *Server) getStream(w http.ResponseWriter, r *http.Request) 

      msgs := server.getStreams(w, r)
      w.Write(resp)
复制
func (server *Server) getStreams(w http.ResponseWriter, r *http.Request) *streams

      rtmpStream := server.handler.(*rtmp.RtmpStream)
      msgs := new(streams)
      rtmpStream.GetStreams().Range(func(key, val interface{}) bool {
        msg := stream{key.(string), s.GetReader().Info().UID}
        msgs.Publishers = append(msgs.Publishers, msg)
      rtmpStream.GetStreams().Range(func(key, val interface{}) bool {
        ws := val.(*rtmp.Stream).GetWs()
        ws.Range(func(k, v interface{}) bool {
          msg := stream{key.(string), pw.GetWriter().Info().UID}
          msgs.Players = append(msgs.Players, msg)
复制
 protocol/hls/cache.go

func (tcCacheItem *TSCacheItem) GenM3U8PlayList() ([]byte, error)
      w.Write(m3u8body.Bytes())
复制
func (tcCacheItem *TSCacheItem) GetItem(key string) (TSItem, error)

      item, ok := tcCacheItem.lm[key]
复制
type TSCacheItem struct {

  id   string
  num  int
  lock sync.RWMutex
  ll   *list.List
  lm   map[string]TSItem
}
复制
 protocol/httpflv/writer.go

type FLVWriter struct {
  Uid string
  av.RWBaser
  app, title, url string
  buf             []byte
  closed          bool
  closedChan      chan struct{}
  ctx             http.ResponseWriter
  packetQueue     chan *av.Packet
}
复制
 av/av.go 

type Handler interface {
  HandleReader(ReadCloser)
  HandleWriter(WriteCloser)
}
复制
	直播服务的管理控制接口实现在protocol/api/api.go：

type Server struct {
  handler  av.Handler
  session  map[string]*rtmprelay.RtmpRelay
  rtmpAddr string
}
复制
	它也实现了一个静态文件服务器，如果有静态文件，可以放在static里面：	

func (s *Server) Serve(l net.Listener) error 

   mux.Handle("/statics/", http.StripPrefix("/statics/", http.FileServer(http.Dir("statics"))))
   mux.HandleFunc("/control/push", func(w http.ResponseWriter, r *http.Request) {
    s.handlePush(w, r)
  }) 
   mux.HandleFunc("/control/pull", func(w http.ResponseWriter, r *http.Request) {
    s.handlePull(w, r)
  })   
  mux.HandleFunc("/control/get", func(w http.ResponseWriter, r *http.Request) {
    s.handleGet(w, r)
  })
   mux.HandleFunc("/control/reset", func(w http.ResponseWriter, r *http.Request) {
    s.handleReset(w, r)
  })
  mux.HandleFunc("/control/delete", func(w http.ResponseWriter, r *http.Request) {
    s.handleDelete(w, r)
  })
  mux.HandleFunc("/stat/livestat", func(w http.ResponseWriter, r *http.Request) {
    s.GetLiveStatics(w, r)
  })
复制
func (s *Server) handlePush(w http.ResponseWriter, req *http.Request) 

          //http://127.0.0.1:8090/control/push?&oper=start&app=live&name=123456&url=rtmp://192.168.16.136/live/123456
           oper == "stop"
            pushRtmprelay, found := s.session[keyString]
            pushRtmprelay.Stop()
            delete(s.session, keyString)
          else
            pushRtmprelay := rtmprelay.NewRtmpRelay(&localurl, &remoteurl)
            err = pushRtmprelay.Start()
            s.session[keyString] = pushRtmprelay      
复制
func (s *Server) handlePull(w http.ResponseWriter, req *http.Request) 

          //http://127.0.0.1:8090/control/pull?&oper=start&app=live&name=123456&url=rtmp://192.168.16.136/live/123456
          oper == "stop"
            pullRtmprelay, found := s.session[keyString]
            pullRtmprelay.Stop()
          else
            pullRtmprelay := rtmprelay.NewRtmpRelay(&localurl, &remoteurl)
            err = pullRtmprelay.Start()
            s.session[keyString] = pullRtmprelay    
复制
func (s *Server) handleGet(w http.ResponseWriter, r *http.Request) 
          //http://127.0.0.1:8090/control/get?room=ROOM_NAME
          msg, err := configure.RoomKeys.GetKey(room)    
复制
func (s *Server) handleReset(w http.ResponseWriter, r *http.Request) 
          //http://127.0.0.1:8090/control/reset?room=ROOM_NAME
          msg, err := configure.RoomKeys.SetKey(room)      
复制
func (s *Server) handleDelete(w http.ResponseWriter, r *http.Request)
          //http://127.0.0.1:8090/control/delete?room=ROOM_NAME
          configure.RoomKeys.DeleteChannel(room)     
复制
func (server *Server) GetLiveStatics(w http.ResponseWriter, req *http.Request)
          //http://127.0.0.1:8090/stat/livestat
          rtmpStream := server.handler.(*rtmp.RtmpStream)
          rtmpStream.GetStreams().Range(func(key, val interface{}) bool {
            v := s.GetReader().(*rtmp.VirReader)
              msg := stream{key.(string), v.Info().URL, v.ReadBWInfo.StreamId, v.ReadBWInfo.VideoDatainBytes, v.ReadBWInfo.VideoSpeedInBytesperMS,
              v.ReadBWInfo.AudioDatainBytes, v.ReadBWInfo.AudioSpeedInBytesperMS}
          rtmpStream.GetStreams().Range(func(key, val interface{}) bool {
            v := pw.GetWriter().(*rtmp.VirWriter)
            msg := stream{key.(string), v.Info().URL, v.WriteBWInfo.StreamId, v.WriteBWInfo.VideoDatainBytes, v.WriteBWInfo.VideoSpeedInBytesperMS,
                v.WriteBWInfo.AudioDatainBytes, v.WriteBWInfo.AudioSpeedInBytesperMS}
          roomInfo, exists := (rtmpStream.GetStreams()).Load(room)
      http.Serve(l, JWTMiddleware(mux))
              ErrorHandler: func(w http.ResponseWriter, r *http.Request, err string) {
        res := &Response{
          w:      w,
          Status: 403,
          Data:   err,
        }
        res.SendJson()
      },
复制
 protocol/rtmp/rtmprelay/rtmprelay.go 

type RtmpRelay struct {
  PlayUrl              string
  PublishUrl           string
  cs_chan              chan core.ChunkStream
  sndctrl_chan         chan string
  connectPlayClient    *core.ConnClient
  connectPublishClient *core.ConnClient
  startflag            bool
}  
复制
func (self *RtmpRelay) Start() error 

      self.connectPlayClient = core.NewConnClient()
      self.connectPublishClient = core.NewConnClient()
      err := self.connectPlayClient.Start(self.PlayUrl, av.PLAY)
      err = self.connectPublishClient.Start(self.PublishUrl, av.PUBLISH)
      go self.rcvPlayChunkStream()
      go self.sendPublishChunkStream()
复制
func (self *RtmpRelay) rcvPlayChunkStream() {
      err := self.connectPlayClient.Read(&rc)
            r := bytes.NewReader(rc.Data)
      vs, err := self.connectPlayClient.DecodeBatch(r, amf.AMF0)
      self.cs_chan <- rc
复制
func (self *RtmpRelay) sendPublishChunkStream() 

      self.connectPublishClient.Write(rc)
      self.connectPublishClient.Close(nil)
复制
 configure/channel.go

var RoomKeys = &RoomKeysType{
  localCache: cache.New(cache.NoExpiration, 0),
}
复制
type RoomKeysType struct {

  redisCli   *redis.Client
  localCache *cache.Cache
}
复制
func (r *RoomKeysType) SetKey(channel string) (key string, err error) 

      key = uid.RandStringRunes(48)
      if _, err = r.redisCli.Get(key).Result(); err == redis.Nil {
      err = r.redisCli.Set(channel, key, 0).Err()
      err = r.redisCli.Set(key, channel, 0).Err()
复制
func (r *RoomKeysType) DeleteChannel(channel string) bool 

      r.redisCli.Del(channel).Err() 
      key, ok := r.localCache.Get(channel)
      r.localCache.Delete(channel)
      r.localCache.Delete(key.(string))
复制
文章分享自微信公众号：

golang算法架构leetcode技术php
复制公众号名称
本文参与 腾讯云自媒体分享计划 ，欢迎热爱写作的你一起参与！

作者：技术牛犊
原始发表时间：2021-12-19
如有侵权，请联系 cloudcommunity@tencent.com 删除。

