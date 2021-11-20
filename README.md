# instruction_research
### instruction research about getopt, getopts, sed, awk
---
---
1. **getopt**

쉘스크립트에서 실행 옵션을 부여하는 함수이다. getopts보다 더 긴 옵션을 넣을 때 사용한다.

다양한 입력값이 존재할 경우 사용자와 개발자의 편의를 보장할 수 있다.

***getopt -o|--options shortopts와 [-l|--longoptions longopts] [-n|--name progname][--] parameters***

shortopts = 옵션을 정의하는 문자, getopts의 option_string과 같음

longopts = 긴 옵션을 정의하는 문자 --diff와 같이 한글자로 정의되지 않는 긴 옵션을 정의, 글자 수가 정의되지 않아서 ,(콤마)로 구분한다.

progname = 오류 발생시 리포팅할 프로그램 명칭(현재 쉘 스크립트 파일명 = $0)

parameters = 옵션에 해당하는 실제 명령 구문(보통은 모든 파라미터를 뜻하는 $@ 사용)

getopt -o d:u:f:h -l diffs:,unit:,format:,help -n $0 -- "$@"

-o에서 짧은 옵션은 d:u:f:h로 설정됐고 -l에는 그와 같은 뜻으로 쓰이는 긴 옵션인 diffs:,unit:,format:,help가 들어간다.

```sh
#!/bin/bash
print_try(){ # 옵션값이 하나도 없을 때 실행되는 명령어
    echo "Try 'datefmt2.sh -h|--help' for more information"
    exit 1
}

print_help(){ # 옵션값을 어떻게 넣을지 알려주는 명령어
    echo "usage: datefmt2.sh -d|diffs <diffs> -u|--unit <unit> [-f|--format format]"
    exit 1
}

options="$(getopt -o d:u:f:h -l diffs:,unit:,format:,help -n $0 -- "$@")" # $@처럼 쌍따음표("")를 안넣으면 getopt에 옵션값이 들어가버림
eval set -- $options
# eval 문자열을 명령어처럼 인식되게 함
# set 문자열을 띄어쓰기 기준으로 쪼개서 변수 $1,$2...에 저장
# set -- 는 -a,-b와 같은 옵션처럼 생긴 문자열도 읽어서 변수 $1,$2...로 저장

while true
do
    case $1 in
        -d|--diffs)
            D=$2 # 옵션 $2의 옵션값을 변수 D에 저장
            shift 2;; # $3 의 옵션값을 $1으로 옮기도록 2칸 이동
        -u|--unit)
            U=$2 # 옵션 $2의 옵션값을 변수 U에 저장
            shift 2;; # $3 의 옵션값을 $1으로 옮기도록 2칸 이동
        -f|--format)
            F=$2 # 옵션 $2의 옵션값을 변수 F에 저장
            shift 2;; # $3 의 옵션값을 $1으로 옮기도록 2칸 이동
        -h|--help)
            print_help;;
        --) # $@에서 옵션들의 끝이라는 걸 알기 위해 --가 끝에 붙음, 그래서 --에서 while이 끝나도록 설정함
            break;;
        *)
            print_try;;
    esac
done

RET=`date $F --date="$D $U"` # ``은 ''이 아니라 자판1 왼쪽에 있는 자판임
echo "$RET"
```
입력예1

*한글자 옵션인 -d와, 긴 옵션이름인 --unit, --format을 적고 옵션값을 각각 넣었음*

`
./datefmt2.sh -d 3 --unit month --format +%Y-%m-%d
`

출력예

`
2022-02-20
`

입력예2

*한글자 옵션인 -u와 -f, 긴 옵션이름인 --diffs을 적고 옵션값을 각각 넣었음*

`
./datefmt2.sh --diffs 3 -u month -f +%Y-%m-%d
`

출력예

`
2021-02-20
`

이처럼 옵션이름을 한글자나 한글자 이상의 옵션이름 중 어느것으로 적어도 똑같이 실행됨

--------
2. **getopts**

쉘 스크립트에서 실행 옵션을 부여하는 함수이다.

***getopts (option_string) (varname)***

option_string = 옵션을 정의하는 문자, 뒤에 콜론(:)이 있으면 옵션값을 받는 다는 의미(d:u:f:h )

varname = 옵션 명(d,u,f,h)을 받을 변수, opt 변수에는 옵션의 이름(d,u,f,h)이 들어감, OPTARG 변수에는 실제 옵션의 값(콜론':'의 값)이 세팅됨!

while getopts d:u:f:h opt 라면 opt 변수명에 d,u,f,h의 옵션값이 차례대로 들어감 

```sh
#!/bin/bash
print_try(){ # 옵션값이 하나도 없을 때 실행되는 명령어
    echo "Try 'datefmt.sh -h' for more information"
    exit 1
}

print_help(){ # 옵션값을 어떻게 넣을지 알려주는 명령어
    echo "usage: datefmt.sh -d <diffs> -u <unit> [-f format]"
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

---

3. **sed**



---

4. **awk**


