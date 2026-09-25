```python
sensor.reset()##在这一行之后直接添加
print(dir(sensor))   # ← 加这一行
```
然后就可以在终端显示出来
```python
MPY: soft reboot CanMV 0295137e6(based on Micropython e00a144) on 2025-09-20; k230d_canmv_atk_dnk230d with K230D find sensor gc2093_csi2, type 10, output 1280x960@90 ['__class__', '__init__', '__module__', '__qualname__', '__str__', 'stop', '__del__', '__dict__', 'GRAYSCALE', 'RGB565', 'RGB888', 'RGBP888', 'deinit', 'fd', 'flush', 'get_id', 'get_type', 'height', 'ioctl', 'reset', 'run', 'set_brightness', 'sleep', 'width', 'wrap', 'dumped_image', '_devs', '_is_mcm_device', '_handle_mcm_device', '_set_inbufs', '_run_mcm_device', '_dev_attr', '_is_started', '_chn_attr', '_get_dev_id', '_csis', '_dft_input_buff_num', '_dft_output_buff_num', '_csi_bus', '_type', '_dev_id', '_buf_init', '_buf_in_init', 'sensor_name', '_imgs', '_is_rgb565', '_is_grayscale', '_framesize', '_pixel_format', 'shutdown', '_release_image', '_release_all_chn_image', '_dumped_image', 'snapshot', 'skip_frames', 'get_fb', 'alloc_extra_fb', 'dealloc_extra_fb', 'set_pixformat', 'get_pixformat', '_parse_framesize', 'set_framesize', 'get_framesize', 'set_framerate', 'get_framerate', 'set_windowing', 'get_windowing', 'set_contrast', 'set_saturation', 'set_quality', 'set_colorbar', 'set_auto_gain', 'get_gain_db', 'set_auto_exposure', 'get_exposure_us', 'set_auto_whitebal', 'get_rgb_gain_db', 'set_auto_blc', 'get_blc_regs', 'set_hmirror', 'get_hmirror', 'set_vflip', 'get_vflip', 'set_transpose', 'get_transpose', 'set_auto_rotation', 'get_auto_rotation', 'set_framebuffers', 'get_framebuffers', 'disable_delays', 'disable_full_flush', 'set_lens_correction', 'set_vsync_callback', 'set_frame_callback', 'set_color_palette', 'get_color_palette', '__write_reg', '__read_reg', 'again', '_set_chn_fps', 'bind_info', 'YUV420SP', 'QQCIF', 'QCIF', 'CIF', 'QSIF', 'SIF', 'QQVGA', 'QVGA', 'VGA', 'HQQVGA', 'HQVGA', 'HVGA', 'B64X64', 'B128X64', 'B128X128', 'B160X160', 'B320X320', 'QQVGA2', 'WVGA', 'WVGA2', 'SVGA', 'XGA', 'WXGA', 'SXGA', 'SXGAM', 'UXGA', 'HD', 'FHD', 'QHD', 'QXGA', 'WQXGA', 'WQXGA2', 'FRAME_SIZE_INVAILD'] Sensor EXP lock: NOT SUPPORTED Sensor GAIN: NOT SUPPORTED Sensor AWB: auto (not lockable on this fw) vb common pool count 4 sensor(0), mode 0, buffer_num 4, buffer_size 0 not support alloc_extra_fb now...
```
然后可以总结出
从 `dir(sensor)` 里提取的**所有有用方法**：
### 曝光（Exposure）
```
set_auto_exposure(enable)    ← 开关自动曝光
get_exposure_us()            ← 读当前曝光值 (微秒)
```
### 增益（Gain）
```
set_auto_gain(enable)        ← 开关自动增益
get_gain_db()                ← 读当前增益值 (dB)
again(value)                 ← 手动设模拟增益 (float)
```
### 白平衡（White Balance）
```
set_auto_whitebal(enable)    ← 开关自动白平衡
get_rgb_gain_db()            ← 读 RGB 各通道增益值
```
### 黑电平（Black Level）
```
set_auto_blc(enable)         ← 开关自动黑电平校准 (关掉减少暗部噪点)
get_blc_regs()               ← 读黑电平寄存器值
```
### 图像质量
```
set_brightness(-2~2)         ← 亮度
set_contrast(-2~2)           ← 对比度
set_saturation(-2~2)         ← 饱和度
set_quality(0~100)           ← JPEG 质量
set_colorbar(enable)         ← 彩条测试模式
```
### 镜像/旋转
```
set_hmirror(enable)          ← 水平镜像
set_vflip(enable)            ← 垂直翻转
set_transpose(enable)        ← 转置
set_auto_rotation(enable)    ← 自动旋转
```
### 帧率/窗口
```
set_framerate(fps)           ← 设置帧率
set_windowing(x,y,w,h)       ← 设置 ROI 窗口
skip_frames(n)               ← 跳过 N 帧 (让 sensor 稳定)
```
### 底层寄存器（终极手段）
```
__write_reg(addr, val)       ← 直接写 GC2093 寄存器
__read_reg(addr)             ← 直接读 GC2093 寄存器
```