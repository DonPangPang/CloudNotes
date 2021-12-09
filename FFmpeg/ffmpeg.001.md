# PCM to AAC

网上很多例子都没有提到重采样这件事情，可能是FFmpeg的版本问题, `ffmpeg 4.x` 的`AAC`编码器仅支持`FLTP`，但是对于一些`PCM`数据而言，可能内容是`S16`，并且，编码前、编码后的这些参数可能都需要进行变更。

其它还需验证的内容, 例如: 采样率(Sample Rate), 位深(bit Rate), 声道类型(channel Layout)等是否需要进行转置, 以及如何猜测.

## 1 获取Pcm音频声道数

这个音频声道数分为很多种, 不仅仅平时我们理解的单声道、多声道、立体声等，通过阅读FFmpeg，其中大概涵盖了十多种。

## 2 对PCM进行重采样

重采样的时候应该需要先对`PCM`进行解码，然后调用`swr_context`来对`PCM`进行转码，主要是由于ffmpeg版本的原因，需要将`PCM`从`S16`转为其它格式。

## 3 编码

对`PCM`进行重采样后，会得到一笔数据，该数据与`channels`相关联，一般有几个声道，就需要填充到`AvFrame->data[n]`中，然后再进行初始化对应的编码器，将AvFrame传入其中。

## 4 获取编码后的音频帧

由于不同的编码算法，在编码原始输入数据后，数据长度会发生变化，ffmpeg中提供了一个缓冲区来做这件事情，需要专门有一个线程去调用`av_send_frame()`向缓冲区中填充数据，然后一个单独的线程来调用`av_receive_packet()`来获取数据，其中`Packet->data`中就是编码完成后的数据。当然，这个数据是裸数据，并没有header之类的信息，需要另外封装。