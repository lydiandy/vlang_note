# Vlang abstract syntax tree(AST) introduction

## Overview

### AST sumtype

```v
pub type TypeDecl = AliasTypeDecl | FnTypeDecl | SumTypeDecl

pub type Expr = AnonFn | ArrayInit | AsCast | Assoc | AtExpr | BoolLiteral | CTempVar |
	CallExpr | CastExpr | ChanInit | CharLiteral | Comment | ComptimeCall | ConcatExpr | EnumVal |
	FloatLiteral | Ident | IfExpr | IfGuardExpr | IndexExpr | InfixExpr | IntegerLiteral |
	Likely | LockExpr | MapInit | MatchExpr | None | OrExpr | ParExpr | PostfixExpr | PrefixExpr |
	RangeExpr | SelectExpr | SelectorExpr | SizeOf | SqlExpr | StringInterLiteral | StringLiteral |
	StructInit | Type | TypeOf | UnsafeExpr

pub type Stmt = AssertStmt | AssignStmt | Block | BranchStmt | CompFor | ConstDecl | DeferStmt |
	EnumDecl | ExprStmt | FnDecl | ForCStmt | ForInStmt | ForStmt | GlobalDecl | GoStmt |
	GotoLabel | GotoStmt | HashStmt | Import | InterfaceDecl | Module | Return | SqlStmt |
	StructDecl | TypeDecl
```

All the AST struct declarations can be found in V source code: [vlib/v/ast/ast.v](https://github.com/vlang/v/blob/master/vlib/v/ast/ast.v)

### Diagram



## AST tool

If you are new of Vlang AST, You can install the [vast tool](https://github.com/lydiandy/vast). It can generate example code to AST json format.

The json file can help you more understand the AST.

```shell
vast example.v       //generate example.json file and exit.

vast -w example.v    //generate example.json and watch,if file change,regenerate.

```



## File

AST struct

```v
// Each V source file is represented by one ast.File structure.
// When the V compiler runs, the parser will fill an []ast.File.
// That array is then passed to V's checker.
pub struct File {
pub:
	path             string // path of the source file
	mod              Module // the module of the source file (from `module xyz` at the top)
	global_scope     &Scope
pub mut:
	scope            &Scope
	stmts            []Stmt // all the statements in the source file
	imports          []Import // all the imports
	imported_symbols map[string]string // used for `import {symbol}`, it maps symbol => module.symbol
	errors           []errors.Error // all the checker errors in the file
	warnings         []errors.Warning // all the checker warings in the file
	generic_fns      []&FnDecl
}
```

example code

```v
module main

import os
import time

fn main() {
  
}
```

generate AST json

```json
{
	"ast_type": "ast.File",
	"path": "/Users/xxx/v/vprojects/v_test/main.v",
	"mod": {
		"ast_type": "Module",
		"name": "main",
		"is_skipped": false,
		"pos": {
			"line_nr": 0,
			"pos": 0,
			"len": 19
		}
	},
	"imports": [
		{
			"ast_type": "Import",
			"mod": "os",
			"alias": "os",
			"syms": [],
			"pos": {
				"line_nr": 2,
				"pos": 20,
				"len": 2
			}
		},
		{
			"ast_type": "Import",
			"mod": "time",
			"alias": "time",
			"syms": [],
			"pos": {
				"line_nr": 3,
				"pos": 30,
				"len": 4
			}
		}
	],
	"global_scope": {
		"ast_type": "Scope",
		"parent": "0",
		"children": [],
		"start_pos": 0,
		"end_pos": 0,
		"objects": {},
		"struct_fields": []
	},
	"scope": {
		"ast_type": "Scope",
		"parent": "7f970ef07c90",
		"children": [
			{
				"parent": "7f970ef081f0",
				"start_pos": 39,
				"end_pos": 51
			}
		],
		"start_pos": 0,
		"end_pos": 53,
		"objects": {},
		"struct_fields": []
	},
	"errors": [],
	"warnings": [],
	"imported_symbols": {},
	"generic_fns": [],
	"stmts": [
		{
			"ast_type": "Module",
			"name": "main",
			"is_skipped": false,
			"pos": {
				"line_nr": 0,
				"pos": 0,
				"len": 19
			}
		},
		{
			"ast_type": "Import",
			"mod": "os",
			"alias": "os",
			"syms": [],
			"pos": {
				"line_nr": 2,
				"pos": 20,
				"len": 2
			}
		},
		{
			"ast_type": "Import",
			"mod": "time",
			"alias": "time",
			"syms": [],
			"pos": {
				"line_nr": 3,
				"pos": 30,
				"len": 4
			}
		},
		{
			"ast_type": "FnDecl",
			"name": "main.main",
			"mod": "main",
			"is_deprecated": false,
			"is_pub": false,
			"is_variadic": false,
			"is_anon": false,
			"receiver": {
				"ast_type": "Field",
				"name": "",
				"typ": "void",
				"pos": {
					"line_nr": 0,
					"pos": 0,
					"len": 0
				}
			},
			"receiver_pos": {
				"line_nr": 0,
				"pos": 0,
				"len": 0
			},
			"is_method": false,
			"method_idx": 0,
			"rec_mut": false,
			"rec_share": "enum:0(mut)",
			"language": "enum:0(v)",
			"no_body": false,
			"is_builtin": false,
			"is_generic": false,
			"is_direct_arr": false,
			"pos": {
				"line_nr": 5,
				"pos": 36,
				"len": 9
			},
			"body_pos": {
				"line_nr": 7,
				"pos": 51,
				"len": 1
			},
			"file": "/Users/xxx/v/vprojects/v_test/main.v",
			"return_type": "void",
			"source_file": 0,
			"scope": 250645264,
			"attrs": [],
			"params": [],
			"stmts": [],
			"comments": [],
			"next_comments": []
		}
	]
}
```

##  Module

### module declaration

AST struct

```v
// module declaration
pub struct Module {
pub:
	name       string
	path       string
	expr       Expr
	pos        token.Position
	is_skipped bool // module main can be skipped in single file programs
}
```

example code

```v
module main

fn main() {
	
}
```

### module import

AST struct

```v
// import statement
pub struct Import {
pub:
	mod   string // the module name of the import
	alias string // the `x` in `import xxx as x`
	pos   token.Position
pub mut:
	syms  []ImportSymbol // the list of symbols in `import {symbol1, symbol2}`
}

// import symbol,for import {symbol} syntax
pub struct ImportSymbol {
pub:
	pos  token.Position
	name string
}
```

example code

```v
module main

import os
import time as t
import math { min, max }
```

## Const

AST struct

```v
// const declaration
pub struct ConstDecl {
pub:
	is_pub       bool
	pos          token.Position
pub mut:
	fields       []ConstField // all the const fields in the `const (...)` block
	end_comments []Comment // comments that after last const field
}

// const field in const declaration group
pub struct ConstField {
pub:
	mod      string
	name     string
	expr     Expr // the value expr of field; everything after `=`
	is_pub   bool
	pos      token.Position
pub mut:
	typ      table.Type // the type of the const field, it can be any type in V
	comments []Comment // comments before current const field
}
```

example code

```v
module main

const (
	// version comment 1
	version = '0.2.0' // version comment 2
	usage   = 'usage:xxxx'
	pi      = 3.14
	//end comment 1
	//end comment 2
)
```

## Enum

 AST struct

```v
// enum declaration
pub struct EnumDecl {
pub:
	name             string
	is_pub           bool
	is_flag          bool // true when the enum has [flag] tag,for bit field enum
	is_multi_allowed bool // true when the enum has [_allow_multiple_values] tag
	comments         []Comment // comments before the first EnumField
	fields           []EnumField // all the enum fields
	attrs            []table.Attr // attributes of enum declaration
	pos              token.Position
}

// enum field in enum declaration
pub struct EnumField {
pub:
	name          string
	pos           token.Position
	comments      []Comment // comment after Enumfield in the same line
	next_comments []Comment // comments between current EnumField and next EnumField
	expr          Expr // the value of current EnumField; 123 in `ename = 123`
	has_expr      bool // true, when .expr has a value
}

// an enum value, like OS.macos or .macos
pub struct EnumVal {
pub:
	enum_name string
	val       string
	mod       string // for full path `mod_Enum_val`
	pos       token.Position
pub mut:
	typ       table.Type
}
```

example code

```v
module main

[attr1]
['attr2=123']
  enum Color { // enum comment 1
	// black comment 1
	// black comment 2
	black = 2 // black comment 3
	// white comment 1
	// white comment 2
	white // white comment 3
	blue
	green // green comment
	// end comment 1
	// end comment 2
}

[flag]
enum BitEnum {
	e1
	e2
	e3
}

[_allow_multiple_values]
enum MultipleEnum {
	v1 = 1
}

fn main() {
	mut color := Color.black
	color = .blue
}
```

## Variable

### Assign

AST struct

```v
// variable assign statement
pub struct AssignStmt {
pub:
	right         []Expr
	op            token.Kind // include: =,:=,+=,-=,*=,/= and so on; for a list of all the assign operators, see vlib/token/token.v
	pos           token.Position
	comments      []Comment
	end_comments  []Comment
pub mut:
	left          []Expr
	left_types    []table.Type
	right_types   []table.Type
	is_static     bool // for translated code only
	is_simple     bool // `x+=2` in `for x:=1; ; x+=2`
	has_cross_var bool
}
```

example code

```v
module main

fn main() {
	// signle assign
	a := 'abc' // comment for a
	mut b := 1
	// more operator
	b = 2
	b += 2
	b -= 2
	b *= 2
	b /= 2
	b %= 2
	// multi assign
	x, y, z := 1, 'y', 3.3
	mut xx, mut yy, zz := 1, 3, 5
	// swap variable
	mut c := 1
	mut d := 2
	c, d = d, c
}

```

### Identifier

AST struct

```v
pub struct IdentFn {
pub mut:
	typ table.Type
}

// TODO: (joe) remove completely, use ident.obj
// instead which points to the scope object
pub struct IdentVar {
pub mut:
	typ         table.Type
	is_mut      bool
	is_static   bool
	is_optional bool
	share       table.ShareType
}

pub type IdentInfo = IdentFn | IdentVar

pub enum IdentKind {
	unresolved
	blank_ident
	variable
	constant
	global
	function
}

// A single identifier
pub struct Ident {
pub:
	language table.Language
	tok_kind token.Kind
	pos      token.Position
	mut_pos  token.Position
pub mut:
	scope    &Scope
	obj      ScopeObject
	mod      string
	name     string
	kind     IdentKind
	info     IdentInfo
	is_mut   bool
}
```

example code(need more)

```v
module main

fn main() {
	i := 123 // common(unresolved) identifier
	_, x := 1, 2 // blank identifier
	mut s := 'abc' // with mut
	s = 'aaa'
}

```

### Literal

AST struct

```v
pub struct IntegerLiteral {
pub:
	val string
	pos token.Position
}

pub struct FloatLiteral {
pub:
	val string
	pos token.Position
}

pub struct StringLiteral {
pub:
	val      string
	is_raw   bool
	language table.Language
	pos      token.Position
}

// 'name: $name'
pub struct StringInterLiteral {
pub:
	vals       []string
	exprs      []Expr
	fwidths    []int
	precisions []int
	pluss      []bool
	fills      []bool
	fmt_poss   []token.Position
	pos        token.Position
pub mut:
	expr_types []table.Type
	fmts       []byte
	need_fmts  []bool // an explicit non-default fmt required, e.g. `x`
}

pub struct CharLiteral {
pub:
	val string
	pos token.Position
}

pub struct BoolLiteral {
pub:
	val bool
	pos token.Position
}
```

example code

```v
module main

fn main() {
	a := 1 // integer literal
	b := 1.2 // float literal
	c := 'abc' // string literal
  name:='tom'
	age:= 33
  //string literal with `$xx` or `${xxx}`
	s1 := 'a is $a,b is $b,c is $c' 
	s2 := 'name is ${name}, age is ${age}'
	e := `c` // char literal
	f := true // bool literal
}
```

### AsCast

AST struct

```v
// as cast statement
pub struct AsCast {
pub:
	expr      Expr // `x` in `x as int`
	typ       table.Type // `int` in `x as int`
	pos       token.Position
pub mut:
	expr_type table.Type
}
```

example code

```v
module main

type Mysumtype = bool | f64 | int | string

fn main() {
	x := Mysumtype(3)
	x2 := x as int // as must be used for sumtype
	println(x2)
}
```

### SizeOf

AST struct

```v
// the builtin sizeof function,can be used for type and variable
pub struct SizeOf {
pub:
	is_type   bool // true, if argument is a type
	typ       table.Type
	type_name string
	expr      Expr
	pos       token.Position
}
```

example code

```v
module main

struct Point {
	x int
	y int
}

fn main() {
	a := sizeof(int) // basic type
	b := sizeof(bool) // basic type
	p := Point{
		x: 1
		y: 2
	}
	s1 := sizeof(Point) // struct type
	s2 := sizeof(p) // variable
}
```

### TypeOf

AST struct

```v
//the builtin typeof function
pub struct TypeOf {
pub:
	expr      Expr
	pos       token.Position
pub mut:
	expr_type table.Type
}
```

example code

```v
module main

type MySumType = f32 | int

fn myfn(i int) int {
	return i
}

fn main() {
	a := 123
	s := 'abc'
	aint := []int{}
	astring := []string{}
	println(typeof(a)) // int
	println(typeof(s)) // string
	println(typeof(aint)) // array_int
	println(typeof(astring)) // array_string
	// sumtype
	sa := MySumType(32)
	println(typeof(sa)) // int
	// function type
	println(typeof(myfn)) // fn (int) int
}

```

## Array

### ArrayInit

AST struct

```v
pub struct ArrayInit {
pub:
	pos            token.Position // `[]` in []Type{} position
	elem_type_pos  token.Position // `Type` in []Type{} position
	exprs          []Expr // `[expr, expr]` or `[expr]Type{}` for fixed array
	ecmnts         [][]Comment // optional iembed comments after each expr
	is_fixed       bool
	has_val        bool // fixed size literal `[expr, expr]!!`
	mod            string
	len_expr       Expr // len: expr
	cap_expr       Expr // cap: expr
	default_expr   Expr // init: expr
	has_len        bool
	has_cap        bool
	has_default    bool
pub mut:
	expr_types     []table.Type // [Dog, Cat] // also used for interface_types
	is_interface   bool // array of interfaces e.g. `[]Animal` `[Dog{}, Cat{}]`
	interface_type table.Type // Animal
	elem_type      table.Type // element type
	typ            table.Type // array type
}

```

example code

```v
module main

fn main() {
	mut arr := []string{len: 3, cap: 6, init: 'default'}
	arr[0] = 'a'
	arr[1] = 'b'
	println(arr)
}

```

### IndexExpr

AST struct

```v
pub struct IndexExpr {
pub:
	pos       token.Position
	left      Expr
	index     Expr // [0], RangeExpr [start..end] or map[key]
	or_expr   OrExpr
pub mut:
	left_type table.Type // array, map, fixed array
	is_setter bool
}
```

example code

```v

```



## Map

### MapInit

AST struct

```v
pub struct MapInit {
pub:
	pos        token.Position
	keys       []Expr
	vals       []Expr
pub mut:
	typ        table.Type
	key_type   table.Type
	value_type table.Type
}
```

example code

```v
module main

fn main() {
	mut m := map[string]string{}
	m['name'] = 'tom'
	m['age'] = '33'
	println(m)
	m2 := {
		'one':   1
		'two':   2
		'three': 3
	}
	println(m2)
}

```

## Expr

### RangeExpr

AST struct

```v

```

example code

```v

```



### CastExpr

AST struct

```v

```

example code

```v

```



### PrefixExpr

AST struct

```v

```

example code

```v

```



### InfixExpr

AST struct

```v

```

example code

```v

```



### PostfixExpr

AST struct

```v

```

example code

```v

```



### ConcatExpr

AST struct

```v

```

example code

```v

```



### SelectorExpr

AST struct

```v

```

example code

```v

```



### AtExpr

AST struct

```v

```

example code

```v

```



### Likely

AST struct

```v

```

example code

```v

```



### ParExpr

AST struct

```v

```

example code

```v

```



## Function/Method

### Function declaration

AST struct

```v
//function or method declaration
pub struct FnDecl {
pub:
	name            string
	mod             string
	params          []table.Param
	is_deprecated   bool
	is_pub          bool
	is_variadic     bool
	is_anon         bool
	receiver        Field
	receiver_pos    token.Position
	is_method       bool
	method_type_pos token.Position
	method_idx      int
	rec_mut         bool // is receiver mutable
	rec_share       table.ShareType
	language        table.Language
	no_body         bool // just a definition `fn C.malloc()`
	is_builtin      bool // this function is defined in builtin/strconv
	pos             token.Position
	body_pos        token.Position
	file            string
	is_generic      bool
	is_direct_arr   bool // direct array access
	attrs           []table.Attr
pub mut:
	stmts           []Stmt
	return_type     table.Type
	comments        []Comment // comments *after* the header, but *before* `{`; used for InterfaceDecl
	source_file     &File = 0
	scope           &Scope
}

//function or method call
pub struct CallExpr {
pub:
	pos                token.Position
	left               Expr // `user` in `user.register()`
	mod                string
pub mut:
	name               string // left.name()
	is_method          bool
	is_field           bool // temp hack, remove ASAP when re-impl CallExpr / Selector (joe)
	args               []CallArg
	expected_arg_types []table.Type
	language           table.Language
	or_block           OrExpr
	left_type          table.Type // type of `user`
	receiver_type      table.Type // User
	return_type        table.Type
	should_be_skipped  bool
	generic_type       table.Type // TODO array, to support multiple types
	generic_list_pos   token.Position
	free_receiver      bool // true if the receiver expression needs to be freed
	scope              &Scope
	from_embed_type    table.Type // holds the type of the embed that the method is called from
}

//function call argument
pub struct CallArg {
pub:
	is_mut          bool
	share           table.ShareType
	expr            Expr
	comments        []Comment
pub mut:
	typ             table.Type
	is_tmp_autofree bool // this tells cgen that a tmp variable has to be used for the arg expression in order to free it after the call
	pos             token.Position
	// tmp_name        string // for autofree
}

//return statement
pub struct Return {
pub:
	pos      token.Position
	exprs    []Expr
	comments []Comment
pub mut:
	types    []table.Type
}
```

example code

```v
module main

fn main() {
	s := add(1, 3)
	println(s)
}

pub fn add(x int, y int) int {
	return x + y
}
```



### Function call

AST struct

```v

```

example code

```v

```



### Return

AST struct

```v

```

example code

```v

```



### Anonymous function

AST struct

```v
//anonymous function
pub struct AnonFn {
pub:
	decl FnDecl
pub mut:
	typ  table.Type
}
```

example code

```v
module main

fn main() {
	f1 := fn (x int, y int) int {
		return x + y
	}
	f1(1,3)
}

```



### DeferStmt

AST struct

```v

```

example code

```v

```







## Struct

### StructDecl

AST struct

```v

```

example code

```v

```

### StructInit

AST struct

```v

```

example code

```v

```



### Assoc

AST struct

```v

```

example code

```v

```



## Interface

### InterfaceDecl

AST struct

```v

```

example code

```v

```



## Type

### Alias Type 

AST struct

```v

```

example code

```v

```



### Function Type

AST struct

```v

```

example code

```v

```



### SumType

AST struct

```v

```

example code

```v

```



## FlowControl

### Block

Block

AST struct

```v

```

example code

```v

```



### if

IfExpr

AST struct

```v

```

example code

```v

```



IfGuardExpr

AST struct

```v

```

example code

```v

```



### match

MatchExpr

AST struct

```v

```

example code

```v

```



BranchStmt

AST struct

```v

```

example code

```v

```



### for

ForCStmt

AST struct

```v

```

example code

```v

```



ForInStmt

AST struct

```v

```

example code

```v

```



ForStmt

AST struct

```v

```

example code

```v

```



### goto

GotoLabel

AST struct

```v

```

example code

```v

```



GotoStmt

AST struct

```v

```

example code

```v

```



AST struct

```v

```

example code

```v

```



## Error handle

### OrExpr

AST struct

```v

```

example code

```v

```



### None

AST struct

```v

```

example code

```v

```



## Concurrent

### ChanInit

AST struct

```v

```

example code

```v

```



### GoStmt

AST struct

```v

```

example code

```v

```



### SelectExpr

AST struct

```v

```

example code

```v

```



### LockExpr

AST struct

```v

```

example code

```v

```



## Unsafe

### UnsafeExpr

AST struct

```v

```

example code

```v

```



## SQL

### SqlStmt

AST struct

```v

```

example code

```v

```



### SqlExpr

AST struct

```v

```

example code

```v

```



## Test

### AssertStmt

AST struct

```v

```

example code

```v

```



## Compile time

### ComptimeCall

AST struct

```v

```

example code

```v

```



### CompFor

AST struct

```v

```

example code

```v

```



## C Integration

### GlobalDecl

AST struct

```v

```

example code

```v

```



### HashStmt

AST struct

```v

```

example code

```v

```



### CTempVar

AST struct

```v

```

example code

```v

```



## Comment

### Commen

AST struct

```v

```

example code

```v

```
