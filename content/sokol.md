## sokol图形库

vlib/sokol模块已经对soko图形库l进行了封装,可以使用

### sokol图形库参考

sokol是ui依赖的C图形库,如果想要更清楚理解ui是如何运作的,得掌握一下所依赖的sokol图形库:

官方网址:https://github.com/floooh/sokol

官方DEMO:https://floooh.github.io/sokol-html5/index.html (WASM版本)

官方DEMO源代码:https://github.com/floooh/sokol-samples

作者整体思路文档:https://floooh.github.io/2017/07/29/sokol-gfx-tour.html

官方简介:简单,单文件,跨平台库,可供C/C++使用,C写的

整个库包含这几个C文件,每个C文件可以单独使用:

Cross-platform libraries:

- **sokol_gfx.h**: 3D-API wrapper (GL + Metal + D3D11)
- **sokol_app.h**: app framework wrapper (entry + window + 3D-context + input)
- **sokol_time.h**: time measurement
- **sokol_audio.h**: minimal buffer-streaming audio playback
- **sokol_fetch.h**: asynchronous data streaming from HTTP and local filesystem
- **sokol_args.h**: unified cmdline/URL arg parser for web and native apps

Utility libraries:

- **sokol_imgui.h**: sokol_gfx.h rendering backend for [Dear ImGui](https://github.com/ocornut/imgui)
- **sokol_gl.h**: OpenGL 1.x style immediate-mode rendering API on top of sokol_gfx.h
- **sokol_fontstash.h**: sokol_gl.h rendering backend for [fontstash](https://github.com/memononen/fontstash)
- **sokol_gfx_imgui.h**: debug-inspection UI for sokol_gfx.h (implemented with Dear ImGui)

vui目前使用了这4个,主要是前两个核心文件,已包含在V源代码的thirdparth/sokol中,无需单独下载

**sokol_gfx.h**

- 简单,现代地封装了GLES2/WebGL, GLES3/WebGL2, GL3.3, D3D11 和 Metal
- 提供buffers, images, shaders, pipeline-state-objects 和 render-passes
- 无需控制窗体的创建或者3D API的上下文初始化
- 无需提供着色器方言交叉翻译

**sokol_app.h**

-  统一的应用入口
-  单窗体或画布提供3D渲染
-  3D上下文初始化
-  事件驱动的键盘,鼠标,触摸板输入
-  支持的平台: Win32, MacOS, Linux (X11), iOS, WASM/asm.js, Android (planned: RaspberryPi)
-  支持的3D-APIs: GL3.3 (GLX/WGL), Metal, D3D11, GLES2/WebGL, GLES3/WebGL2

**sokol_gl.h**

OpenGL 1.x样式的立即模式渲染API,基于sokol_gfx.h

**sokol_fontstash.h**

为[fontstash](https://github.com/memononen/fontstash)提供渲染后端

### sapp模块

- 枚举

  EventType  //事件类型

  MouseButton  //鼠标按键

  KeyCode //键盘按键

- 结构体

  sapp_desc //应用程序选项

  sapp_event //窗体事件:鼠标,键盘,触摸板等窗口事件

  sapp_touchpoint //触摸板事件类型

- 回调函数

  init_cb //完成初始化后回调

  frame_cb  //刷新每一帧之前回调,通常是1秒钟,回调60次

  event_cb  //捕捉事件后回调

  cleanup_cb //退出前回调

  fail_cb //报错后回调

- 带用户自定义数据的回调函数

    init_userdata_cb

    frame_userdata_cb

    event_userdata_cb

    cleanup_userdata_cb

    fail_userdata_cb

  简单例子:

  ```c
  module main
  
  import (
  	sokol
  	sokol.sapp
  )
  
  fn main() {
  	//创建app
  	app:=sapp_desc {
  		window_title:'myapp'.str
  		width:640
  		height:480
  		high_dpi:true
  
  		init_cb:init_cb //app完成初始化后回调
  		frame_cb:frame_cb //app刷新每一帧之前回调,通常是1秒钟,回调60次
  		cleanup_cb:cleanup_cb //app退出前回调
  		fail_cb:fail_cb //app报错后回调
  		event_cb:event_cb //app捕捉事件后回调
  	}
  	//启动
  	sapp.run(&app)
  }
  
  fn init_cb() {
  	v:=sapp.isvalid()
  	println(v)
  	sapp.show_mouse(false)
  }
  fn frame_cb() {
  
  }
  fn cleanup_cb() {
  
  }
  fn fail_cb(msg byteptr) {
  
  }
  fn event_cb(e voidptr) {
  	event:=&sapp_event(e)
  	match event.@type {
  		sapp.EventType.key_up {
  			println(byte(event.key_code).str())
  			if event.key_code==sapp.KeyCode.q {
  				sapp.quit()
  			}
  		}
  		sapp.EventType.mouse_move {
  			println('x:$event.mouse_x,y:$event.mouse_y')
  			println('framecount:$event.frame_count')
  		}
  		sapp.EventType.resized {
  			println('resized:width:$event.window_width,height:$event.window_height')
  		}
  		else {
  
  		}
  	}
  }
  
  ```

---

### sgx模块

- 枚举

  Backend //后端类型

  PixelFormat //像素点格式

  ResourceState

  Usage

  BufferType

  ImageType

  CubeFace

  ShaderStage

  PrimitiveType

  Filter

  Wrap

  BorderColor

  VertexFormat //顶点数据格式

  VertexStep

  UniformType

  CullMode

  FaceWinding

  CompareFunc

  StencilOp

  BlendFactor

  BlendOp

  ColorMask

  Action

- 结构体

  sg_desc //gfx选项

  sg_context //上下文

  

  sg_buffer_desc //缓存数据选项

  sg_image_desc //图片数据选项

  sg_shader_desc //着色器选项

  sg_pipeline_desc //渲染管线选项

  sg_pass_desc //渲染通道选项

  sg_attachment_desc //颜色附加点选项

  

  sg_buffer //缓存数据

  sg_image //图像数据

  sg_shader //着色器

  sg_pipeline //渲染管线

  sg_pass //渲染通路,一次绘制

  

  sg_slot_info

  sg_buffer_info

  sg_image_info

  sg_shader_info

  sg_pipeline_info

  sg_pass_info

  

- 函数

  初始化gfx:

  setup() //根据选项,初始化gfx

  关闭sg:

  shutdown() //关闭gfx

  

  创建资源对象:

  make_buffer()

  make_image()

  make_shader()

  make_pipeline()

  make_pass()

  

  开始渲染:

  begin_default_pass() //开始默认的渲染通道

  begin_pass() 开始自定义渲染通道

  

  设置:

  apply_viewport()  //设置视口

  apply_scissor_rect() //设置裁剪矩形

  apply_pipeline() //设置渲染管线

  apply_bindings() //设置资源绑定

  apply_uniforms() //设置统一变量

  

  绘制:

  draw()

  

  完成/提交:

  end_pass() //结束渲染通道

  commit() //提交

  

  更新资源对象:

  update_buffer() //更新缓存数据

  update_image() //更新图像数据

  append_buffer() //追加缓存数据

  

  查询内部资源属性:

  query_buffer_info()

  query_image_info()

  query_shader_info()

  query_pipeline_info()

  query_pass_info()

  

  查询后端:

  query_backend()

  

  销毁资源对象:

  destroy_buffer()

  destroy_image()

  destroy_shader()

  destroy_pipeline()

  destroy_pass()

  

  上下文:

  setup_context()

  activate_context()

  discard_context()

  ---

  ###

### sgl模块

- 枚举

  SglError

- 结构体

  sgl_pipeline

  sgl_desc_t

- 函数

  初始化:

  setup()

  关闭:

  shutdown()

  创建/销毁:

  make_pipeline()

  destroy_pipeline()

  

  渲染状态函数:

  viewport()

  scissor_rect()

  enable_texture()

  disable_texture()

  texture()

  

  渲染命令:

  begin_points()

  begin_lines()

  begin_line_strip()

  begin_triangles()

  begin_triangle_strip()

  begin_quads()

  ...

  绘制:

  draw()

  结束:

  end()

### sfons模块

​	fontstash的字体渲染器

​	create() //创建字体上下文

​	destroy() //销毁字体上下文

​	rgba() //转换颜色格式从RGBA到uint32_t,fontstash需要的

​	flush()

