## 生成wasm

V编译器支持wasm后端，将V源代码编译生成wasm代码。

编译器后端使用[binaryen](https://github.com/WebAssembly/binaryen)来生成wasm代码，安装步骤如下：

```shell
git clone https://github.com/WebAssembly/binaryen.git
git submodule init
git submodule update
cmake . && make #需要提前安装好cmake
make install
```

安装binaryen封装库：

```shell
git clone https://github.com/l1mey112/binaryen-v.git
```

示例代码：

mandelbrot.v

```v
// v -b wasm -no-builtin mandelbrot.v # create `mandelbrot.wasm`
// 
// python -m http.server 8080
// emrun mandelbrot.html
// ....
// ....

fn JS.canvas_x() int
fn JS.canvas_y() int
fn JS.setpixel(x int, y int, c f64)

fn main() {
	max_x := JS.canvas_x()
	max_y := JS.canvas_y()

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

		const env = {
			__vsp: new WebAssembly.Global({value: "i32", mutable: true}, 0),
			__vmem: new WebAssembly.Memory({initial: 256, maximum: 256}),
			canvas_x: () => canvas.width,
			canvas_y: () => canvas.height,
			setpixel: (x, y, c) => {
				ctx.fillStyle = "rgba(1,1,1,"+(c/255)+")";
				ctx.fillRect(x, y, 1, 1);
			}
		}

		WebAssembly.instantiateStreaming(fetch("mandelbrot.wasm"), {env: env}).then((res) => {
			console.time('main.main')
			res.instance.exports['main.main']()
			console.timeEnd('main.main')
		});
	</script>
</body>
</html>
```

编译，生成wasm文件：

```shell
v -b wasm mandelbrot.v
```

就可以打开mandelbrot.html运行wasm文件。