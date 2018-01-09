---
layout: post
title: "[ Python ] 정규식을 써보자. "
description: >
  자주 사용하는 정규식 패턴과 예제  
tags: [python]

---

개발을 하다보면 종종 스크립트를 작성해야할 때가 생긴다. 
이런 경우 어김없이 정규식을 써야하는 상황에 처하게 된다. 
이번 포스팅에선 필자가 최근 사용했던 python 정규식 패턴을 소개하고자 한다. 


#문제상황1
나만의 script를 만들어서 실행할 때 여러 옵션을 주고 싶은 경우가 있다. 
여러 옵션을 제각기 if else로 처리하게 되면 코드가 복잡해지고 쓸데없이 비대해진다. 
이런 경우 정규식을 이용해서 옵션값을 처리할 수 있다. 

### 콘솔에서 실행 예 
~~~python
$myscript.py -option1 -option2 -option3 
~~~

### 실행 option처리 코드 
~~~python
import re
import sys
commandString = ""
for word in sys.argv:
	commandString += word
	commandString += " "


# 다양한 option을 정규식으로 찾게 되면 len(sys.argv)과 독립적으로 코드를 잘 수 있게 된다.
mat = re.search(r"\s(-option1)", commandString)
	if mat != None :
		if len(mat.groups()) == 1:
        # something todo 
        
mat = re.search(r"\s(-option2)", commandString)
	if mat != None :
		if len(mat.groups()) == 1:
        # something todo 

mat = re.search(r"\s(-option3)", commandString)
	if mat != None :
		if len(mat.groups()) == 1:
        # something todo 

~~~

#문제상황2
Text가 있고 Text내 특정 범위에서 원하는 패턴의 문자열을 추출하고 싶은 경우를 
예로 들어보겠다. 

예제 Text가 아래와 같이 주어졌을 때 iOS 10의 iphone 6s의 식별자를 구해보자. 
~~~
== Devices ==
-- iOS 8.1 --
    iPhone 4s (4323AF5D-428D-4D40-A0B1-D6BC6EBD16F9) (Shutdown)
    iPhone 4s (E70D2209-9635-4CB2-8FA0-EB0CF4F41332) (Shutdown)
    iPhone 5 (FBE63563-A73B-4095-B39D-901ADB4002CB) (Shutdown)
    iPhone 5 (24288608-E17E-49F8-A0B1-E9D0FD24032F) (Shutdown)
    iPhone 5s (06980D4F-4996-4AB6-BF97-E012810331D1) (Shutdown)
    iPhone 5s (18113A18-537C-403B-956A-43400A283560) (Shutdown)
    iPhone 6 (B7756D11-2CBD-4DCC-A511-5E3C883250E8) (Shutdown)
    iPhone 6 (93D5CC11-5D05-4CFD-A6EB-2F9080326FA4) (Shutdown)
    iPhone 6 Plus (258EA675-B45F-4114-A083-316994CF40CB) (Shutdown)
    iPhone 6 Plus (6E8D2CE5-881C-4D50-AEA8-B72881F3EF18) (Shutdown)
    iPad 2 (9FC6EAEC-9A07-4D54-9F26-CD58680612A9) (Shutdown)
    iPad 2 (B51C4C7D-C827-45A7-B946-4A5054F0271B) (Shutdown)
    iPad Retina (5DDAAC94-B102-40FA-868F-5ECFD89EAA5E) (Shutdown)
    iPad Retina (4AF3584C-4039-430F-8898-4E28CF06D7D5) (Shutdown)
    iPad Air (7A394489-CAD4-4137-BC9E-6C86F6DAB359) (Shutdown)
    iPad Air (DBED22B2-BDA0-4A15-9A98-2E52D7B5A907) (Shutdown)
-- iOS 9.3 --
    iPhone 4s (83EC2985-1905-4CEF-8ADD-46C5A8348679) (Shutdown)
    iPhone 5 (321B1AD4-C4A1-4B07-9C55-C3EF27E204DF) (Shutdown)
    iPhone 5s (300893DD-DC5B-4389-B489-5721FCD08462) (Shutdown)
    iPhone 6 (70BCC652-7482-46BA-A5A8-1224F56F0264) (Shutdown)
    iPhone 6 Plus (114FEFA2-D90C-49FB-A5AE-29FB43494A01) (Shutdown)
    iPhone 6s (739912CD-E9DA-4326-A89D-6034987FAC90) (Shutdown)
    iPhone 6s Plus (98BB84F7-2130-41DC-9C69-F82A32B608F3) (Shutdown)
    iPad 2 (8BB9E573-FE53-487C-97DD-2E422C0B3087) (Shutdown)
    iPad Retina (D3668AF1-98BD-458F-BE9E-4D2B44EF7B70) (Shutdown)
    iPad Air (63EC3CBE-15EE-4A25-A612-6C251641EDA4) (Shutdown)
    iPad Air 2 (4ABFF429-A809-4CF7-AB9F-A14FD11E25F5) (Shutdown)
    iPad Pro (6F4F2253-88D7-4460-B746-FFDCC910F091) (Shutdown)
-- iOS 10.0 --
    iPhone 5 (9C86D90E-66C5-401C-B260-686AE4703EED) (Shutdown)
    iPhone 5s (8171471A-7296-454F-A075-C3E2548CF446) (Shutdown)
    iPhone 6 (A0A66737-A718-441A-AE9B-195D04FD5D2F) (Shutdown)
    iPhone 6 Plus (66F6662E-B236-49B4-A1C3-C8DA2DF9C8D1) (Shutdown)
    iPhone 6s (4644DD45-4614-4197-A09D-A9503F2F99C5) (Shutdown)
    iPhone 6s Plus (13B908CA-DABD-47B1-BFBC-294DF5A58FE6) (Shutdown)
    iPhone 7 (0D596F6E-4473-4B66-ABCC-C730B93709DF) (Shutdown)
    iPhone 7 Plus (F0395EA7-C29C-4608-BD01-490F7512180F) (Shutdown)
    iPhone SE (D7732E4F-4FC6-4A66-B239-1964CB9256FA) (Shutdown)
    iPad Air (F8B3486A-59A0-4764-A1CD-9816AAF8EFD8) (Shutdown)
    iPad Air 2 (C66DD907-ACAA-4001-A32B-2D6102979F2F) (Shutdown)
    iPad Pro (9.7 inch) (DD0F8C16-F333-4FD5-9725-631E1C12C202) (Shutdown)
    iPad Pro (12.9 inch) (F53124A0-18B0-41E4-9CBB-3FBECCF9409C) (Shutdown)
~~~

## Text내의 특정 범위에서 문자열 추출 코드 
~~~python
import re
	deviceText = "" 
    #deviceText = 예제 텍스트 입력 
    
    # 정규식  : "iOS 10(.*)?\n(\s*iPhone(.*)\n)*(\s*iPhone 6s \(((.*?)\)))"
    # \s : 공백문자
    # (.*) : 임의의 문자가 0개 이상 반복
    # ? : 0또는 1차례 발생 의미
	match = re.search(r"iOS 10(.*)?\n(\s*iPhone(.*)\n)*(\s*iPhone 6s \(((.*?)\)))", deviceText)

	if len(match.groups()) == 6 : # 정규식 패턴에서 6번째 그룹으로 매칭된 것을 찾는다. 
    	print(match.groups()[5])
~~~