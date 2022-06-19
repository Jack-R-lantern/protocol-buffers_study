# Protocol Buffers Version 3 Language Sepcification
Protocol Buffers Version 3의 구문은 [EBNF](https://en.wikipedia.org/wiki/Extended_Backus%E2%80%93Naur_form)를 사용하여 지정됨.
```
| alternation
() grouping
[] option (zero or one time)
{} repetition (ant number of times)
```

## 어휘 요소
### 문자 및 숫자
```
letter       = "A" ... "Z" | "a" ... "z"
decimalDigit = "0" ... "9"
octalDigit   = "0" ... "7"
hexDigit     = "0" ... "9" | "A" ... "F" | "a" ... "f"
```

### 구분자
```
ident       = letter { letter | decimalDigit | "_" }
fullIdent   = idnet  { "." ident }
messageName = ident
enumName    = ident
fieldName   = ident
oneofName   = ident
mapName     = ident
servieName  = ident
rpcName     = ident
messageType = [ "." ] { ident "." } messageName
enumType    = [ "." ] { ident "." } enumName
```

### 정수형 리터럴
```
intLit     = decimalLit | octalLit | hexLit
decimalLit = ("1" ... "9") { decimalDigit }
octalLit   = "0" { octalDigit }
hexLit     = "0" ( "x" | "X" ) hexDigit { hexDigit }
```

### 부동 소수점 리터럴
```
floatLit = ( decimals "." [ decimals ] [ exponent ] | decimals exponent | "."decimals [ exponent ]) | "inf" | "nan"
decimals = decimalDigit { decimalDigit }
exponent = ( "e" | "E" ) [ "+" | "-" ] decimals
```

### 부울
```
boolLit = "true" | "false"
```

### 문자열 리터럴
```
strLit     = ("'" { charValue } "'") | ('"' { charValue } '"')
hexEscape  = '\' ( "x" | "X" ) hexDigit hexDigit
octEscape  = '\' octalDigit octalDigit octalDigit
charEscape = '\' ( "a" | "b" | "f" | "n" | "r" | "t" | "v" | '\' | "'" | '"')
```

### 빈 구문
```
emptyStatement = ";"
```

### 상수
```
constant = fullIndent | ( [ "-" | "+" ] intLit ) | ( [ "-" | "+" ] floatLit ) | strLit | boolLit
```

## Syntax
`syntax`문은 `protobuf`버전을 정의하는데 사용.
```
syntax = "syntax" "=" ("'" "proto3" "'" | '"' "proto3" '"') ";"
```

## Import Statement
`import`문은 다른 `.proto`의 정의를 가져오는데 사용.
```
import = "import" [ "weak" | "public" ] strLit ";"
```

## Package
패키지 구분자는 프로토콜 메세지 타입간의 충돌을 예방하는데 사용.
```
package = "package" fullIndent ";"
```

## Option
옵션은 proto파일, 메세지, enum, 서비스에 사용할 수 있음.

```
option = "option" optionName "=" constant ";"
optionName = ( ident | "(" fullIdent ")") { "." ident }
```

## Fields
필드는 `protocol buffers`의 기본 요소, 필드는 일반 필드, 필드의 하나, 또는 맵 필드임. \
필드는 타입과 숫자를 가짐.
```
type        = "double" | "float" | "int32" | "int64" | "uint32" | "uint64" | "sint32" | "sint64" | "fixed32" | "fixed64" | "sfixed32" | "sfixed64" | "bool" | "string" | "bytes" | messageType | enumType
filedNumber = intLit;
```

### Normal field
각 필드는 타입, 이름 그리고 필드 넘버를 가지고 필드 옵션을 가질 수 있음.
```
field        = [ "repeated" ] type fieldName "=" fieldNumber [ "[" fieldOptions "]" ] ";"
fieldOptions = fieldOption { "," fieldOption }
fieldOption  = optionName "=" constant
```

### Oneof and oneof field
`onefo`는 `oneof`필드와 oneof`이름으로 구성됨.
```
oneof = "oneof" oneofName "{" { option | oneofField | emptyStatement } "}"
oneofField = type fieldName "=" fieldNumber [ "[" fieldOptions "]" ]";"
```

### Map field
맵 필드에는 키 유형, 값 유형, 이름 및 필드 번호가 있음.
```
mapField = "map" "<" keyType "," type ">" mapName "=" fieldNumber [ "[" fieldOptions "]" ] ";"
keyType = "int32" | "int64" | "uint32" | "uint64" | "sint32" | "sint64" | "fixed32" | "fixed64" | "sfixed32" | "sfixed64" | "bool" | "string"
```

## Reserved
`Reserved`는 메시지에서 사용할 수 없는 필드 번호 또는 필드 이름의 범위를 선언.
```
reserved      = "reserved" ( ranges | strFieldNames ) ";"
ranges        = range { "," range }
range         = intLit [ "to" (intLit | "max" )]
strFieldNames = strFieldName { "," strFieldName }
strFieldName  = "'" fieldName "'" | '"' fieldName '"'
```

## Top Level definitions
### Enum definition
`enum definition`은 이름과 몸체로 구성됨. \
몸체는 옵션과 필드를 가질 수 있음. \
`Enum definitions`는 반드시 0 부터 시작해야함.
```
enum            = "enum" enumName enumBody
enumBody        = "{" { option | enumField | emptyStatement } "}"
enumField = ident "=" [ "-" ] intLit [ "[" enumValueOption { "," enumValueOption } "]" ]";"
enumValueOption = optionName "=" constant
```

### Message definition
`message`는 이름과 몸체로 구성됨. \
몸체는 필드를 가지고 중첩된 열거형, 중첩된 메시지 정의, oneofs, map fields, 예약어 구문을 가짐
```
message     = "message" messageName messageBody
messageBody = "{" { field | enum | message | option | oneof | mapField | reserved | emptyStatement } "}"
```

### Service definition
```
service = "service" serviceName "{" { option | rpc | emptyStatement } "}"
rpc     = "rpc" rpcName "(" [ "stream" ] messageType ")" "returns" "(" [ "stream" ] messageType ")" (( "{" { option | emptyStatement } "}" ) | ";" )
```

## Proto file
```
proto       = syntax { import | package | option | topLevelDef | emptyStatement }
topLevelDef = message | enum | service
```
**example**
```
syntax = "proto3";
import public "other.proto";
option java_package = "com.example.foo";
enum EnumAllowingAlias {
	option allow_alias = true;
	UNKNOWN = 0;
	STARTED = 1;
	RUNNING = 2 [(custom_option) = "hello world"];
}
message Outer {
	option (my_option).a = true;
	message Inner {		// Level 2
		int64 ival = 1;
	}
	repeated Inner inner_message = 2;
	EnumAllowingAlias enum_field = 3;
	map<int32, string> my_map	 = 4;
}
```