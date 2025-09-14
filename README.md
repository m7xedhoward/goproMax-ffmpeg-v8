# GoPro Max FFmpeg Processing


## Acknowledgments

This project builds upon the GoPro Max filter implementation by gmat. 

## Building

To build FFmpeg with GoPro Max 360° video processing capabilities, ensure you have the necessary dependencies installed, including OpenCL support. Clone the FFmpeg repository and configure it with the required flags:

```bash
./configure --disable-static --enable-shared --disable-x86asm --enable-opencl --enable-opengl --enable-sdl2 --enable-libx264 --enable-gpl
```

Then compile and install:

```bash
make -j$(nproc)
sudo make install
```

Optionally run from source directory:

```bash
LD_LIBRARY_PATH=./libavutil:./libavcodec:./libavformat:./libavfilter:./libswscale:./libswresample:./libavdevice ./ffmpeg ...
```

## Example Usage

### Basic GoPro Max 360° Video Processing

```bash
ffmpeg -y \
  -init_hw_device opencl \
  -hwaccel opencl \
  -filter_complex '[0:0]format=yuv420p,hwupload[a],[0:5]format=yuv420p,hwupload[b],[a][b]gopromax_opencl,hwdownload,format=yuv420p' \
  -i input.360 \
  -c:v libx264 \
  -pix_fmt yuv420p \
  -map_metadata 0 \
  -map 0:1 \
  -map 0:3 \
  -t 30 \
  output.mp4
```

### Command Breakdown

| Parameter | Purpose |
|-----------|---------|
| `-y` | Overwrite output file without asking |
| `-init_hw_device opencl` | Initialize OpenCL hardware acceleration |
| `-hwaccel opencl` | Use OpenCL hardware acceleration |
| `-filter_complex` | Apply complex filter chain (see below) |
| `-i input.360` | Input GoPro Max 360° file |
| `-c:v libx264` | Use H.264 video codec |
| `-pix_fmt yuv420p` | Set pixel format to YUV 4:2:0 |
| `-map_metadata 0` | Copy metadata from input |
| `-map 0:1` | Map audio stream |
| `-map 0:3` | Map additional stream e.g. GPS data|
| `-t 30` | Limit output to 30 seconds |
| `output.mp4` | Output file name |

### Filter Chain Explanation

The complex filter performs these steps:

1. **Format Conversion**: `[0:0]format=yuv420p` - Convert first video stream to YUV420p
2. **Hardware Upload**: `hwupload[a]` - Upload to GPU memory as stream 'a'
3. **Second Stream**: `[0:5]format=yuv420p,hwupload[b]` - Process stream 5 as stream 'b'
4. **GoPro Processing**: `[a][b]gopromax_opencl` - Apply custom GoPro Max filter
5. **Hardware Download**: `hwdownload,format=yuv420p` - Download result back to CPU

## Requirements

- FFmpeg with OpenCL support
- OpenCL-capable GPU for hardware acceleration



FFmpeg README
=============

FFmpeg is a collection of libraries and tools to process multimedia content
such as audio, video, subtitles and related metadata.

## Libraries

* `libavcodec` provides implementation of a wider range of codecs.
* `libavformat` implements streaming protocols, container formats and basic I/O access.
* `libavutil` includes hashers, decompressors and miscellaneous utility functions.
* `libavfilter` provides means to alter decoded audio and video through a directed graph of connected filters.
* `libavdevice` provides an abstraction to access capture and playback devices.
* `libswresample` implements audio mixing and resampling routines.
* `libswscale` implements color conversion and scaling routines.

## Tools

* [ffmpeg](https://ffmpeg.org/ffmpeg.html) is a command line toolbox to
  manipulate, convert and stream multimedia content.
* [ffplay](https://ffmpeg.org/ffplay.html) is a minimalistic multimedia player.
* [ffprobe](https://ffmpeg.org/ffprobe.html) is a simple analysis tool to inspect
  multimedia content.
* Additional small tools such as `aviocat`, `ismindex` and `qt-faststart`.

## Documentation

The offline documentation is available in the **doc/** directory.

The online documentation is available in the main [website](https://ffmpeg.org)
and in the [wiki](https://trac.ffmpeg.org).

### Examples

Coding examples are available in the **doc/examples** directory.

## License

FFmpeg codebase is mainly LGPL-licensed with optional components licensed under
GPL. Please refer to the LICENSE file for detailed information.

## Contributing

Patches should be submitted to the ffmpeg-devel mailing list using
`git format-patch` or `git send-email`. Github pull requests should be
avoided because they are not part of our review process and will be ignored.
