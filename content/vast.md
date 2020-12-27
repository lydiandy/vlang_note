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



## Function/Method

AST struct:

```v

```

example code:

```v

```

generate AST:

```json

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



## common AST

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

