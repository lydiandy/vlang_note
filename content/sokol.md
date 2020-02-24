## sokol

vlib/sokol模块已经对sokol进行了封装,可以初步使用

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

  

