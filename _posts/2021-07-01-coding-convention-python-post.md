---
title: "python coding convention"
date: 2021-07-01 01:01:01 -0400
categories: python coding-convention
---

# Python 언어에서 가장 중요한 점?
* 가독성!!이 제일 중요하다. 속도, 메모리 효율을 높이고 싶다면 다른 언어로 가는 것이 맞다.
* 만들기 전에 검색하자. 우리가 생각하는 건 누군가 이미 만들었다.

# 기본 컨벤션
## 목표점 
* 컨벤션을 지켜서 누가 보아도 눈이 편하고 (가독성)
* 누가 작성한지 모르게 개성이 없으며 (통일)
* 선진화 되고 효율적 코드 작성
## 가이드
* 영문 : https://www.python.org/dev/peps/pep-0008/
* 한글 : https://wikidocs.net/7896
* zen of python : https://wikidocs.net/7907

## os.path 보다 pathlib 을 쓰자
* 연산자를 사용할 수 있어서 더욱 직관적이다
* 를 빠트려 코드에서 에러가 생길 수 없다
* 참고 : https://docs.python.org/ko/3/library/pathlib.html

```python
No:
    path = root + f'/{company}/' + f'/{department}/'  + f'/{age}/' 
    path = os.path.join(root, company, department, str(age))
Yes:
    path = Path(root) / company / department / str(age)
```

## open보다 pathlib의 read write를 쓰자
* open을 사용하여 file descriptor를 종료 하기 애매할수 있는 문제가 없고 직관적이다
* with를 사용해도 되지만 이 방법이 더 좋은 거 같음.

```python
No:
    with open(fpath, 'r') as f:
        json_data = json.load(f)
    with open(fpath, 'w') as f:
        json.dump(json_data, f)
    with open(fpath, 'rb') as f:
        byte_data = f
Yes:
    json_data = json.loads(Path(fpath).read_text())
    Path(fpath).write_text(json.dumps(json_data))
    byte_data = Path(fpath).read_bytes()
```

## format이나 %s보다 f"" f-string을 쓰자
* f-string이 제일 빠르다
* flynt 를 이용해서 자동으로 변경할 수 있음 
  - https://pypi.org/project/flynt/
* 출처: https://bluese05.tistory.com/70

```python
No:
    test = "test"
    print("this is %s" % test)
    print("this is {}".format(test))
 
    import datetime
    date = datetime.datetime.now()
    '{} is on a {}'.format(date.strftime("%Y-%m-%d"), date.strftime("%A") )
     >>> '2019-05-11 is on a Saturday'
 
Yes:
    print(f"this is {test}")
 
    import datetime
    date = datetime.datetime.now()
    f'{date:%Y-%m-%d} is on a {date:%A}'
     >>> '2019-05-11 is on a Saturday'
```

## print 보다 pprint를 쓰자 (복잡한 log는 pformat 쓰자)
* 복잡한 포멧에서 출력을 보기 편하게 함.
* 출처 : https://www.programmersought.com/article/2697394940/

```python
No:
    data = ("test", [1, 2, 3,'test', 4, 5], "This is a string!",
            {'age':23, 'gender':'F'})
    print(data)
    ('test', [1, 2, 3, 'test', 4, 5], 'This is a string!', {'gender': 'F', 'age': 23})
 
    logger.info("{'gender': 'F', 'age': 23}")
Yes:
    import pprint
    data = ("test", [1, 2, 3,'test', 4, 5], "This is a string!",
            {'age':23, 'gender':'F'})
    pprint.pprint(data)
    ('test',
     [1, 2, 3, 'test', 4, 5],
     'This is a string!',
     {'age': 23, 'gender': 'F'})
 
    from pprint import pformat
    logger.info(pformat("{'gender': 'F', 'age': 23}"))
```

## for loop 보다는 map, filter를 쓰자 
* 속도가 차이가 남.
```python
No:
    def add_1(n):
          return n + 1
 
    target = [1, 2, 3, 4, 5]
    result = []
    for value in target:
         result.append(add_1(value))
 
    print(result) # 출력결과 : [2, 3, 4, 5, 6]
 
Yes:
    def add_1(n):
          return n + 1
    target = [1, 2, 3, 4, 5]
    result = map(add_1, target)
    print(list(result))
 
    target = [1, 2, 3, 4, 5]
    result = map(lambda x : x+1, target)
    print(list(result))
```

## if elif else 보다는 return continue break를 사용하여 들여쓰기를 아끼자
* tab이 한없이 길어 지는 걸 피하고 코드 흐림이 더 쉬워짐.

```python
No:
    def test():
        do_something()
        if condition == True:
            aaa()
        else:
            bbb()
 
Yes:
    def test():
        do_something()
        if condition == True:
            aaa()
            return
        bbb()
 
예외 :
    def test():
        do_something()
        if condition == True:
            aaa()
        else:
            bbb()
        do_end_something()
```

## getopt보다 argparse를 쓰자
* for 루프 같은 것을 돌지 않아도 됨
* help를 따로 만들지 않고 직강제적으로 생성하게 할 수 있음.
* https://www.python.org/dev/peps/pep-0389/

```python
No:
    import getopt
    opts, _ = getopt.getopt(sys.argv[1:], "p:o", ["input_path=", "output_path="])
 
    for opt, arg in opts:
        if opt == '-h':
            aicommon.Utils.usage()
            sys.exit()
        elif opt in ("-m", "--input_path"):
            input_path = arg
        elif opt in ("-t", "--output_path"):
            output_path = arg
 
Yes:
    import argparse
    parser = argparse.ArgumentParser(prog="path config",
                                        description="path config", add_help=True)
    parser.add_argument('-i', '--INPUT', help='input path.', required=True)
    parser.add_argument('-o', '--OUTPUTP', help='output path.', required=True)
    args = parser.parse_args()
    input_path = args.INPUT
    output_path = args.OUTPUTP
```

## 문자열이 개행이 되면 ", \ 보다 """를 쓰자
* f-string과 함께 여러 행으로 나눠서 가독성을 높일 수 있음.

```python
No:
    message = "company : {company} \n \
    department : {department}  \n \
    age : {age}  \n"
 
Yes:
    msg_str = """
    company : {company} 
    department : {department} 
    age : {age} 
    """
```

## __doc__을 사용할 수 있도록 주석은 자세히 써 주자
* 이후에 API 형태의 문서화를 할때 유리하고
* 사용자에게 바로 어떤 일을 하는 함수 인지 jupyter나 ipython 콘솔에서 알려 줄 수 있어 직관적이다

```python
class Path:
    def __init__(self):
    """
    ('PurePath subclass that can make system calls.'
    '    Path represents a filesystem path but unlike PurePath, also offers'
    '    methods to do system calls on path objects. Depending on your system,'
    '    instantiating a Path will return either a PosixPath or a WindowsPath'
    '    object. You can also instantiate a PosixPath or WindowsPath directly,'
    '    but cannot instantiate a WindowsPath on a POSIX system or vice versa.'
    '    ')
    """
 
In [5]: pprint.pprint(Path.__doc__)
('PurePath subclass that can make system calls.\n'
'\n'
'    Path represents a filesystem path but unlike PurePath, also offers\n'
'    methods to do system calls on path objects. Depending on your system,\n'
'    instantiating a Path will return either a PosixPath or a WindowsPath\n'
'    object. You can also instantiate a PosixPath or WindowsPath directly,\n'
'    but cannot instantiate a WindowsPath on a POSIX system or vice versa.\n'
'    ')
```

## 가능하면 입력 인자와 출력 인자의 type을 명시해 주자
* python, java script의 경우 c++, java 와 같이 데이터 타입을 선언하는 부분이 없다
* 직관적으로 입력 출력 되는 인자의 정보가 없고 주석이 없다면 더욱 알기가 힘들다
* 입력이나 출력인자를 정해 놓으면 다른 타입이라도 python이 casting하려는 시도를 하게 되고 잘못된 사용을 할때 인터프리터가 알아 차릴 수 있다
   - __str__() 같은 operator가 동작하게 할수 있음.

```python
No:
class Path:
    def __init__(self, path):
    ...
    def __div__(self, path): 
    ...
 
Yes:
class Path:
    def __init__(self, path:str):
    ...
    def __div__(self, path:Path)->Path: 
    ...
```
