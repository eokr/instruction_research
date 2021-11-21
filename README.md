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
2022-02-20
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

비상호대화형 스트림 편집기

텍스트 파일 내용 출력, 수정, 추가, 제거 등을 할 수 있으며 반복된 텍스트 처리를 위해 사용 가능

***sed [option] 'INSTRUCTIONS' INPUT_FILE***

***sed [option] SED_SCRIPT INPUT_FILE***

|옵션|설명|
|-|-|
|-n|입력줄 자동 출력을 제한하기 위해 사용한다. 검색한 내용만 출력하고 싶을때 쓴다.(자주 씀)|
|-e|하나 이상의 지시어를 사용할 경우 사용한다.|
|-i|직접적 파일 편집을 위해 사용한다.|
|-f|sed 지시어 스크립트 파일을 사용하기 위해 사용한다.|

sed 명령어를 알아보기 위해 사용한 .txt파일(a, b, c, d, ..., cz까지의 알파벳이 있음)

<img src="https://user-images.githubusercontent.com/93603660/142735416-8f0b9516-c7e8-40e5-9162-ace169306145.png" width="300" hight="400">

**🔍sed 명령어에서 출력 할때 -n 옵션을 많이 쓰는 이유🔍**

sed로 첫 번째 행을 출력할 때 (-n있음)

`
sed -n '1p' aaa.txt
`

결과 *검색한 내용만 출력된다.*

`
a
`

sed로 첫 번째 행을 출력할 때 (-n없음)

`
sed '1p' aaa.txt
`

결과 *검색한 내용 외에 원래 파일 내용까지 출력된다.*

```sh
a
a
b
c
d
e
f
g
h
i
j
k
l
n
m
o
p
q
r
s
t
u
v
w
x
y
z
aa
....
cz
```

**특정 범위의 파일 내용 출력하기**
```sh
sed -n '1p' aaa.txt # aaa.txt파일 에서 첫 번째 행을 출력한다.
sed -n '1,3p' aaa.txt # aaa.txt파일 에서 1~3 행을 출력한다.
sed -n '98,$p' aaa.txt # aaa.txt파일 에서 98번째 행에서 파일 끝 행 까지 출력한다. $는 마지막을 의미한다.
sed -n -e '1p' -e '100,$p' aaa.txt # aaa.txt파일의 첫 번째 행과 100번째 행에서 파일 끝 행 까지 출력한다. -e는 여러 개의 편집 명령을 실행할 때 사용하는 옵션이다.
```
p는 출력을 의미한다.

결과

`
a
`
```
a
b
c
```
```
ct
cu
cv
cw
cx
cy
cz
```
```
a
cv
cw
cx
cy
cz
```
**특정 단어의 행을 출력하기**
```sh
sed -n '/z/p' aaa.txt # aaa.txt파일에서 z를 포함하는 행을 출력한다.
sed -n '/^z/p' aaa.txt # aaa.txt파일에서 z로 시작하는 행을 출력한다. ^는 시작을 의미한다.
```
결과
```sh
a
cv
cw
cx
cy
cz
```

`
z
`

**특정 행 삭제하기**
```sh
sed '5,$d' aaa.txt # 5번째 행부터 마지막 행까지 삭제하고 출력한다. d는 삭제를 의미한다.
```
결과
```sh
a
b
c
d
```
**특정 단어 치환하기**
```sh
sed -n 's/z/a/gp' aaa.txt
# z라는 단어를 a로 치환한 내용을 출력한다. s는 문자열을 치환한다는 의미이고 g는 치환이 행 전체를 대상으로 이루어 진다는 뜻이다.
```
결과
```sh
a
aa
ba
ca
```

---

4. **awk**

awk는 1970년대 만들어진 데이터 처리를 위한 프로그래밍 언어이다.

패턴 검색, 산술 연산 외에도 변수 선언, 배열, 조건문, 반복문 등 다양한 기능을 가지고 있는 프로그래밍 언어이다.

*awk [OPTION] 'PROGRAM' INPUT_FILE*

|옵션|설명|
|-|-|
|-f|awk 프로그램 파일을 사용하기 위해 사용한다. (예: -f 프로그램 파일명)|
|-F|입력 필드 구분자를 정의하기 위해 사용한다. 기본 필드 구분자는 스페이스이다. (예: -F: or -F^)|
|-v|변수를 선언하기 위해 사용한다. (예: -v 변수명=값)|

|변수|설명|
|-|-|
|FS|입력 필드 구분자 (기본 스페이스)|
|RS|입력 레코드 구분자 (기본 줄바꿈)|
|OFS|출력 필드 구분자 (기본 스페이스)|
|ORS|출력 필드 그룹 구분자 (기본 줄바꿈)|
|NR|입력된 총 레코드(Record)수 (레코드는 입력 줄로 생각하면 이해하기 쉬움)|
|NF|입력된 레코드의 총 필드 수|
|$1|첫 번째 필드값|
|$n|n번째 필드값|

프로그램 구조는 BEGIN, BODY, END로 이루어져 있다.

|구조|설명|
|-|-|
|BEGIN|변수 선언과 레코드를 읽기 전 실행되어야 하는 내용 정의|
|BODY|패턴에 일치하는 모든 입력 레코드를 ACTION에 의해 처리|
|END|awk 프로그램 종료 전 처리되어야 하는 내용 정의|

awk구조를 알아보기 위한 file.txt파일

![image](https://user-images.githubusercontent.com/93603660/142754520-88165180-ef30-44bc-ae82-28a523bed2d0.png)

**문자 구분해서 출력하기**

`awk '{print $1}' file.txt`

결과 *입력 필드 기본 구분자인 스페이스에 의해 첫 번째 필드 $1이 출력되었다.*

```
aaa
bbb
ccc
ddd
```

**정규식을 사용하여 문자 출력하기**

`awk '/bbb/{print $1,$2}' file.txt`

결과 *정규식 패턴 /bbb/에 해당하는 줄의 첫 번째 필드$1과 두 번째 필드$2를 출력한다.*

`bbb 2`

**조건 표현식을 이용하여 문자 출력하기**

`awk '$2 > 1' file.txt`

결과 *두 번째 필드$2의 값이 1보다 큰 줄만 출력한다.*

```
bbb 2
ccc 3
ddd 4
```


---
---
