
# Notes

- Tested with Ubuntu 14.04.5

# Hints

- Commercial
  - https://gstreamer.freedesktop.org/data/events/gstreamer-conference/2015/Luis%20de%20Bethencourt%20-%20How%20to%20Contribute%20to%20GStreamer.pdf
- `fdsink, fdsrc`
  - Is it possible to use it: http://developers-club.com/posts/204014/
  - Example of usage: https://www.raspberrypi.org/forums/viewtopic.php?t=49278&p=664616
- RTSP
  - https://gstreamer.freedesktop.org/data/events/gstreamer-conference/2013/GStreamer%20Conference%202013%20-%20Wim%20Taymans%20-%20Latest%20GStreamer%20RTSP%20Server%20features.pdf
  - https://gstreamer.freedesktop.org/modules/gst-rtsp-server.html
  - http://stackoverflow.com/questions/13744560/using-gstreamer-to-serve-rtsp-stream-working-example-sought
  - https://gstreamer.freedesktop.org/data/doc/gstreamer/head/gst-plugins-good-plugins/html/gst-plugins-good-plugins-rtspsrc.html
  - https://gstreamer.freedesktop.org/documentation/rtp.html
- Streaming: http://www.polytech2go.fr/topinternet/sp_inetmedia/Lect.03.Gstreamer.multimedia.and.internet.pdf
- Known misstakes
  - in `... ! videoconvert ! video/x-raw width=1920, height=1080, format=RGB ,bpp=24 ! ...` there is now `,` between `video/x-raw` and the afterwards comma-seperated list
  - Commonly, the sink of a plugin tells the source of the other plugin what it expects. But if not you can tell the sink with so called `caps` (capabilities) the target format`... ! videoconvert ! video/x-raw width=1920, height=1080, format=RGB ,bpp=24 ! ...`. These `caps` are just another element in the pipeline, but are no plugin.
  - The sink and source `caps` can be investigated, if you use the `-v` switch with the tools, ot with `GST_DEBUG=2`
- Building test applications: https://gstreamer.freedesktop.org/data/doc/gstreamer/head/pwg/html/chapter-building-testapp.html

# Known issues

- Using idintity plugin to tap images from a gstreamer pipeling into program: http://gstreamer-devel.966125.n4.nabble.com/Using-idintity-plugin-to-tap-images-from-a-gstreamer-pipeling-into-program-td4679750.html , http://gstreamer-devel.966125.n4.nabble.com/Using-identity-causes-memory-leak-Ported-application-from-gstreamer-0-10-to-gstreamer-1-0-td4679488.html
- Video-Acceleration (X Error of failed request: BadRequest' when using nvidia driver with video acceleration (VA)): http://askubuntu.com/questions/830183/x-error-of-failed-request-badrequest-when-using-nvidia-driver-with-video-acce/831111#831111

# Debugging

- Set debug output for current executed application (http://docs.gstreamer.com/display/GstSDK/gst-launch):
- `export GST_DEBUG=2` : Like `-v`in the console
- `export GST_DEBUG=5 2>&1 | grep caps`: Get the capabilities of the pipeline
- `export GST_DEBUG=5` : Ultra verbose

# testsource

## gstreamer-0.10

- `gst-launch-0.10 videotestsrc ! video/x-raw-rgb,bpp=24,width=1000,height=1000 ! autoconvert ! ximagesink`
- `gst-launch-0.10 videotestsrc ! autoconvert ! ximagesink`
- `gst-launch-0.10 videotestsrc ! ximagesink`

## gstreamer-1.0

- `gst-launch-1.0 videotestsrc ! video/x-raw, format="I420" ,width=640,height=480,framerate=30/1 ! identity name=artoolkit sync=true ! autovideosink`
- If `autovideosink` does not work (bacause of VA API stuff), one can use `xvimagesink`

# Video Playback

Using H.264 testsamples from `http://jell.yfish.us/`: `http://jell.yfish.us/media/jellyfish-3-mbps-hd-h264.mkv` as `video.mkv`

## gstreamer-0.10

- `gst-launch-0.10 -v playbin uri=file:////path/video.mkv`
- `gst-launch filesrc location=video.mkv ! decodebin2 name=dec ! ffmpegcolorspace ! autovideosink`
- don't know why this does not work: `./simpleTest "playbin uri=file:////opt/repositories/artoolkit5/bin/video.mkv ! video/x-raw-rgb,bpp=24 ! fakesink"`
- Hints
  - http://stackoverflow.com/questions/1576750/how-to-display-avi-video-with-gstreamer
  - Using Ubuntu14.04, first the missing ffmpeg decode/encoder needs to be installed, because ffmpeg was replaced by libav (https://askubuntu.com/questions/575869/how-do-i-install-gstreamer0-10-ffmpeg-on-ubuntu-14-10/575888)
    - `sudo add-apt-repository ppa:mc3man/gstffmpeg-keep`
    - `sudo apt-get update`
    - `sudo apt-get install gstreamer0.10-ffmpeg`

## gstreamer-1.0

- `gst-launch-1.0 -v playbin uri=file:///path/to/somefile.mp4`
- `gst-launch-1.0 -v playbin uri=file:///path/video.mkv`
- `gst-launch-1.0 -v filesrc location=/path/video.mkv ! matroskademux ! avdec_h264 ! autovideosink`
- `gst-launch-1.0 -v filesrc location=/path/video.mkv ! matroskademux ! avdec_h264 ! videoconvert ! autovideosink`
- Hints
  - GSTREAMER 1.0 Playback a file: https://gstreamer.freedesktop.org/data/doc/gstreamer/head/gst-plugins-base-plugins/html/gst-plugins-base-plugins-playbin.html
  - matroska: https://gstreamer.freedesktop.org/data/doc/gstreamer/head/gst-plugins-good-plugins/html/gst-plugins-good-plugins-matroskademux.html
  - RAW H264: http://blog.petehouston.com/2015/12/28/play-raw-h264-files-using-gstreamer/
  - RAW YUV: http://stackoverflow.com/questions/27419113/playing-a-raw-video-using-gst-launch

# Camera Playback

## gstreamer-0.10

- `gst-launch v4l2src ! xvimagesink`
- `gst-launch v4l2src ! video/x-raw-yuv,width=320,height=240,framerate=30/1 ! xvimagesink`
- `gst-launch v4l2src ! video/x-raw-yuv,format=\(fourcc\)YUY2,width=320,height=240 ! xvimagesink`

## gstreamer-1.0

- `gst-launch-1.0 v4l2src device="/dev/video0" ! video/x-raw,width=640,height=480 ! autovideosink`
- Convert for no reason: `gst-launch-1.0 v4l2src device="/dev/video0" ! videoconvert ! video/x-raw,format=RGB,width=640,height=480,framerate=30/1 ! videoconvert ! video/x-raw,format=I420,width=640,height=480,framerate=30/1 ! autovideosink`

# Shared Memory Playback

## gstreamer-1.0

From: https://mazdermind.wordpress.com/2014/08/29/getting-shmsinkshmsrc-to-work-with-videomixer/

- Server: `gst-launch-1.0   shmsrc socket-path=/tmp/foo !   video/x-raw,width=1000,height=1000,framerate=48/1,format=I420 !   autovideosink`
- Client (Can be started multiple time)
  - For display: `gst-launch-1.0 shmsrc socket-path=/tmp/foo ! video/x-raw, format=BGR ,width=640,height=480,framerate=30/1 ! videoconvert ! autovideosink`
  - For record: `gst-launch-1.0 shmsrc socket-path=/tmp/foo ! video/x-raw,width=640,height=480,framerate=25/1,format=I420 ! avimux ! filesink location=output.avi`
- With `mat2gstreamer`
  - Server: `./mat2gstreamer`
  - Client (My webcam has 640x480@30fps): `gst-launch-1.0 shmsrc socket-path=/tmp/foo ! video/x-raw, format=BGR ,width=640,height=480,framerate=30/1 ! videoconvert ! autovideosink`

# UDP Playback

## gstreamer-0.10

- UDP Theora
  - First start the client: `gst-launch-0.10 -v udpsrc port=5000 ! application/x-gdp ! gdpdepay ! video/x-theora ! theoradec ! video/x-raw-yuv,format=\(fourcc\)I420,width=352,height=288 ! ffmpegcolorspace ! capsfilter caps=video/x-raw-rgb,bpp=32,endianness=4321,red_mask=65280,green_mask=16711680,blue_mask=-16777216 ! autoconvert ! ximagesink`
  - Then start the server: `gst-launch-0.10 -v videotestsrc ! video/x-raw-yuv,format=\(fourcc\)I420,width=352,height=288 ! theoraenc ! video/x-theora ! gdppay ! application/x-gdp ! udpsink host=127.0.0.1 port=5000`
- RTP/UDP with testsource
  - Server: `gst-launch -v videotestsrc ! video/x-raw-rgb, framerate=25/1, width=640, height=480 ! rtpvrawpay ! udpsink host=localhost port=5000`
  - Client: `gst-launch-0.10 -v udpsrc port=5000 caps=" application/x-rtp, media=(string)video, clock-rate=(int)90000, encoding-name=(string)RAW, sampling=(string)RGB, depth=(string)8, width=(string)640, height=(string)480, colorimetry=(string)SMPTE240M, payload=(int)96, ssrc=(uint)3496538899, clock-base=(uint)2820015588, seqnum-base=(uint)5902" ! rtpvrawdepay ! autoconvert ! ximagesink`
- UDP with testsource
  - Server: `gst-launch -v videotestsrc ! video/x-raw-rgb, framerate=25/1, width=100, height=100 ! udpsink host=localhost port=5000`
  - Client: `gst-launch-0.10 -v udpsrc port=5000 caps=" application/x-rtp, media=(string)video, clock-rate=(int)90000, encoding-name=(string)RAW, sampling=(string)BGR, depth=(string)8, width=(string)640, height=(string)480, framerate=30/1" ! rtpvrawdepay ! autoconvert ! ximagesink`
  - Hint
    - in the UDP case (without RTP fragmentation) the packages can be only as big as one TCP frame

## gstreamer-1.0

- RTP/UDP with testsource
  - Client: `gst-launch-1.0 -v udpsrc port=5000 caps=" application/x-rtp, media=(string)video, clock-rate=(int)90000, encoding-name=(string)RAW, sampling=(string)RGB, depth=(string)8, width=(string)640, height=(string)480, colorimetry=(string)SMPTE240M, payload=(int)96, ssrc=(uint)3496538899, clock-base=(uint)2820015588, seqnum-base=(uint)5902" ! rtpvrawdepay ! videoconvert ! ximagesink`
  - Server: `gst-launch-1.0 -v videotestsrc ! video/x-raw, format=RGB, framerate=25/1, width=640, height=480 ! rtpvrawpay ! udpsink host=localhost port=5000`
- RTP/UDP with mat2gstreamer
  - Client: `gst-launch-1.0 -v udpsrc port=5000 caps=" application/x-rtp, media=(string)video, clock-rate=(int)90000, encoding-name=(string)RAW, sampling=(string)BGR, depth=(string)8, width=(string)640, height=(string)480, colorimetry=(string)SMPTE240M, payload=(int)96, ssrc=(uint)3496538899, clock-base=(uint)2820015588, seqnum-base=(uint)5902" ! rtpvrawdepay ! videoconvert ! ximagesink`
  - Server: `./mat2gstreamer`

# TCP Playback

## gstreamer-1.0

- RTP/TCP
  - Server: `gst-launch-1.0 -v videotestsrc ! video/x-raw, format=RGB, framerate=30/1, width=1000, height=1000 ! gdppay ! tcpserversink port=3000`
  - Client: `gst-launch-1.0 -v tcpclientsrc port=3000 ! gdpdepay ! videoconvert ! ximagesink`

# ARtoolkit

NOTE:
The `identity` plugin just pipes the source to the sink as it is.
But it has the feature, the any program (here it is ARToolkit) which calls this gstreamer pipeline, can grab the data via the given `name`

## testsource

### gstreamer-0.10

- `./simpleTest "videotestsrc ! video/x-raw-rgb,bpp=24,width=1000,height=1000 ! identity name=artoolkit sync=true ! fakesink"`

### gstreamer-1.0

- `./simpleTest "videotestsrc ! video/x-raw, format=RGB,width=640,height=480,framerate=30/1 ! identity name=artoolkit sync=true ! fakesink"`

## Video

### gstreamer-0.10

- h.264/MKV: `./simpleTest "filesrc location=video.mkv ! decodebin2 ! ffmpegcolorspace ! video/x-raw-rgb,bpp=24 ! identity name=artoolkit sync=true ! fakesink"`
- IYUV/AVI (Uncompressed I420): 

### gstreamer-1.0

- Does not work properly: `./simpleTest 'filesrc location=video.mkv ! matroskademux ! avdec_h264 ! identity name=artoolkit sync=true ! capabilities video/x-raw,width=1920,height=1080,format=RGB,bpp=24 ! fakesink'`

## Camera

### gstreamer-0.10

- `./simpleTest -device=GStreamer  "v4l2src ! ffmpegcolorspace ! video/x-raw-rgb,bpp=24 ! identity name=artoolkit sync=true ! fakesink"`

### gstreamer-1.0

- Does not work: `./simpleTest -device=GStreamer  "v4l2src device="/dev/video0" ! videoconvert ! video/x-raw,format=RGB,width=640,height=480,framerate=30/1 ! identity name=artoolkit sync=true ! fakesink"`

## UDP

### gstreamer-0.10

- testsource
  - Server: `gst-launch -v videotestsrc ! video/x-raw-rgb, framerate=25/1, width=100, height=100 ! udpsink host=localhost port=5000`
  - Client: `./simpleTest -device=GStreamer  'udpsrc port=5000 caps="video/x-raw-rgb, framerate=(fraction)25/1, width=(int)100, height=(int)100, bpp=(int)32, endianness=(int)4321, depth=(int)24, red_mask=(int)16711680, green_mask=(int)65280, blue_mask=(int)255" ! ffmpegcolorspace ! video/x-raw-rgb,bpp=24 ! identity name=artoolkit sync=true ! fakesink'`
- Camera
  - Server: `./mat2gstreamer`
  - Client: `./simpleTest -device=GStreamer  'udpsrc port=5000 caps="application/x-rtp, media=(string)video, clock-rate=(int)90000, encoding-name=(string)RAW, sampling=(string)BGR, depth=(string)8, width=(string)640, height=(string)480, framerate=30/1" ! rtpvrawdepay ! ffmpegcolorspace ! video/x-raw-rgb,bpp=24 ! identity name=artoolkit sync=true ! fakesink'`

Non-WORKING SETUP WITH UDP STREAM from OpenCV Webcam:
- `./simpleTest -device=GStreamer  'udpsrc port=5000 caps="video/x-raw-rgb, framerate=(fraction)30/1, width=(int)640, height=(int)480, bpp=(int)24, endianness=(int)4321, depth=(int)24, red_mask=(int)16711680, green_mask=(int)65280, blue_mask=(int)255" ! ffmpegcolorspace ! video/x-raw-rgb,bpp=24 ! identity name=artoolkit sync=true ! fakesink'`

###  gstreamer-1.0

- testsource
  - Server: `gst-launch-1.0 -v videotestsrc ! video/x-raw, format=BGR, framerate=25/1, width=100, height=100 ! rtpvrawpay ! udpsink host=localhost port=5000`
  - Client: `./simpleTest -device=GStreamer 'udpsrc port=5000 caps="application/x-rtp, media=(string)video, clock-rate=(int)90000, encoding-name=(string)RAW, sampling=(string)BGR, depth=(string)8, width=(string)100, height=(string)100, colorimetry=(string)SMPTE240M, payload=(int)96, ssrc=(uint)1744359127, timestamp-offset=(uint)1879147832, seqnum-offset=(uint)3344" ! rtpvrawdepay ! videoconvert video/x-raw, format=RGB ,bpp=24 ! identity name=artoolkit sync=true ! fakesink'`

## Shared Memory

### gstreamer-1.0

- Webcam
  - Server: `./mat2gstreamer`
  - Client (Should work, but has segmentation fault when reaching function `arVideoCapStart()`): `./localizationTwb2 --vconf 'shmsrc socket-path=/tmp/foo ! video/x-raw,width=640,height=480,framerate=30/1,format=RGB ! identity name=artoolkit sync=true ! fakesink' --matrixCodeType AR_MATRIX_CODE_3x3 --patternDetectionMode AR_MATRIX_CODE_DETECTION --cpara calibration/cameraTimoNotebook_para.dat`
  - Client (Works, but only with critical errors: [`GStreamer-CRITICAL **: gst_mini_object_unref: assertion 'mini_object->refcount > 0' faile`]): `/localizationTwb2 --vconf 'shmsrc socket-path=/tmp/foo ! video/x-raw,width=640,height=480,framerate=30/1,format=RGB ! videoconvert ! video/x-raw,format=I420 ! videoconvert ! video/x-raw,format=BGR ! identity name=artoolkit sync=true ! fakesink' --matrixCodeType AR_MATRIX_CODE_3x3 --patternDetectionMode AR_MATRIX_CODE_DETECTION --cpara calibration/cameraTimoNotebook_para.dat`

# TWB

# gstreamer-1.0

- RTP/UDP: Show the TWB camera attached to the local PC
  - Server: ./localizationTwb with pipeline: `appsrc ! rtpvrawpay mtu=65000 ! udpsink host=localhost port=5000 max-bitrate=10000000 sync=true`
  - Client: `gst-launch-1.0 -v udpsrc port=5000 caps=" application/x-rtp, media=(string)video, clock-rate=(int)90000, encoding-name=(string)RAW, sampling=(string)RGB, depth=(string)8, width=(string)1000, height=(string)1000, colorimetry=(string)SMPTE240M, payload=(int)96, ssrc=(uint)3496538899, clock-base=(uint)2820015588, seqnum-base=(uint)5902" ! rtpvrawdepay ! videoconvert ! ximagesink`
- RTP/TCP
  - Server: ./localizationTwb with pipeline: `appsrc! gdppay ! tcpserversink port=3000`
  - Client : `./localizationTwb2 --vconf 'tcpclientsrc port=3000 ! gdpdepay ! videoconvert video/x-raw, format=RGB ,bpp=24 ! identity name=artoolkit sync=true ! fakesink' --matrixCodeType AR_MATRIX_CODE_3x3 --patternDetectionMode AR_MATRIX_CODE_DETECTION --cpara calibration/camera3twb_para.dat`

# RTSP

In contradiction to RTP, a RTSP server negotiates the connection between a RTP-server and a client on demand.
Thus, the target address of the RTP stream does not to be known in advance.
The `gst-rtsp-server` is not a gstreamer plugin, but a library which can be used to implement your own RTSP application.
The following test case was applied on a Ubuntu 12.04.5 machine:

- Preliminars
  - Install gstreamer-1.0 with base/good/ugly/bad plugins
  - Install `autoconf automake autopoint libtool`
- Build `gst-rtsp-server`
  - `git clone git://anongit.freedesktop.org/gstreamer/gst-rtsp-server && cd gst-rtsp-server`
  - We use gstreamer 1.2: `git checkout remotes/origin/1.2`
  - Build: `./autogen.sh --noconfigure && GST_PLUGINS_GOOD_DIR=$(pkg-config --variable=pluginsdir gstreamer-plugins-bad-1.0) ./configure ./configure && make` (For some reason, `GST_PLUGINS_GOOD_DIR` is not set by `pkg-config`, so we set it explicitly)
- Test run
  - Run test application: `cd examples && ./test-launch "( videotestsrc ! x264enc ! rtph264pay name=pay0 pt=96 )"`
  - One can now access the stream (e.g. using VLC) remotely by the address: `rtsp://HOST_IP:8554/test`
  
## TWB

- Run RTSP application to stream TWB camera (unfunctional try): `./test-launch "( shmsrc socket-path=/tmp/cam1 is-live=true do-timestamp=true ! video/x-raw, format=BGR ,width=1000,height=1000,framerate=48/1, bpp=24 ! videoconvert format="I420" ! x264enc ! rtph264pay name=pay0 pt=96 )"`

# Play Images TBD

    FOLDER=/media/filestorage/pjuenemann/floor/BMP/;idx=0;for FILE in ${FOLDER}*bmp; do ln -s ${FILE} $(printf '%010d.bmp\n' $idx);idx=$((idx+1)); done
    FOLDER=/media/filestorage/pjuenemann/floor/PNG/;idx=0;for FILE in ${FOLDER}*png; do ln -s ${FILE} $(printf '%010d.png\n' $idx);idx=$((idx+1)); done
    gst-launch multifilesrc location=%010d.bmp \ caps="image/bmp,framerate=30/1,pixel-aspect-ratio=1/1" ! ffmpegcolorspace ! video/x-raw-yuv,format=(fourcc)I420 ! imagesink
    gst-launch multifilesrc location=%010d.bmp caps="image/bmp,framerate=30/1,pixel-aspect-ratio=1/1" ! ffmpegcolorspace ! video/x-raw-yuv,format=(fourcc)I420 ! imagesink
    gst-launch multifilesrc location=%010d.bmp caps="image/bmp,framerate=30/1,pixel-aspect-ratio=1/1" ! ffmpegcolorspace ! video/x-raw-yuv,format=I420 ! imagesink
    gst-launch-1.0 multifilesrc location=%010d.bmp caps="image/bmp,framerate=30/1,pixel-aspect-ratio=1/1" ! ffmpegcolorspace ! video/x-raw-yuv,format=I420 ! imagesink
    gst-launch-1.0 multifilesrc location=%010d.bmp caps="image/bmp,framerate=30/1,pixel-aspect-ratio=1/1" ! video/x-raw-yuv,format=I420 ! imagesink
    gst-launch-1.0 multifilesrc location=%010d.bmp caps="image/bmp,framerate=30/1,pixel-aspect-ratio=1/1" ! video/x-raw-yuv,format=I420 ! videosink
    gst-launch-1.0 multifilesrc location=%010d.bmp caps="image/bmp,framerate=30/1,pixel-aspect-ratio=1/1" ! video/x-raw-yuv,format=I420 ! autovideosink
    gst-launch-1.0 multifilesrc location=%010d.bmp ! caps="image/bmp,framerate=30/1,pixel-aspect-ratio=1/1" ! video/x-raw-yuv,format=I420 ! autovideosink
    gst-launch-1.0 multifilesrc location=%010d.bmp ! image/bmp,framerate=30/1,pixel-aspect-ratio=1/1 ! video/x-raw-yuv,format=I420 ! autovideosink
    gst-launch-1.0 multifilesrc location=%010d.bmp ! image/bmp,framerate=30/1,pixel-aspect-ratio=1/1 ! autoconvert ! video/x-raw-yuv,format=I420 ! autovideosink
    gst-launch-1.0 multifilesrc location=%010d.bmp ! image/bmp,framerate=30/1,pixel-aspect-ratio=1/1 ! autoconvert ! video/x-raw,format=I420 ! autovideosink
    gst-launch-1.0 multifilesrc location=%010d.bmp ! image/bmp,framerate=30/1,pixel-aspect-ratio=1/1 ! videoconvert ! video/x-raw,format=I420 ! autovideosink
    gst-launch-1.0 multifilesrc location=%10d.bmp ! image/bmp,framerate=30/1,pixel-aspect-ratio=1/1 ! autoconvert ! video/x-raw,format=I420 ! autovideosink
    gst-launch-1.0 multifilesrc location=%010d.bmp ! image/bmp,framerate=30/1,pixel-aspect-ratio=1/1 ! autoconvert ! video/x-raw,format=I420 ! autovideosink
    gst-launch-1.0 multifilesrc location=%010d.bmp index=0 caps="image/bmp,framerate=\(fraction\)30/1,pixel-aspect-ratio=1/1" ! autoconvert ! video/x-raw,format=I420 ! autovideosink
    gst-launch-1.0 multifilesrc location=%010d.bmp index=0 caps="image/bmp,framerate=\(fraction\)30/1,pixel-aspect-ratio=1/1" ! videoconvert ! video/x-raw,format=I420 ! autovideosink
    gst-launch-1.0 multifilesrc location=%010d.bmp index=0 caps="image/bmp,framerate=\(fraction\)30/1,pixel-aspect-ratio=1/1" ! autoconvert ! video/x-raw,format=I420 ! videorate ! autovideosink
    gst-launch-1.0 multifilesrc location="%010d.bmp" index=0 ! autovideosink
    gst-launch-1.0 multifilesrc location="%09d.bmp" index=0 ! autovideosink
    gst-launch-1.0 multifilesrc location="img.%04d.png" index=0 caps="image/png,framerate=\(fraction\)12/1" !     pngdec ! videoconvert ! videorate ! theoraenc ! oggmux ! \
    gst-launch-1.0 multifilesrc location="img.%010d.png" index=0 caps="image/png,framerate=\(fraction\)12/1" ! pngdec ! videoconvert ! videorate ! autovideosink
    gst-launch-1.0 multifilesrc location="%010d.png" index=0 caps="image/png,framerate=\(fraction\)12/1" ! pngdec ! videoconvert ! videorate ! autovideosink
    gst-launch-1.0 multifilesrc location="%010d.png" index=0 caps="image/png,framerate=\(fraction\)12/1" ! pngdec ! videoconvert ! videorate ! autosink
    gst-launch-1.0 multifilesrc location="%010d.png" index=0 caps="image/png,framerate=\(fraction\)12/1" ! pngdec ! videoconvert  ! autovideosink
    gst-launch-1.0 multifilesrc location="%010d.png" index=0 caps="image/png,framerate=\(fraction\)12/1" ! pngdec ! videoconvert  ! ximagesink
    gst-launch-1.0 multifilesrc location="%010d.png" index=0 caps="image/png,framerate=\(fraction\)12/1" ! pngdec ! videoconvert ! videorate  ! ximagesink
    gst-launch-1.0 multifilesrc location="%010d.png" index=0 caps="image/png,framerate=\(fraction\)12/1" ! pngdec ! videoconvert ! videorate  ! ximagesink
    gst-launch-1.0 multifilesrc location="%010d.png" index=1 caps="image/png,framerate=\(fraction\)12/1" ! pngdec ! videoconvert ! videorate  ! ximagesink
    gst-launch-1.0 multifilesrc location="%010d.png" index=1 caps="image/png,framerate=\(fraction\)30/1" ! pngdec ! videoconvert ! videorate  ! ximagesink
    gst-launch-1.0 multifilesrc location="%010d.png" index=1 caps="image/png,framerate=\(fraction\)30/1" ! pngdec ! videoconvert ! videorate ! theoraenc ! oggmux ! filesink location="images.ogg"
    gst-launch-1.0 multifilesrc location="%010d.png" index=1 caps="image/png,framerate=\(fraction\)30/1" ! pngdec ! videoconvert ! videorate ! theoraenc ! oggmux ! autovideosink
    gst-launch-1.0 multifilesrc location="%010d.png" index=1 caps="image/png,framerate=\(fraction\)30/1" ! pngdec ! videoconvert ! videorate ! theoraenc ! oggmux ! xvimagesink
    gst-launch-1.0 multifilesrc location="%010d.png" index=1 caps="image/png,framerate=\(fraction\)30/1" ! pngdec ! videoconvert ! videorate ! xvimagesink
    gst-launch-1.0 multifilesrc location="%010d.png" index=1 caps="image/png,framerate=\(fraction\)30/1" ! pngdec ! videoconvert ! videorate ! avdec_png ! autovideosink
    gst-launch-1.0 multifilesrc location="%010d.png" index=1 caps="image/png,framerate=\(fraction\)30/1" !  avdec_png ! autovideosink
    gst-launch multifilesrc location="%010d.png" ! image/png,framerate=30/1 ! pngdec ! videorate ! video/x-raw-rgb,framerate=30/1 ! ffmpegcolorspace ! xvimagesink
    gst-launch-1.0 multifilesrc location="%010d.png" ! image/png,framerate=30/1 ! pngdec ! videorate ! video/x-raw-rgb,framerate=30/1 ! videoconvert ! xvimagesink
    gst-launch-1.0 multifilesrc location="%010d.png" ! image/png,framerate=30/1 ! pngdec ! videorate ! video/x-raw-rgb,framerate=30/1 !  xvimagesink
    gst-launch-1.0 multifilesrc location="%010d.png" ! image/png,framerate=30/1 ! pngdec ! videorate ! video/x-raw-rgb,framerate=30/1 !  ffmpegcolorspace ! xvimagesink
    gst-launch-1.0 multifilesrc location="%010d.png" ! image/png,framerate=30/1 ! pngdec ! videorate ! video/x-raw-rgb,framerate=30/1 !  videoconvert  ! xvimagesink
    gst-launch-1.0 multifilesrc location="%010d.png" ! image/png,framerate=30/1 ! pngdec ! videorate ! video/x-raw,format=rgb,framerate=30/1 !  videoconvert  ! xvimagesink
    gst-launch-1.0 multifilesrc location="%010d.png" ! image/png,framerate=30/1 ! pngdec ! videorate ! video/x-raw,format=RGB,framerate=30/1 !  videoconvert  ! xvimagesink
    gst-launch-1.0 multifilesrc location="%010d.png" ! image/png,framerate=30/1 ! pngdec ! videorate ! video/x-raw,format=RGB,framerate=30/1 !  videoconvert  ! 
    gst-launch-1.0 multifilesrc location="%010d.png" ! image/png,framerate=30/1 ! pngdec ! videorate ! video/x-raw,format=RGB,framerate=30/1 !  videoconvert  ! xvimagesink
    gst-launch-1.0 multifilesrc location="%010d.png" ! image/png,framerate=30/1 ! pngdec ! videorate ! video/x-raw,format=I420,framerate=30/1 !  videoconvert  ! xvimagesink
    gst-launch-1.0 multifilesrc location="%010d.png" ! image/png,framerate=30/1 ! pngdec ! videorate ! video/x-raw ,format=I420 ,framerate=30/1 !  videoconvert  ! xvimagesink
    gst-launch-1.0 multifilesrc location="%010d.png" ! image/png,framerate=30/1 ! pngdec ! videorate ! video/x-raw ,format=RGB ,framerate=30/1 !  videoconvert  ! xvimagesink
    gst-launch-1.0 multifilesrc location="%010d.png" ! image/png,framerate=30/1 ! pngdec ! videorate ! video/x-raw ,format=BGR ,framerate=30/1 !  videoconvert  ! xvimagesink
    gst-launch-0.10 videotestsrc ! ximagesink
    gst-launch-1.0 multifilesrc location="%010d.png" ! image/png,framerate=30/1 ! pngdec ! videorate ! video/x-raw ,format=RGB ,framerate=30/1 !  videoconvert  ! ximagesink
    gst-launch-1.0 multifilesrc location="%010d.png" ! image/png,framerate=30/1 ! pngdec ! videorate ! video/x-raw ,format=RGB ,framerate=30/1 !  videoconvert  ! autovideosink
    gst-launch-1.0 multifilesrc location="%010d.png" ! image/png,framerate=30/1 ! pngdec ! videorate ! video/x-raw ,format=I420 ,framerate=30/1 !  videoconvert  ! autovideosink
    gst-launch-1.0 multifilesrc location="%010d.png" ! image/png,framerate=30/1 ! pngdec ! videorate ! video/x-raw ,format="I420" ,framerate=30/1 !  videoconvert  ! autovideosink
    gst-launch-1.0 multifilesrc location="%010d.png" ! image/png,framerate=30/1 ! pngdec ! videorate ! video/x-raw ,format="BGR" ,framerate=30/1 !  videoconvert  ! autovideosink
    gst-launch-1.0 multifilesrc location="%010d.png" ! image/png,framerate=30/1 ! pngdec ! videorate ! video/x-raw ,format="RGB" ,framerate=30/1 !  videoconvert  ! autovideosink
    gst-launch-1.0 multifilesrc location="%010d.png" ! image/png,framerate=30/1 ! avdec_png ! videorate ! video/x-raw ,format="RGB" ,framerate=30/1 !  videoconvert  ! autovideosink
    gst-launch-1.0 multifilesrc location="%010d.png" ! image/png,framerate=30/1 ! avdec_png ! videorate ! video/x-raw ,format="YUV" ,framerate=30/1 !  videoconvert  ! autovideosink
    gst-launch-1.0 multifilesrc location="%010d.png" ! image/png,framerate=30/1 ! avdec_png ! videorate ! video/x-raw ,format="RGB" ,framerate=30/1 !  videoconvert  ! autovideosink
    gst-launch-1.0 multifilesrc location="%010d.png" ! image/png,framerate=30/1 ! avdec_png ! videorate ! video/x-raw ,format="RGB" ,framerate=30/1 !  videoconvert  ! fpsdisplaysink
    gst-launch-1.0 multifilesrc location="%010d.png" ! image/png,framerate=30/1 ! avdec_png ! videorate ! video/x-raw ,format="RGB" ,framerate=30/1 !  fpsdisplaysink
    gst-launch-1.0 multifilesrc location="%010d.png" ! image/png,framerate=30/1 ! pngdec ! videorate ! video/x-raw ,format="RGB" ,framerate=30/1 !  fpsdisplaysink
    gst-launch-1.0 multifilesrc location="%010d.png" ! image/png,framerate=30/1 ! avdec_png ! videorate ! video/x-raw ,format="RGB" ,framerate=30/1 !  videoconvert  ! autovideosink
    gst-launch-1.0 videotestsrc ! video/x-raw, format="I420" ,width=640,height=480,framerate=30/1 ! identity name=artoolkit sync=true ! autovideosink
    gst-launch-1.0 videotestsrc ! video/x-raw, format="I420" ,width=640,height=480,framerate=30/1  ! autovideosink
