## Documentation

The library follows the H264 standard verbatim in most cases, including the naming of most variables and some functions.

A copy of the standard can be obtained from ITU (at http://www.itu.int/rec/T-REC-H.264/e ).

The public API consists of:

    h264_new
    h264_free
    find_nal_unit
    read_nal_unit
    write_nal_unit
    rbsp_to_nal
    nal_to_rbsp
    debug_nal

plus direct access to the fields of h264_stream_t and the data structures nested in that.

Using other functions contained in the library to directly read or write specific types of NALs or parts thereof is not part of the public API, although it is not hard to do if you prepare the required bs_t argument.  Please also note that using *_rbsp functions requires you also to perform handle RBSP to NAL (and vice versa) translation by calling rbsp_to_nal and nal_to_rbsp.


## Programming Notes

All data types are stored in simple int fields in various structures, sometimes nested structures (which are guaranteed to be non-null if the containing structure is non-null, and may therefore be safely dereferenced).  These fields can be used and assigned to directly; due to the very large number of fields, there are no accessor functions.  Boolean values are 0 for false and 1 for true.  Unsigned and signed integers can be assigned and read directly.

The currently active picture parameter set, sequence parameter set, slice header and nal are stored as fields in the h264_stream_t structure h which represents the stream being read.

For example, to write a simple SPS, use code like this:

    h264_stream_t* h = h264_new();

    h->nal->nal_ref_idc = 0x03;
    h->nal->nal_unit_type = NAL_UNIT_TYPE_PPS;

    h->sps->profile_idc = 0x42;
    h->sps->level_idc = 0x33;
    h->sps->log2_max_frame_num_minus4 = 0x05;
    h->sps->log2_max_pic_order_cnt_lsb_minus4 = 0x06;
    h->sps->num_ref_frames = 0x01;
    h->sps->pic_width_in_mbs_minus1 = 0x2c;
    h->sps->pic_height_in_map_units_minus1 = 0x1d;
    h->sps->frame_mbs_only_flag = 0x01;
    
    int len = write_nal_unit(h, buf, size);

All operations rely on an underlying set of bitstream functions which operate on bs_t* structures. Those are inherently buffer-overflow-safe and endiannes-independent, but provide only limited error handling at this time.  Reads beyond the end of a buffer succeed and return an infinite sequence of zero bits; writes beyond the end of a buffer succeed and are ignored.  To be sure that the buffer passed was large enough, check that the return of the read_nal_unit or write_nal_unit is *less* than the size of the buffer you passed in; if it is equal it is possible you're missing the end of the data.

You should always call find_nal_unit before calling read_nal_unit as shown in the quick start example. Successive reads without a find may fail, either due to bugs in the handling of the various types of rbsp padding, or because the stream is not compliant.