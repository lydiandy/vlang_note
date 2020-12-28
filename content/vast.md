# V AST(Abstract Syntax Tree)

## Introduction



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



## File

AST struct:

```v
//Each V source file will generate an ast.File.
//When V compiler runs,the Parser will generate from [ ]os.File to [ ]ast.File
pub struct File {
pub:
	path             string				//path of the source file
	mod              Module				//current module
	global_scope     &Scope
pub mut:
	scope            &Scope
	stmts            []Stmt					//all the source code AST
	imports          []Import				//all the imports
  imported_symbols map[string]string	 //use for import {symbol},Parser find symbol=>module.symbol
	errors           []errors.Error		//all the checker errors in the file
	warnings         []errors.Warning	//all the checker warings in the file
	generic_fns      []&FnDecl
}
```

example code:

```v
module main

import os
import time
```

generate AST:

```json
{
	"ast_type":	"ast.File",
	"path":	"your/source file/path",
	"mod":	{
		"ast_type":	"Module",
		"name":	"main",
		"is_skipped":	false,
		"pos":	{
			"line_nr":	0,
			"pos":	0,
			"len":	19
		}
	},
	"imports":	[{
			"ast_type":	"Import",
			"mod":	"os",
			"alias":	"os",
			"syms":	[],
			"pos":	{
				"line_nr":	2,
				"pos":	20,
				"len":	2
			}
		}, {
			"ast_type":	"Import",
			"mod":	"time",
			"alias":	"time",
			"syms":	[],
			"pos":	{
				"line_nr":	3,
				"pos":	30,
				"len":	4
			}
		}],
	"global_scope":	{
		"ast_type":	"Scope",
		"parent":	"0",
		"children":	[],
		"start_pos":	0,
		"end_pos":	0,
		"objects":	{
		},
		"struct_fields":	[]
	},
	"scope":	{
		"ast_type":	"Scope",
		"parent":	"7ff5ffc07740",
		"children":	[],
		"start_pos":	0,
		"end_pos":	35,
		"objects":	{
		},
		"struct_fields":	[]
	},
	"errors":	[],
	"warnings":	[],
	"imported_symbols":	{
	},
	"generic_fns":	[],
	"stmts":	[{
			"ast_type":	"Module",
			"name":	"main",
			"is_skipped":	false,
			"pos":	{
				"line_nr":	0,
				"pos":	0,
				"len":	19
			}
		}, {
			"ast_type":	"Import",
			"mod":	"os",
			"alias":	"os",
			"syms":	[],
			"pos":	{
				"line_nr":	2,
				"pos":	20,
				"len":	2
			}
		}, {
			"ast_type":	"Import",
			"mod":	"time",
			"alias":	"time",
			"syms":	[],
			"pos":	{
				"line_nr":	3,
				"pos":	30,
				"len":	4
			}
		}]
}
```

##  Module

### module declaration

AST struct:

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

genereate AST:

```json
{
			"ast_type":	"Module",
			"name":	"main",
			"is_skipped":	false,
			"pos":	{
				"line_nr":	0,
				"pos":	1,
				"len":	20
			}
}
```

### module import

AST struct:

```v
// import statement
pub struct Import {
pub:
	pos   token.Position
	mod   string	//imported module name
	alias string	//if use import xxx as x,alias is different from mod
pub mut:
	syms  []ImportSymbol //use for import {symbol},Parser find symbol=>module.symbol
}

pub struct ImportSymbol {
pub:
	pos  token.Position
	name string
}
```

example code:

```v
module main

import os
import time as t
import math { min, max }
```

generate AST:

```json
"stmts":	[{
			"ast_type":	"Module",
			"name":	"main",
			"is_skipped":	false,
			"pos":	{
				"line_nr":	0,
				"pos":	0,
				"len":	18
			}
		}, {
			"ast_type":	"Import",
			"mod":	"os",
			"alias":	"os",
			"syms":	[],
			"pos":	{
				"line_nr":	1,
				"pos":	19,
				"len":	2
			}
		}, {
			"ast_type":	"Import",
			"mod":	"time",
			"alias":	"t",
			"syms":	[],
			"pos":	{
				"line_nr":	2,
				"pos":	29,
				"len":	4
			}
		}, {
			"ast_type":	"Import",
			"mod":	"math",
			"alias":	"math",
			"syms":	[{
					"name":	"min",
					"pos":	{
						"line_nr":	3,
						"pos":	53,
						"len":	3
					}
				}, {
					"name":	"max",
					"pos":	{
						"line_nr":	3,
						"pos":	58,
						"len":	3
					}
				}],
			"pos":	{
				"line_nr":	3,
				"pos":	46,
				"len":	4
			}
		}]
```

## Const

AST struct:

```v
//const declaration
pub struct ConstDecl {
pub:
	is_pub       bool
	pos          token.Position
pub mut:
	fields       []ConstField 	//all the const fields
	end_comments []Comment			//comments that after last const field
}

pub struct ConstField {
pub:
	mod      string
	name     string
  expr     Expr		//the value expr of field,after =
	is_pub   bool
	pos      token.Position
pub mut:
	typ      table.Type	//type of const field,const field in V can be any type
	comments []Comment	//comments before current const field
}
```

example code:

```v
module main

const (
	// version comment 1
	version = '0.2.0' // version comment 2
	//usage comment 1
	usage   = 'usage:xxxx' // usage comment 2
	//pi comment 1
	pi      = 3.14 // pi comment 2
	//end comment 1
	//end comment 2
)

```

generate AST:

```json
	"stmts": [
		{
			"ast_type": "ConstDecl",
			"is_pub": false,
			"fields": [
				{
					"ast_type": "ConstField",
					"mod": "main",
					"name": "main.version",
					"expr": {
						"ast_type": "StringLiteral",
						"val": "0.2.0",
						"is_raw": false,
						"language": 0,
						"pos": {
							"line_nr": 4,
							"pos": 54,
							"len": 7
						}
					},
					"is_pub": false,
					"typ": null,
					"pos": {
						"line_nr": 4,
						"pos": 44,
						"len": 7
					},
					"comments": [
						{
							"ast_type": "Comment",
							"text": "\u0001 version comment 1",
							"pos": {
								"line_nr": 3,
								"pos": 21,
								"len": 21
							}
						}
					]
				},
				{
					"ast_type": "ConstField",
					"mod": "main",
					"name": "main.usage",
					"expr": {
						"ast_type": "StringLiteral",
						"val": "usage:xxxx",
						"is_raw": false,
						"language": 0,
						"pos": {
							"line_nr": 6,
							"pos": 113,
							"len": 12
						}
					},
					"is_pub": false,
					"typ": null,
					"pos": {
						"line_nr": 6,
						"pos": 103,
						"len": 5
					},
					"comments": [
						{
							"ast_type": "Comment",
							"text": " version comment 2",
							"pos": {
								"line_nr": 4,
								"pos": 62,
								"len": 20
							}
						},
						{
							"ast_type": "Comment",
							"text": "\u0001usage comment 1",
							"pos": {
								"line_nr": 5,
								"pos": 83,
								"len": 18
							}
						}
					]
				},
				{
					"ast_type": "ConstField",
					"mod": "main",
					"name": "main.pi",
					"expr": {
						"ast_type": "FloatLiteral",
						"val": "3.14",
						"pos": {
							"line_nr": 8,
							"pos": 172,
							"len": 4
						}
					},
					"is_pub": false,
					"typ": null,
					"pos": {
						"line_nr": 8,
						"pos": 162,
						"len": 2
					},
					"comments": [
						{
							"ast_type": "Comment",
							"text": " usage comment 2",
							"pos": {
								"line_nr": 6,
								"pos": 126,
								"len": 18
							}
						},
						{
							"ast_type": "Comment",
							"text": "\u0001pi comment 1",
							"pos": {
								"line_nr": 7,
								"pos": 145,
								"len": 15
							}
						}
					]
				}
			],
			"pos": {
				"line_nr": 2,
				"pos": 13,
				"len": 5
			},
			"end_comments": [
				{
					"ast_type": "Comment",
					"text": " pi comment 2",
					"pos": {
						"line_nr": 8,
						"pos": 177,
						"len": 15
					}
				},
				{
					"ast_type": "Comment",
					"text": "\u0001end comment 1",
					"pos": {
						"line_nr": 9,
						"pos": 193,
						"len": 16
					}
				},
				{
					"ast_type": "Comment",
					"text": "\u0001end comment 2",
					"pos": {
						"line_nr": 10,
						"pos": 210,
						"len": 16
					}
				}
			]
		}
	]
```

## Enum

 AST struct:

```v
pub struct EnumDecl {
pub:
	name             string
	is_pub           bool
	is_flag          bool // true when the enum has [flag] tag,for bit field enum
	is_multi_allowed bool // true when the enum has [_allow_multiple_values] tag
	comments         []Comment //comments before the first EnumField
	fields           []EnumField
	attrs            []table.Attr
	pos              token.Position
}

pub struct EnumField {
pub:
	name          string
	pos           token.Position
	comments      []Comment	//comment after Enumfield in the same line
	next_comments []Comment 	//comments between current EnumField and next EnumField
	expr          Expr	//the value of current EnumField
	has_expr      bool	//true,when expr is not null
}

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

example code:

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

generate AST:

```json
	"stmts":	[{
			"ast_type":	"EnumDecl",
			"name":	"main.Color",
			"is_pub":	false,
			"is_flag":	false,
			"is_multi_allowed":	false,
			"pos":	{
				"line_nr":	4,
				"pos":	37,
				"len":	10
			},
			"fields":	[{
					"ast_type":	"EnumField",
					"name":	"black",
					"has_expr":	true,
					"expr":	{
						"ast_type":	"IntegerLiteral",
						"val":	"2",
						"pos":	{
							"line_nr":	7,
							"pos":	117,
							"len":	1
						}
					},
					"pos":	{
						"line_nr":	7,
						"pos":	109,
						"len":	5
					},
					"comments":	[{
							"ast_type":	"Comment",
							"text":	" black comment 3",
							"pos":	{
								"line_nr":	7,
								"pos":	119,
								"len":	18
							}
						}],
					"next_comments":	[{
							"ast_type":	"Comment",
							"text":	"\u0001 white comment 1",
							"pos":	{
								"line_nr":	8,
								"pos":	138,
								"len":	19
							}
						}, {
							"ast_type":	"Comment",
							"text":	"\u0001 white comment 2",
							"pos":	{
								"line_nr":	9,
								"pos":	158,
								"len":	19
							}
						}]
				}, {
					"ast_type":	"EnumField",
					"name":	"white",
					"has_expr":	false,
					"expr":	null,
					"pos":	{
						"line_nr":	10,
						"pos":	179,
						"len":	5
					},
					"comments":	[{
							"ast_type":	"Comment",
							"text":	" white comment 3",
							"pos":	{
								"line_nr":	10,
								"pos":	185,
								"len":	18
							}
						}],
					"next_comments":	[]
				}, {
					"ast_type":	"EnumField",
					"name":	"blue",
					"has_expr":	false,
					"expr":	null,
					"pos":	{
						"line_nr":	11,
						"pos":	205,
						"len":	4
					},
					"comments":	[],
					"next_comments":	[]
				}, {
					"ast_type":	"EnumField",
					"name":	"green",
					"has_expr":	false,
					"expr":	null,
					"pos":	{
						"line_nr":	12,
						"pos":	211,
						"len":	5
					},
					"comments":	[{
							"ast_type":	"Comment",
							"text":	" green comment",
							"pos":	{
								"line_nr":	12,
								"pos":	217,
								"len":	16
							}
						}],
					"next_comments":	[{
							"ast_type":	"Comment",
							"text":	"\u0001 end comment 1",
							"pos":	{
								"line_nr":	13,
								"pos":	234,
								"len":	17
							}
						}, {
							"ast_type":	"Comment",
							"text":	"\u0001 end comment 2",
							"pos":	{
								"line_nr":	14,
								"pos":	252,
								"len":	17
							}
						}]
				}],
			"comments":	[{
					"ast_type":	"Comment",
					"text":	" enum comment 1",
					"pos":	{
						"line_nr":	4,
						"pos":	50,
						"len":	17
					}
				}, {
					"ast_type":	"Comment",
					"text":	"\u0001 black comment 1",
					"pos":	{
						"line_nr":	5,
						"pos":	68,
						"len":	19
					}
				}, {
					"ast_type":	"Comment",
					"text":	"\u0001 black comment 2",
					"pos":	{
						"line_nr":	6,
						"pos":	88,
						"len":	19
					}
				}],
			"attrs":	[{
					"ast_type":	"Attr",
					"name":	"attr1",
					"is_string":	false,
					"is_ctdefine":	false,
					"arg":	"",
					"is_string_arg":	false
				}, {
					"ast_type":	"Attr",
					"name":	"attr2=123",
					"is_string":	true,
					"is_ctdefine":	false,
					"arg":	"",
					"is_string_arg":	false
				}]
		}, {
			"ast_type":	"EnumDecl",
			"name":	"main.BitEnum",
			"is_pub":	false,
			"is_flag":	true,
			"is_multi_allowed":	false,
			"pos":	{
				"line_nr":	18,
				"pos":	280,
				"len":	12
			},
			"fields":	[{
					"ast_type":	"EnumField",
					"name":	"e1",
					"has_expr":	false,
					"expr":	null,
					"pos":	{
						"line_nr":	19,
						"pos":	296,
						"len":	2
					},
					"comments":	[],
					"next_comments":	[]
				}, {
					"ast_type":	"EnumField",
					"name":	"e2",
					"has_expr":	false,
					"expr":	null,
					"pos":	{
						"line_nr":	20,
						"pos":	300,
						"len":	2
					},
					"comments":	[],
					"next_comments":	[]
				}, {
					"ast_type":	"EnumField",
					"name":	"e3",
					"has_expr":	false,
					"expr":	null,
					"pos":	{
						"line_nr":	21,
						"pos":	304,
						"len":	2
					},
					"comments":	[],
					"next_comments":	[]
				}],
			"comments":	[],
			"attrs":	[{
					"ast_type":	"Attr",
					"name":	"flag",
					"is_string":	false,
					"is_ctdefine":	false,
					"arg":	"",
					"is_string_arg":	false
				}]
		}, {
			"ast_type":	"EnumDecl",
			"name":	"main.MultipleEnum",
			"is_pub":	false,
			"is_flag":	false,
			"is_multi_allowed":	true,
			"pos":	{
				"line_nr":	25,
				"pos":	335,
				"len":	17
			},
			"fields":	[{
					"ast_type":	"EnumField",
					"name":	"v1",
					"has_expr":	true,
					"expr":	{
						"ast_type":	"IntegerLiteral",
						"val":	"1",
						"pos":	{
							"line_nr":	26,
							"pos":	361,
							"len":	1
						}
					},
					"pos":	{
						"line_nr":	26,
						"pos":	356,
						"len":	2
					},
					"comments":	[],
					"next_comments":	[]
				}],
			"comments":	[],
			"attrs":	[{
					"ast_type":	"Attr",
					"name":	"_allow_multiple_values",
					"is_string":	false,
					"is_ctdefine":	false,
					"arg":	"",
					"is_string_arg":	false
				}]
		}, {
			"ast_type":	"FnDecl",
			"name":	"main.main",
			"mod":	"main",
			"is_deprecated":	false,
			"is_pub":	false,
			"is_variadic":	false,
			"is_anon":	false,
			"receiver":	{
				"ast_type":	"Field",
				"name":	"",
				"typ":	"void",
				"pos":	{
					"line_nr":	0,
					"pos":	0,
					"len":	0
				}
			},
			"receiver_pos":	{
				"line_nr":	0,
				"pos":	0,
				"len":	0
			},
			"is_method":	false,
			"method_idx":	0,
			"rec_mut":	false,
			"rec_share":	0,
			"language":	0,
			"no_body":	false,
			"is_builtin":	false,
			"is_generic":	false,
			"is_direct_arr":	false,
			"pos":	{
				"line_nr":	29,
				"pos":	366,
				"len":	9
			},
			"body_pos":	{
				"line_nr":	30,
				"pos":	379,
				"len":	3
			},
			"file":	"main.v",
			"return_type":	"void",
			"source_file":	0,
			"scope":	-1921948768,
			"attrs":	[],
			"params":	[],
			"stmts":	[{
					"ast_type":	"AssignStmt",
					"left":	[{
							"ast_type":	"Ident",
							"name":	"color",
							"mod":	"main",
							"language":	0,
							"is_mut":	true,
							"tok_kind":	35,
							"kind":	0,
							"info":	{
								"ast_type":	"IdentVar",
								"typ":	null,
								"is_mut":	true,
								"is_static":	false,
								"is_optional":	false,
								"share":	0
							},
							"pos":	{
								"line_nr":	30,
								"pos":	383,
								"len":	5
							},
							"mut_pos":	{
								"line_nr":	30,
								"pos":	379,
								"len":	3
							},
							"obj":	{
							},
							"scope":	-1921948768
						}],
					"left_types":	[],
					"right":	[{
							"ast_type":	"EnumVal",
							"enum_name":	"main.Color",
							"mod":	"",
							"val":	"black",
							"typ":	null,
							"pos":	{
								"line_nr":	31,
								"pos":	405,
								"len":	5
							}
						}],
					"right_types":	[],
					"op":	35,
					"_op":	":=",
					"is_static":	false,
					"is_simple":	false,
					"has_cross_var":	false,
					"pos":	{
						"line_nr":	30,
						"pos":	389,
						"len":	2
					}
				}, {
					"ast_type":	"AssignStmt",
					"left":	[{
							"ast_type":	"Ident",
							"name":	"color",
							"mod":	"main",
							"language":	0,
							"is_mut":	false,
							"tok_kind":	34,
							"kind":	0,
							"info":	{
								"ast_type":	"IdentVar",
								"typ":	null,
								"is_mut":	false,
								"is_static":	false,
								"is_optional":	false,
								"share":	0
							},
							"pos":	{
								"line_nr":	31,
								"pos":	405,
								"len":	5
							},
							"mut_pos":	{
								"line_nr":	31,
								"pos":	405,
								"len":	5
							},
							"obj":	{
							},
							"scope":	-1921948768
						}],
					"left_types":	[],
					"right":	[{
							"ast_type":	"EnumVal",
							"enum_name":	"",
							"mod":	"",
							"val":	"blue",
							"typ":	null,
							"pos":	{
								"line_nr":	31,
								"pos":	413,
								"len":	5
							}
						}],
					"right_types":	[],
					"op":	34,
					"_op":	"=",
					"is_static":	false,
					"is_simple":	false,
					"has_cross_var":	false,
					"pos":	{
						"line_nr":	31,
						"pos":	411,
						"len":	1
					}
				}],
			"comments":	[]
		}]
```



## Variable

### Assign

AST struct:

```v
// variable assign statement
pub struct AssignStmt {
pub:
	right         []Expr
	op            token.Kind // include: =,:=,+=,-=,*=,/= and so on,all the assigns can see vlib/token/token.v
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

example code:

```v

```

generate AST:

```json

```

### Identifier



### Literal

## Function/Method

### Function declaration

AST struct:

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

example code:

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

generate AST:

```json
"stmts": [
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
			"rec_share": 0,
			"language": 0,
			"no_body": false,
			"is_builtin": false,
			"is_generic": false,
			"is_direct_arr": false,
			"pos": {
				"line_nr": 2,
				"pos": 13,
				"len": 9
			},
			"body_pos": {
				"line_nr": 3,
				"pos": 26,
				"len": 1
			},
			"file": "main.v",
			"return_type": "void",
			"source_file": 0,
			"scope": 1388436464,
			"attrs": [],
			"params": [],
			"stmts": [
				{
					"ast_type": "AssignStmt",
					"left": [
						{
							"ast_type": "Ident",
							"name": "s",
							"mod": "main",
							"language": 0,
							"is_mut": false,
							"tok_kind": 35,
							"kind": 0,
							"info": {
								"ast_type": "IdentVar",
								"typ": null,
								"is_mut": false,
								"is_static": false,
								"is_optional": false,
								"share": 0
							},
							"pos": {
								"line_nr": 3,
								"pos": 26,
								"len": 1
							},
							"mut_pos": {
								"line_nr": 3,
								"pos": 26,
								"len": 1
							},
							"obj": {},
							"scope": 1388436464
						}
					],
					"left_types": [],
					"right": [
						{
							"ast_type": "CallExpr",
							"left": null,
							"is_method": false,
							"mod": "main",
							"name": "add",
							"language": 0,
							"scope": 1388436464,
							"args": [
								{
									"ast_type": "CallArg",
									"typ": null,
									"is_mut": false,
									"share": 0,
									"expr": {
										"ast_type": "IntegerLiteral",
										"val": "1",
										"pos": {
											"line_nr": 3,
											"pos": 35,
											"len": 1
										}
									},
									"is_tmp_autofree": false,
									"pos": {
										"line_nr": 3,
										"pos": 35,
										"len": 1
									},
									"comments": []
								},
								{
									"ast_type": "CallArg",
									"typ": null,
									"is_mut": false,
									"share": 0,
									"expr": {
										"ast_type": "IntegerLiteral",
										"val": "3",
										"pos": {
											"line_nr": 3,
											"pos": 38,
											"len": 1
										}
									},
									"is_tmp_autofree": false,
									"pos": {
										"line_nr": 3,
										"pos": 38,
										"len": 1
									},
									"comments": []
								}
							],
							"expected_arg_types": [],
							"or_block": {
								"ast_type": "OrExpr",
								"stmts": [],
								"kind": 0,
								"pos": {
									"line_nr": 4,
									"pos": 42,
									"len": 7
								}
							},
							"left_type": null,
							"receiver_type": null,
							"return_type": null,
							"should_be_skipped": false,
							"generic_type": "void",
							"generic_list_pos": {
								"line_nr": 3,
								"pos": 34,
								"len": 1
							},
							"free_receiver": false,
							"from_embed_type": null,
							"pos": {
								"line_nr": 3,
								"pos": 31,
								"len": 9
							}
						}
					],
					"right_types": [],
					"op": 35,
					"_op": ":=",
					"is_static": false,
					"is_simple": false,
					"has_cross_var": false,
					"pos": {
						"line_nr": 3,
						"pos": 28,
						"len": 2
					}
				},
				{
					"ast_type": "ExprStmt",
					"typ": null,
					"is_expr": false,
					"expr": {
						"ast_type": "CallExpr",
						"left": null,
						"is_method": false,
						"mod": "main",
						"name": "println",
						"language": 0,
						"scope": 1388436464,
						"args": [
							{
								"ast_type": "CallArg",
								"typ": null,
								"is_mut": false,
								"share": 0,
								"expr": {
									"ast_type": "Ident",
									"name": "s",
									"mod": "main",
									"language": 0,
									"is_mut": false,
									"tok_kind": 49,
									"kind": 0,
									"info": {
										"ast_type": "IdentVar",
										"typ": null,
										"is_mut": false,
										"is_static": false,
										"is_optional": false,
										"share": 0
									},
									"pos": {
										"line_nr": 4,
										"pos": 50,
										"len": 1
									},
									"mut_pos": {
										"line_nr": 4,
										"pos": 50,
										"len": 1
									},
									"obj": {},
									"scope": 1388436464
								},
								"is_tmp_autofree": false,
								"pos": {
									"line_nr": 4,
									"pos": 50,
									"len": 1
								},
								"comments": []
							}
						],
						"expected_arg_types": [],
						"or_block": {
							"ast_type": "OrExpr",
							"stmts": [],
							"kind": 0,
							"pos": {
								"line_nr": 5,
								"pos": 53,
								"len": 1
							}
						},
						"left_type": null,
						"receiver_type": null,
						"return_type": null,
						"should_be_skipped": false,
						"generic_type": "void",
						"generic_list_pos": {
							"line_nr": 4,
							"pos": 49,
							"len": 1
						},
						"free_receiver": false,
						"from_embed_type": null,
						"pos": {
							"line_nr": 4,
							"pos": 42,
							"len": 10
						}
					},
					"pos": {
						"line_nr": 4,
						"pos": 42,
						"len": 10
					},
					"comments": []
				}
			],
			"comments": []
		},
		{
			"ast_type": "FnDecl",
			"name": "main.add",
			"mod": "main",
			"is_deprecated": false,
			"is_pub": true,
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
			"rec_share": 0,
			"language": 0,
			"no_body": false,
			"is_builtin": false,
			"is_generic": false,
			"is_direct_arr": false,
			"pos": {
				"line_nr": 7,
				"pos": 56,
				"len": 28
			},
			"body_pos": {
				"line_nr": 8,
				"pos": 88,
				"len": 6
			},
			"file": "main.v",
			"return_type": "int",
			"source_file": 0,
			"scope": 1388439776,
			"attrs": [],
			"params": [
				{
					"ast_type": "Param",
					"name": "x",
					"typ": "int",
					"is_mut": false
				},
				{
					"ast_type": "Param",
					"name": "y",
					"typ": "int",
					"is_mut": false
				}
			],
			"stmts": [
				{
					"ast_type": "Return",
					"exprs": [
						{
							"ast_type": "InfixExpr",
							"op": 7,
							"_op": "+",
							"left": {
								"ast_type": "Ident",
								"name": "x",
								"mod": "main",
								"language": 0,
								"is_mut": false,
								"tok_kind": 7,
								"kind": 0,
								"info": {
									"ast_type": "IdentVar",
									"typ": null,
									"is_mut": false,
									"is_static": false,
									"is_optional": false,
									"share": 0
								},
								"pos": {
									"line_nr": 8,
									"pos": 95,
									"len": 1
								},
								"mut_pos": {
									"line_nr": 8,
									"pos": 95,
									"len": 1
								},
								"obj": {},
								"scope": 1388439776
							},
							"left_type": null,
							"right": {
								"ast_type": "Ident",
								"name": "y",
								"mod": "main",
								"language": 0,
								"is_mut": false,
								"tok_kind": 47,
								"kind": 0,
								"info": {
									"ast_type": "IdentVar",
									"typ": null,
									"is_mut": false,
									"is_static": false,
									"is_optional": false,
									"share": 0
								},
								"pos": {
									"line_nr": 8,
									"pos": 99,
									"len": 1
								},
								"mut_pos": {
									"line_nr": 8,
									"pos": 99,
									"len": 1
								},
								"obj": {},
								"scope": 1388439776
							},
							"right_type": null,
							"auto_locked": "",
							"pos": {
								"line_nr": 8,
								"pos": 97,
								"len": 1
							}
						}
					],
					"types": [],
					"pos": {
						"line_nr": 8,
						"pos": 88,
						"len": 12
					}
				}
			],
			"comments": []
		}
	]
```

### Function call



### Anonymous function

AST struct:

```v
//anonymous function
pub struct AnonFn {
pub:
	decl FnDecl
pub mut:
	typ  table.Type
}
```

example code:

```v
module main

fn main() {
	f1 := fn (x int, y int) int {
		return x + y
	}
	f1(1,3)
}

```

generate AST:

```json
			"stmts": [
				{
					"ast_type": "AssignStmt",
					"left": [
						{
							"ast_type": "Ident",
							"name": "f1",
							"mod": "main",
							"language": 0,
							"is_mut": false,
							"tok_kind": 35,
							"kind": 0,
							"info": {
								"ast_type": "IdentVar",
								"typ": null,
								"is_mut": false,
								"is_static": false,
								"is_optional": false,
								"share": 0
							},
							"pos": {
								"line_nr": 3,
								"pos": 26,
								"len": 2
							},
							"mut_pos": {
								"line_nr": 3,
								"pos": 26,
								"len": 2
							},
							"obj": {},
							"scope": 1388597776
						}
					],
					"left_types": [],
					"right": [
						{
							"ast_type": "AnonFn",
							"decl": {
								"ast_type": "FnDecl",
								"name": "anon_75_7_7_7",
								"mod": "main",
								"is_deprecated": false,
								"is_pub": false,
								"is_variadic": false,
								"is_anon": true,
								"receiver": {
									"ast_type": "Field",
									"typ": null,
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
								"rec_share": 0,
								"language": 0,
								"no_body": false,
								"is_builtin": false,
								"is_generic": false,
								"is_direct_arr": false,
								"pos": {
									"line_nr": 3,
									"pos": 32,
									"len": 2
								},
								"body_pos": {
									"line_nr": 0,
									"pos": 0,
									"len": 0
								},
								"file": "/Users/zhijiayou02/v/vprojects/tony_test/main.v",
								"return_type": "int",
								"source_file": 0,
								"scope": 1388597776,
								"attrs": [],
								"params": [
									{
										"ast_type": "Param",
										"name": "x",
										"typ": "int",
										"is_mut": false
									},
									{
										"ast_type": "Param",
										"name": "y",
										"typ": "int",
										"is_mut": false
									}
								],
								"stmts": [
									{
										"ast_type": "Return",
										"exprs": [
											{
												"ast_type": "InfixExpr",
												"op": 7,
												"_op": "+",
												"left": {
													"ast_type": "Ident",
													"name": "x",
													"mod": "main",
													"language": 0,
													"is_mut": false,
													"tok_kind": 7,
													"kind": 0,
													"info": {
														"ast_type": "IdentVar",
														"typ": null,
														"is_mut": false,
														"is_static": false,
														"is_optional": false,
														"share": 0
													},
													"pos": {
														"line_nr": 4,
														"pos": 65,
														"len": 1
													},
													"mut_pos": {
														"line_nr": 4,
														"pos": 65,
														"len": 1
													},
													"obj": {},
													"scope": 1388598896
												},
												"left_type": null,
												"right": {
													"ast_type": "Ident",
													"name": "y",
													"mod": "main",
													"language": 0,
													"is_mut": false,
													"tok_kind": 47,
													"kind": 0,
													"info": {
														"ast_type": "IdentVar",
														"typ": null,
														"is_mut": false,
														"is_static": false,
														"is_optional": false,
														"share": 0
													},
													"pos": {
														"line_nr": 4,
														"pos": 69,
														"len": 1
													},
													"mut_pos": {
														"line_nr": 4,
														"pos": 69,
														"len": 1
													},
													"obj": {},
													"scope": 1388598896
												},
												"right_type": null,
												"auto_locked": "",
												"pos": {
													"line_nr": 4,
													"pos": 67,
													"len": 1
												}
											}
										],
										"types": [],
										"pos": {
											"line_nr": 4,
											"pos": 58,
											"len": 12
										}
									}
								],
								"comments": []
							},
							"typ": "anon_75_7_7_7"
						}
					],
					"right_types": [],
					"op": 35,
					"_op": ":=",
					"is_static": false,
					"is_simple": false,
					"has_cross_var": false,
					"pos": {
						"line_nr": 3,
						"pos": 29,
						"len": 2
					}
				},
				{
					"ast_type": "ExprStmt",
					"typ": null,
					"is_expr": false,
					"expr": {
						"ast_type": "CallExpr",
						"left": null,
						"is_method": false,
						"mod": "main",
						"name": "f1",
						"language": 0,
						"scope": 1388597776,
						"args": [
							{
								"ast_type": "CallArg",
								"typ": null,
								"is_mut": false,
								"share": 0,
								"expr": {
									"ast_type": "IntegerLiteral",
									"val": "1",
									"pos": {
										"line_nr": 6,
										"pos": 78,
										"len": 1
									}
								},
								"is_tmp_autofree": false,
								"pos": {
									"line_nr": 6,
									"pos": 78,
									"len": 1
								},
								"comments": []
							},
							{
								"ast_type": "CallArg",
								"typ": null,
								"is_mut": false,
								"share": 0,
								"expr": {
									"ast_type": "IntegerLiteral",
									"val": "3",
									"pos": {
										"line_nr": 6,
										"pos": 80,
										"len": 1
									}
								},
								"is_tmp_autofree": false,
								"pos": {
									"line_nr": 6,
									"pos": 80,
									"len": 1
								},
								"comments": []
							}
						],
						"expected_arg_types": [],
						"or_block": {
							"ast_type": "OrExpr",
							"stmts": [],
							"kind": 0,
							"pos": {
								"line_nr": 7,
								"pos": 83,
								"len": 1
							}
						},
						"left_type": null,
						"receiver_type": null,
						"return_type": null,
						"should_be_skipped": false,
						"generic_type": "void",
						"generic_list_pos": {
							"line_nr": 6,
							"pos": 77,
							"len": 1
						},
						"free_receiver": false,
						"from_embed_type": null,
						"pos": {
							"line_nr": 6,
							"pos": 75,
							"len": 7
						}
					},
					"pos": {
						"line_nr": 6,
						"pos": 75,
						"len": 7
					},
					"comments": []
				}
			],
			"comments": []
		}
	]
```



## Struct

AST struct:

```v

```

example code:

```v

```

generate AST:

```json

```



## Interface

AST struct:

```v

```

example code:

```v

```

generate AST:

```json

```



## Type

### Alias Type 

AST struct:

```v

```

example code:

```v

```

generate AST:

```json

```



### Function Type

AST struct:

```v

```

example code:

```v

```

generate AST:

```json

```



### SumType

AST struct:

```v

```

example code:

```v

```

generate AST:

```json

```



### FlowControl

#### if



#### match



#### for



## Generic

### Generic Struct



### Generic Function



## common

### Comment

AST struct:

```v

```

example code:

```v

```

generate AST:

```json

```



### Position

AST struct:

```v

```

example code:

```v

```

generate AST:

```json

```

