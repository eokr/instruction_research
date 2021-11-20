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

*getopts (option_string) (varname)*

option_string = 옵션을 정의하는 문자, 뒤에 콜론(:)이 있으면 옵션값을 받는 다는 의미(d:u:f:h )

varname = 옵션 명(d,u,f,h)을 받을 변수, opt 변수에는 옵션의 이름(d,u,f,h)이 들어감, OPTARG 변수에는 실제 옵션의 값(콜론':'의 값)이 세팅됨!

while getopts d:u:f:h opt 라면 opt 변수명에 d,u,f,h의 옵션값이 차례대로 들어감 

```sh
#!/bin/bash
print_try(){ # 옵션값이 하나도 없을 때 실행되는 명령어
    echo "Try 'datefmt2.sh -h' for more information"
    exit 1
}

print_help(){ # 옵션값을 어떻게 넣을지 알려주는 명령어
    echo "usage: datefmt2.sh -d <diffs> -u <unit> [-f format]"
    exit 1
}

while getopts d:u:f:h opt
do
    echo "opt=$opt, OPTARG=$OPTARG"
    case $opt in
        d)
            D=$OPTARG;;
        u)
            U=$OPTARG;;
        f)
            F=$OPTARG;;
        h)
            print_help;;
        *)
            print_try;;
    esac
done

if[ -z $F ]; then # F값이 없을 때 실행
    F="+%m-%d" # F에 기본값을 넣어줌
fi

RET=`date $F --date="$D $U"` # ``은 ''이 아니라 자판1 왼쪽에 있는 자판임
echo "$RET"
```
입력예1

*-h 다음에 -x라는 정의되지 않은 옵션값을 넣었음*
```sh
./datefmt.sh -d 3 -u day -f +%m-%d -h -x
```
결과

*-h 실행하고 종료함, -x는 읽지 않음*
```sh
opt=d, OPTARG=3
opt=u, OPTARG=day
opt=f, OPTARG=+%m-%d
opt=h, OPTARG=
usage: datefmt2.sh -d <diffs> -u <unit> [-f format]
```
입력예2

*while문이 f를 읽고 끝나고 RET변수를 echo로 실행하도록 -f의 옵션값 까지만 넣었음*
```sh
./datefmt.sh -d 3 -u day -f +%Y-%m-%d
```
결과

*RET변수 echo함*
```sh
opt=d, OPTARG=3
opt=u, OPTARG=day
opt=f, OPTARG=+%Y-%m-%d
2021-11-23
```
입력예3

*F에 기본값이 들어가도록 -f옵션을 입력하지 않음*
```sh
./datefmt.sh -d 3 -u day
```
결과

*F에 기본값대로 들어감*
```sh
opt=d, OPTARG=3
opt=u, OPTARG=day
11-23
```
