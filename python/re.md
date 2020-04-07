[正则表达式](../other/正则表达式.md)

* re库
    * raw string &emsp;&emsp;   eg. r'[1-9]\d{5}'
    * 普通字符串   &emsp;&emsp;   eg. '[1-9]\\d{5}'
* 常用函数
    * re.search()  &emsp;&emsp;  在字符串中搜索匹配正则表达式的第一个位置，返回match对象
    * re.match()    &emsp;&emsp;  从一个字符串的开始位置起匹配正则表达式，返回match对象
    * re.findall()  &emsp;&emsp;  搜索字符串，以列表类型返回能匹配的全部子串
    * re.split() &emsp;&emsp;   将字符串按照正则匹配结果进行切分，返回列表类型
    * re.finditer() &emsp;&emsp;  搜索字符串，返回能匹配的迭代类型，每个迭代类型是match对象
    * re.sub() &emsp;&emsp;  在一个字符串中替换所有匹配正则表达式的子串，返回替换后的字符串

* 测试样例
    ```python
    
    import re
    
    pattern = r'[1-9]\d{5}'
    
    match = re.search(pattern, 'BIT100081 TSU100084')
    if match:
        print(match.group(0))  # 100081
    
    lt = re.findall(pattern, 'BIT100081 TSU100084')
    print(lt)  # ['100081', '100084']
    
    lt = re.split(pattern, 'BIT100081 TSU100084')
    print(lt)  # ['BIT', ' TSU', '']
    
    lt = re.split(pattern, 'BIT100081 TSU100084', maxsplit=1)
    print(lt)  # ['BIT', ' TSU100084']
    
    new_str = re.sub(pattern, 'xxxx', 'BIT100081 TSU100084')
    print(new_str)  # BITxxxx TSUxxxx
    
    it = re.finditer(pattern, 'BIT000081 TSU000084')
    for match in it:
        print(match.group(0))  
        #100081
        #100084
        
    ``` 