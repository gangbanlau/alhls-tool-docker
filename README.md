# Docker test environment for Apple HLS Low Latency Beta Tool

This is a script to run Apple's [HLS Low Latency Beta Tool](https://developer.apple.com/download/more/?=hls) with Docker containers.

## Pre requiments
+ You need [Docker Desktop for Mac](https://docs.docker.com/docker-for-mac/) to be installed on your machine.
+ You need [HLS Low Latency Beta Tool](https://developer.apple.com/download/more/?=hls) to be installed on your machine.
+ You need a high spec Mac since the tool encodes a full HD video. (I had a performance problem with MacBook Pro with 2 Core but it works fine with MacBook Pro with 4 Core.)
+ You need macOS Mojave or later.
+ You need [iOS 13 Beta or iPadOS 13 Beta](https://developer.apple.com/documentation/ios_ipados_release_notes) to be installed on a device.
+ You need [Xcode 11 Beta](https://developer.apple.com/documentation/xcode_release_notes/) to be installed on your machine.
+ Your machine and device need to be on the same network.

## Create self signed certificate
You can create a self signed certificate with:
```
$ ./init.sh
```

## Run HLS Low Latency Beta Tool with local Nginx
You can start running the HLS Low Latency Beta Tool and local Nginx with:
```
$ ./run.sh start
```

You can verify it with:
```
$ curl https://localhost/master.m3u8 --cacert ./webroot/server.crt
```

You can stop it with:
```
$ ./run.sh stop
```

## Setup device

### DNS setting on your device
Get your IP address on your machine.
```
$ ipconfig getifaddr en0
192.168.3.2
```

Change DNS server setting to use the IP address above on your device.

This [article](https://appleinsider.com/articles/18/04/22/how-to-change-the-dns-server-used-by-your-iphone-and-ipad) may be helpful to know how to do it.

**You need to delete default DNS servers and include only IP address of your machine.**

### Install self signed certificate
+ Open safari and browse to http://streaming.example.com/server.crt. Safari will prompt you to install the SSL certificate.
+ Open the Settings.app and navigate to General > About > Certificate Trust Settings, and find the streaming.example.com certificate, and switch it on to enable full trust for it

## Deploy HLS player app on device

There are several HLS player samples. I usually use [this one](https://developer.apple.com/documentation/avfoundation/media_assets_playback_and_editing/using_avfoundation_to_play_and_persist_http_live_streams).

### Adding the entitlement to your app
Follow the instructions on README.md in [HLS Low Latency Beta Tool](https://developer.apple.com/download/more/?=hls) to add the entitlement to your app.

## Play Low Latency HLS
Point your player at https://streaming.example.com/master.m3u8.
If you use [this sample app](https://developer.apple.com/documentation/avfoundation/media_assets_playback_and_editing/using_avfoundation_to_play_and_persist_http_live_streams), you can do this by adding the URL to Streams.plist.

## Nginx access logs
You can check the access log with:
```
docker-compose logs -f
```
This should help you to understand how Low Latency HLS works.

## Use your own domain and certificate.
If you have your own trusted certificate and key with your domain, you can use it.
Just replace webroot/server.crt and webroot/server.key with your certificate and key, and replace streaming.example.com with your FQDN.

I coudn't manage to use [Charles](https://www.charlesproxy.com/) with the self signed certificate that my init.sh creates. However, it works with a trusted certificate.

## Change tsrecompressor and mediastreamsegmenter options
It is interesting to change values of `target duration` and `partial target duration`. You can do this with replacing `PART_TARGET_DURATION` and `TARGET_DURATION` on run.sh.

If you want to use capture input instead of the default sample video, you can do it by adding `-c` to COMPRESSOR_COMMAND in run.sh

You can see more detail of usage with:
```
$ tsrecompressor --help
```