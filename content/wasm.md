## 生成wasm

V编译器支持wasm后端，将V源代码编译生成wasm代码。

### 安装依赖

当第一次运行 `v -b wasm xxx.v`时，编译器会提示，缺少外部依赖库[binaryen](https://github.com/WebAssembly/binaryen)，需要先执行脚本下载对应平台的预编译版本。安装脚本会下载并解压到path_to_v/thirdparty/binaryen目录中。

```shell
path_to_v/cmd/tools/install_binaryen.vsh
```

也可以直接下载binaryen源代码，自行编译，编译步骤如下：

```shell
git clone https://github.com/WebAssembly/binaryen.git
git submodule init
git submodule update
cmake . && make #需要提前安装好cmake
make install
```

### 编译目标系统

```shell
v -b wasm -os wasi #编译为符合wasi标准的代码，不特别指定-os选项值，默认是wasi
v -b wasm -os browser #编译为浏览器环境的代码
```

示例代码：

mandelbrot.v

```v
fn JS.canvas_x() int
fn JS.canvas_y() int
fn JS.setpixel(x int, y int, c f64)

// `main` must be public!
pub fn main() {
	max_x := JS.canvas_x()
	max_y := JS.canvas_y()

	println('starting main.main!')

	mut y := 0
	for y < max_y {
		y += 1
		mut x := 0
		for x < max_x {
			x += 1

			e := (f64(y) / 50) - 1.5
			f := (f64(x) / 50) - 1.0

			mut a := 0.0
			mut b := 0.0
			mut i := 0.0
			mut j := 0.0
			mut c := 0.0

			for i * i + j * j < 4 && c < 255 {
				i = a * a - b * b + e
				j = 2 * a * b + f
				a = i
				b = j
				c += 1
			}

			JS.setpixel(x, y, c)
		}
	}

	panic('reached the end!')
}
```

mandelbrot.html

```html
<!DOCTYPE html>
<html lang="en">
<head>
	<meta charset="UTF-8">
	<meta http-equiv="X-UA-Compatible" content="IE=edge">
	<meta name="viewport" content="width=device-width, initial-scale=1.0">
	<title>V Mandelbrot WebAssembly Example</title>
</head>
<body>
	<canvas id="canvas" width="200" height="200" style="width:100%;height:100%;image-rendering: crisp-edges;"></canvas>
	<script>
		var canvas = document.getElementById("canvas");
		var ctx = canvas.getContext("2d");
		var memory;

		function get_string(ptr, len) {
			const buf = new Uint8Array(memory.buffer, ptr, len);
			const str = new TextDecoder("utf8").decode(buf);

			return str
		}

		const env = {
			canvas_x: () => canvas.width,
			canvas_y: () => canvas.height,
			setpixel: (x, y, c) => {
				ctx.fillStyle = "rgba(1,1,1,"+(c/255)+")";
				ctx.fillRect(x, y, 1, 1);
			},
			__writeln: (ptr, len) => {
				console.log(get_string(ptr, len))
			},
			__panic_abort: (ptr, len) => {
				throw get_string(ptr, len);
			}
		}

		WebAssembly.instantiateStreaming(fetch("mandelbrot.wasm"), {env: env}).then((res) => {
			memory = res.instance.exports['memory'];
			
			console.time('main.main')
			res.instance.exports['main.main']()
			console.timeEnd('main.main')
		});
	</script>
</body>
</html>
```

编译生成wasm文件：

```shell
v -b wasm -os browser mandelbrot.v
```

就可以打开mandelbrot.html运行wasm文件。
