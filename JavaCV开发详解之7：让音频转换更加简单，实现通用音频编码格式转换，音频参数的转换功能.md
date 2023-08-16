**javaCV开发详解之7：让音频转换更加简单，实现通用音频编码格式转换，音频参数的转换功能**

前言
之前几章把javaCV-FFmpeg中的能够实现的基本功能大致梳理了一遍，本章在之前几章基础上实现一个通用的音频编码和参数转换器

音频重采样参考请单独查看此章：javaCV开发详解之14：音频重采样

实现功能：
①音频编码转换

②音频格式转换

④等等。。。更多功能自行探索

代码实现：

```
package cn.eguid.audioConvert;

import org.bytedeco.javacpp.avcodec;
import org.bytedeco.javacv.FFmpegFrameGrabber;
import org.bytedeco.javacv.FFmpegFrameRecorder;
import org.bytedeco.javacv.Frame;
import org.bytedeco.javacv.FrameGrabber;
import org.bytedeco.javacv.FrameRecorder;
import org.bytedeco.javacv.FrameRecorder.Exception;

/**
 * 音频参数转换（包含采样率、编码，位数，通道数）
 * 
 * @author eguid
 * 
 */
public class AudioConvert {
  /**
   * 通用音频格式参数转换
   * 
   * @param inputFile
   *            -导入音频文件
   * @param outputFile
   *            -导出音频文件
   * @param audioCodec
   *            -音频编码
   * @param sampleRate
   *            -音频采样率
   * @param audioBitrate
   *            -音频比特率
   */
  public static void convert(String inputFile, String outputFile, int audioCodec, int sampleRate, int audioBitrate,
      int audioChannels) {
    Frame audioSamples = null;
    // 音频录制（输出地址，音频通道）
    FFmpegFrameRecorder recorder = null;
    //抓取器
    FFmpegFrameGrabber grabber = new FFmpegFrameGrabber(inputFile);

    // 开启抓取器
    if (start(grabber)) {
      recorder = new FFmpegFrameRecorder(outputFile, audioChannels);
      recorder.setAudioOption("crf", "0");
      recorder.setAudioCodec(audioCodec);
      recorder.setAudioBitrate(audioBitrate);
      recorder.setAudioChannels(audioChannels);
      recorder.setSampleRate(sampleRate);
      recorder.setAudioQuality(0);
      recorder.setAudioOption("aq", "10");
      // 开启录制器
      if (start(recorder)) {
        try {
          // 抓取音频
          while ((audioSamples = grabber.grab()) != null) {
            recorder.setTimestamp(grabber.getTimestamp());
            recorder.record(audioSamples);
          }

        } catch (org.bytedeco.javacv.FrameGrabber.Exception e1) {
          System.err.println("抓取失败");
        } catch (Exception e) {
          System.err.println("录制失败");
        }
        stop(grabber);
        stop(recorder);
      }
    }

  }

  public static boolean start(FrameGrabber grabber) {
    try {
      grabber.start();
      return true;
    } catch (org.bytedeco.javacv.FrameGrabber.Exception e2) {
      try {
        System.err.println("首次打开抓取器失败，准备重启抓取器...");
        grabber.restart();
        return true;
      } catch (org.bytedeco.javacv.FrameGrabber.Exception e) {
        try {
          System.err.println("重启抓取器失败，正在关闭抓取器...");
          grabber.stop();
        } catch (org.bytedeco.javacv.FrameGrabber.Exception e1) {
          System.err.println("停止抓取器失败！");
        }
      }

    }
    return false;
  }

  public static boolean start(FrameRecorder recorder) {
    try {
      recorder.start();
      return true;
    } catch (Exception e2) {
      try {
        System.err.println("首次打开录制器失败！准备重启录制器...");
        recorder.stop();
        recorder.start();
        return true;
      } catch (Exception e) {
        try {
          System.err.println("重启录制器失败！正在停止录制器...");
          recorder.stop();
        } catch (Exception e1) {
          System.err.println("关闭录制器失败！");
        }
      }
    }
    return false;
  }

  public static boolean stop(FrameGrabber grabber) {
    try {
      grabber.flush();
      grabber.stop();
      return true;
    } catch (org.bytedeco.javacv.FrameGrabber.Exception e) {
      return false;
    } finally {
      try {
        grabber.stop();
      } catch (org.bytedeco.javacv.FrameGrabber.Exception e) {
        System.err.println("关闭抓取器失败");
      }
    }
  }

  public static boolean stop(FrameRecorder recorder) {
    try {
      recorder.stop();
      recorder.release();
      return true;
    } catch (Exception e) {
      return false;
    } finally {
      try {
        recorder.stop();
      } catch (Exception e) {

      }
    }
  }
}
```

用心你会发现这章跟前面几章很相似？



测试效果
以wav转mp3为例

```
// 测试
  public static void main(String[] args) {
    //pcm参数转换
//    convert("东部信息.wav", "eguid.wav", avcodec.AV_CODEC_ID_PCM_S16LE, 8000, 16000,1);
    //pcm转mp3编码示例
    convert("东部信息.wav", "eguid.mp3", avcodec.AV_CODEC_ID_MP3, 8000, 16,1);
  }
```



©著作权归作者所有：来自51CTO博客作者eguid的原创作品，请联系作者获取转载授权，否则将追究法律责任
javaCV开发详解之7：让音频转换更加简单，实现通用音频编码格式转换，音频参数的转换功能（以pcm16le编码的wav转mp3为例）JavaCV实战专栏文章目录（JavaCV速查手册）
https://blog.51cto.com/eguid/5062322