---
layout: default
title: "장고와 리액트 연동하기(데이터베이스(1))"
---

> 장고에 본인 db 연동하기

## 연동 시작하기 : Mysql환경설정(MySQL workbench이용)

<br/>
사실 콘솔창에서 작업을 할 수도 있습니다 
<br/>
하지만 더 편한 환경에서 작업하기 위해 workbench를 활용해 작업하시는 것을 추천드립니다
<br/>

1. 데이터 베이스 스키마 생성하기
2. 생성된 스키마를 장고와 연결시키기
3. 장고 ORM을 활용해 생성된 디비에 테이블 만들기
<br/>
의 순서로 설명을 진행하겠습니다
<br/>

# 데이터 베이스 스키마 생성하기
<br/>
<b>첫 번째로</b> 
root connection이 아닌 새로운 user를 만들고 그 user의 connection안에서 독립된 스키마를 만들어주셔야 합니다.
<br/>
사실 루트유저로 모든 것을 해도 돌아가는데 문제는 없지만 비유하자면, 회장님한테(root유저가) 경영부터(sys작업 등) 인턴잡무까지(우리가 활용하려는 프로젝트의 db)를 모두 맡기는 꼴이 됩니다.
<br/>
새로운 connection을 만들고 user를 만드는 순서는 다음과 같습니다
<br/>
<br/>

## 1. root유저로 접속, 새로운 user을 생성

맨 처음, root유저로 접속하셨다면 이전의 모든 스키마를 보실 수 있을 겁니다

![image1](https://drive.google.com/uc?id=1qKqyrDVwSCl34JnAERNBuFGcrGKEsQD_)

이제 새로운 user을 생성하기 위해 다음의 화면에 들어가 new account를 만들어 줍니다.

![image2](https://drive.google.com/uc?id=1cNsZSw3v2_uqmU1lR4mPEunxhRxN9qzM)

이때 Schema privileges를 설정해 주셔야 새롭게 만든 account가 query를 통해 스키마를 조작할 수 있습니다
![image3](https://drive.google.com/uc?id=16SKU0ML6Pz-TmUzwR46c_eRHjeI0lxjI)

## 2. 새로운 connection을 생성, 방금 만든 user을 그 connection의 id로 등록

새로운 connection을 만들고 해당 connection에 접속할 수 있는 Accounts를 설정해 줍니다.

![image4](https://drive.google.com/uc?id=150SAYD8cyYhLrwCbMdkQHVR-yzD2Rdxm)

아래는 위의 ForPost의 connection에 account를 grant하는 모습입니다.

![image5](https://drive.google.com/uc?id=1c9Q-7SbNEkU8EmR7zelQk2BIFJVtHHuk)

## 3. 새롭게 생성된 connection내부에서 root유저와는 독립된 스키마를 생성

![image6](https://drive.google.com/uc?id=1O3EDatofOptc4fNsQILkGc2Pt6Bef0P5)

이제 보시면 아까 root유저로 접근했을 때와는 다른 화면이 보입니다. 이제 해당 connection을 장고와 연결시키면 됩니다


<br/>
<br/>

# 생성된 스키마를 장고와 연동시키기

<b>두 번째로</b> 
위 지정한 host주소, user id, user pw를 장고의 settings.py 의 환경설정을 통해 연결해주셔야 합니다.
<br/>
방금 독립된 connection으로 스키마를 만드는 것은 사원 한 명을 채용해서(user) A부서를(db schema) 만드는 것과 같으며,
<br/>
이제부터 할 작업은 그 A부서와 B거래처(django)를 연결하는 작업이라고 생각하시면 됩니다.
<br/>
이 과정에서 mysqlclient라는 모듈을 pip 명령어를 통해 설치하라는 경고메시지가 뜰 것입니다.
<br/>
그대로 설치 진행해 주시면 됩니다.
<br/>
<br/>

```python

# [myproject]/[project_folder]/settings.py에서 
# 아래처럼 DATABASES부분을 수정해줍니다
# 'ENGINE'의 경우 사용하는 db에 따라 달라지며 아래 코드는 mysql기준입니다
# 'NAME'의 경우 db 스키마,
# 'USER'와 'PASSWORD'의 경우 위에서 설정한 connection의 id와 pw입니다.

DATABASES = {
    'default': {
    'ENGINE': 'django.db.backends.mysql',
    'NAME': 'something_schema_name',
    'USER' : 'something_user_id',
    'PASSWORD' : 'something_user_pw',
    'HOST': '',
    'PORT' : '',
    }
}

```

## 이제 명령어를 통해 settings.py의 변동내용을 빌드해줍니다

```
// 제 경우는 따로 스키마나 모델을 생성해주지 않았기 때문에 No changes detected의 메시지가 나옵니다.
// 본인의 변경사항이 콘솔창에 출력될 것입니다
(venv_ver01) gimgiseong-ui-MacBook-Pro:local_testing kisung$ python manage.py makemigrations
No changes detected

// 본인의 변경사항이 콘솔창에 출력될 것입니다.
(venv_ver01) gimgiseong-ui-MacBook-Pro:local_testing kisung$ python manage.py migrate
Operations to perform:
Apply all migrations: admin, app01, auth, contenttypes, sessions
Running migrations:
No migrations to apply.

```

# 장고  ORM을 활용해 생성된 디비에 테이블 만들기

<b>마지막으로</b> 
장고의 models.py에서 생성하려는 테이블의 기본 정보를 입력해주시고 manage.py를 활용, makemigrations와 migrate명령어를 통해 적용시켜주면 됩니다.
<br/>
즉 이제 B거래처(django)에서 A부서에게(db schema) 명세서를 만들고(models.py 수정) 그 명세서를 A부서에게 전달(makemigrations등의 명령어 활용)한다고 보시면 됩니다.
<br/>

```python

# 현재 파일은 [myproject]/[application_folder]/models.py 입니다

# 아래는 제가 생성한 테이블을 그대로 따왔습니다.

# 즉 설명하자면 Sangga라는 이름의 테이블을 만드는데,
# ksic_nm이라는 캐릭터필드의 null을 허용하며 최대 길이를 200자까지 허용하는 열을 만든다
# 라는 의미입니다.

# 이렇게 models.py를 수정해주고 makemigrations, migrate명령어를 활용하면,
# 실제 해당 스키마에는 [application_name]_[class_name]라는 테이블,
# 즉 저의 경우에는 app01_sangga라는 이름의 테이블이 자동으로 생성되었습니다

# 추가 내용은 구글링을 참고하세요

# 코드 예시
#########
from django.db import models


class Sangga(models.Model):
    ksic_nm = models.CharField(max_length=200, null=True)
    bizes_nm = models.CharField(max_length=200, null=True)
    inds_l_nm = models.CharField(max_length=200, null=True)
    inds_m_nm = models.CharField(max_length=200, null=True)
    inds_s_nm = models.CharField(max_length=200, null=True)
    brch_nm = models.CharField(max_length=200, null=True)
    adr_rdnm = models.CharField(max_length=200, null=True)
    adr_ino = models.CharField(max_length=200, null=True)
    lat = models.CharField(max_length=200, null=True)
    lon = models.CharField(max_length=200, null=True)

```


