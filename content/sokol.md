## sokol

vlib/sokol模块已经对sokol进行了封装,可以初步使用

### sapp

- 枚举

  EventType  //事件类型

  MouseButton  //鼠标按键

  KeyCode //键盘按键

- 结构体

  sapp_desc //应用程序

  sapp_event //窗体事件:鼠标,键盘,触摸板等窗口事件

  sapp_touchpoint //触摸板事件类型

- 回调函数

  init_cb

  frame_cb 

  event_cb

  cleanup_cb

  fail_cb

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

### sgx

- 枚举

  Backend

  PixelFormat

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

  VertexFormat

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

  sg_desc

  sg_context

  

  sg_buffer_desc

  sg_image_desc

  sg_shader_desc

  sg_pipeline_desc

  sg_pass_desc

  sg_attachment_desc

  

  sg_buffer

  sg_image

  sg_shader

  sg_pipelinie

  sg_pass

  

  sg_slot_info

  sg_buffer_info

  sg_image_info

  sg_shader_info

  sg_pipeline_info

  sg_pass_info

  

- 函数

  初始化sg:

  setup()

  关闭sg:

  shutdown()

  

  创建资源对象:

  make_buffer()

  make_image()

  make_shader()

  make_pipeline()

  make_pass()

  

  开始渲染:

  begin_default_pass()

  begin_pass()

  

  设置:

  apply_viewport()

  apply_scissor_rect()

  apply_pipeline()

  apply_bindings()

  apply_uniforms()

  

  绘制:

  draw()

  

  完成/提交:

  end_pass()

  commit()

  

  更新资源对象:

  update_buffer()

  update_image()

  append_buffer()

  

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

  ###sgl

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

    

  ---

  ###sfons

  create()

  destroy()

  rgba()

  flush()

  