# instruction_research
### instruction research about getopt, getopts, sed, awk
---
1. **getopt**

쉘스크립트에서 실행 옵션을 부여하는 함수이다.

다양한 입력값이 존재할 경우 사용자와 개발자의 편의를 보장할 수 있다.

스크립트를 보다 체계적으로 관리할 수 있다.
```sh
#!/bin/bash
## 도움말 출력하는 함수
help(){
  echo "splt [OPTIONS] FILE"
    echo" -h  도움말 출력."
    echo" -a ARG  인자를 받는 opt."
    echo" -b ARG  인자를 받는 opt2."
    exit 0
}
while getopts "a:b:h" opt
do
case $opt in
a) arg_a=$OPTARG
echo "Arg A: $arg_a"
;;
b) arg_b=$OPTARG
echo "Arg B: $arg_b"
echo "$arg_b"
;;
h) help;;
?) help;;
esac
done
```


2. **getopts**

getopts (option_string) (varname)

option_string = 옵션을 정의하는 문자, 뒤에 콜론(:)이 있으면 옵션값을 받는 다는 의미(d:u:f:h )

varname = 옵션 명(d,u,f)을 받을 변수
