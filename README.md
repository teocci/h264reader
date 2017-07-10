##  h264reader

This is a very simple and straight-forward a C implementation for reading and writing H.264 video

### Disclaimer

This repository contains a simple C implementation for reading and writing H.264 video, which is still a work in progress. It is not intended to be used as-is in applications as a library dependency, and will not be maintained as such. Bug fix contributions are welcome, but issues and feature requests will not be addressed.

### Specifications

#### Design Goals

The main design goal is provide a simple open-source implementation for reading and writing H264 streams.

Reading and writing headers (sequence and picture parameter sets, slice headers, etc) is a higher priority goal than reading and writing encoded picture data, and has been implemented first.

Secondary goals are portability and a simple, clean API.  This repository uses a subset of the language features of the C99 standard, without any compiler-specific extensions or dependence on a particular operating system or hardware.

**This implementation is NOT an encoder or decoder**, although you could use it to make writing an encoder or decoder much easier. It understands the *syntax* of H264, but not the semantics (meaning) of the data it reads or writes.

#### Code sample 

Read one data unit (NAL, or network abstraction layer unit) out of an H264 bitstream and print it out:

```c
int nal_start, nal_end;
uint8_t* buf;
int len;
// read some H264 data into buf
h264_stream_t* h = h264_new();
find_nal_unit(buf, len, &nal_start, &nal_end);
read_nal_unit(h, &buf[nal_start], nal_end - nal_start);
debug_nal(h);
```

#### Limitations

Everything in the H264 standard is implemented except for: 

- Parsing of different SEI messages
- Decoded reference picture marking
- Reference picture list reordering
- SPS extension
- Slice data
- Slice data partitioning

Most of the unimplemented data will be correctly skipped when reading and ignored (not written) while writing; the code to read/write it is present as a stub, but they require somewhat more complex data structures to store the data, and those are not implemented yet.

Reading and writing slice data and slice data partitioning are complex and will not be implemented in the foreseeable future. It would require extensive changes to the current implementation, and as it is only available in Extended Profile, its utility is limited.

*h264reader* does not try to maintain a table of different SPS and PPS, nor does it check that pic_parameter_set_id and seq_parameter_set_id in the current nal match those of the last read SPS/PPS. It stores the most recently seen SPS and PPS, and assumes those are applicable. If you are likely to encounter streams with multiple PPS or SPS, as a workaround you can store copies of each SPS/PPS which is read, check the corresponding id in other elements after reading, and if they do not match substitute the correct parameter set and re-read.  This limitation will be removed in a later version.

#### Known Bugs

- if attempting to write SLICE_TYPE_*_ALT slices, resulting slice header could be incorrect in versions <= 0.1.6
- if no nal was found in entire processing buffer, h264_analyze could exit prematurely in versions <= 0.1.6
- writing complete nals did not work correctly in versions <= 0.1.5
- rbsp_trailing_bits could be (very rarely) handled incorrectly in versions <= 0.1.5
- attempting to write decoded reference picture marking or reference picture list reordering data results in unpredictable behaviour

### Contributing
If you would like to contribute code, you can do so through GitHub by forking the repository and sending a pull request.
When submitting code, please make every effort to follow existing conventions and style in order to keep the code as readable as possible.

## Credits

Based on the h264bitstream written by Alex Izvorski <aizvorski@gmail.com> and Alex Giladi <alex.giladi@gmail.com>

* [h264bitstream Sourceforge Project Page][1]

## License

The code supplied here is covered under the MIT Open Source License..


[1]: http://sourceforge.net/projects/h264bitstream


