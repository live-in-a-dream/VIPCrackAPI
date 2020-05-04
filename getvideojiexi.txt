#!/usr/bin/env python
#coding=utf-8
import execjs
import requests
import re
import execjs
import time
import sys,io
from bs4 import BeautifulSoup
import json
import logging
import sys,io
import base64
sys.stdout = io.TextIOWrapper(sys.stdout.buffer, encoding="utf-8")
logging.captureWarnings(True)
# url='https://www.iqiyi.com/v_19ryhlkaz0.html'
# xl='3'
url=sys.argv[1]
xl=sys.argv[2]
def GetUrlBs():
    headers={
        'Referer':'https://api.sigujx.com/?url='+url,
        'Accept-Language':'zh-CN,zh;q=0.9,und;q=0.8',
        'Host':'jx.126c.cn',
        'Sec-Fetch-Mode':'navigate',
        'Sec-Fetch-Site':'cross-site',
        'Accept':'text/html,application/xhtml+xml,application/xml;q=0.9,image/webp,image/apng,*/*;q=0.8,application/signed-exchange;v=b3;q=0.9',
        'User-Agent':'Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/80.0.3987.163 Safari/537.36'
    }
    re=requests.get('https://jx.126c.cn/?url='+url,headers=headers, verify=False)
    return re.text

#获取sigu_url
def get_sigu_url(bs4):
    return bs4.select('#sigu_url')[0].get('value')
    pass

def get_jss():
    with open("jquery.js", "r",encoding='utf-8') as f:
        data_func = f.read()
    with open("crypto-js.js", "r",encoding='utf-8') as f:
        data_func += f.read()
    with open("jquery.mim.js", "r",encoding='utf-8') as f:
        data_func += f.read()
    return data_func
    pass

def diaoyong_js(fu,key):
    return (execjs.compile(get_jss())).call(fu,key)
    pass



#获取playurl 大概率不需要
def jiexi():
    bs = GetUrlBs()
    bs4 = BeautifulSoup(bs, "html.parser")
    sigu_url=get_sigu_url(bs4)
    key=bs.split('sigu("')[1].split('")')[0]
    key2=bs.split('sigu2("')[1].split('")')[0]
    key3=sigu_url+'|'+'jx.126c.cn'
    codetoken=bs.split('<script type="text/javascript" src="/js/jquery.mim.js?20200202" charset="utf-8"></script>')[1].split('<script type="text/javascript">')[1].split(";$.post")[0]
    token=(execjs.compile(codetoken)).eval('token')
    date={
        'url':sigu_url,
        'key':key,
        'key2':key2,
        'key3':key3,
        'token':token,
        'type':''
    }
    date['key']=(diaoyong_js('sigu',date['key']))
    date['key2'] = (diaoyong_js('sigu2',date['key2']))
    date['key3'] =(diaoyong_js('sigu3', date['key3']))
    headers = {
        'Host': 'jx.126c.cn',
        'User-Agent': 'Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/80.0.3987.163 Safari/537.36',
        'Origin': 'https://jx.126c.cn',
        'Referer': 'https://jx.126c.cn/?url=' + url,
        'X-Requested-With':'XMLHttpRequest',
        'Sec-Fetch-Site':'same-origin',
        'Content-Type':'application/x-www-form-urlencoded; charset=UTF-8',
        'Sec-Fetch-Mode':'cors',
        'Sec-Fetch-Dest':'empty',
        'Accept':'application/json, text/javascript, */*; q=0.01'
    }
    jsonre=requests.post('https://jx.126c.cn/sigu_jx.php',data=date,headers=headers, verify=False).json()
    if jsonre['code'] == 0:
        return diaoyong_js('sigu_decode', jsonre['url'])
        pass
    else :
        return '错误'
        pass
    pass
def getM3u8():
    bs = GetUrlBs()
    bs4 = BeautifulSoup(bs, "html.parser")
    sigu_url = get_sigu_url(bs4)
    date = {
        'url': sigu_url
    }
    headers = {
        'Host': 'zl.126c.cn',
        'User-Agent': 'Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/80.0.3987.163 Safari/537.36',
        'Origin': 'https://zl.126c.cn',
        'Referer': 'https://zl.126c.cn/?url=' + sigu_url,
        'X-Requested-With':'XMLHttpRequest',
        'Accept':'application/json, text/javascript, */*; q=0.01'
    }
    jsonre = requests.post('https://zl.126c.cn/zl_api.php', data=date, headers=headers, verify=False).json()
    if jsonre['code'] == 0:
        return jsonre['url']
        pass
    else :
        return '错误'
        pass
    pass
if xl == '1':
	urlss=jiexi()
	pass
if xl == '2':
	urlss=getM3u8()
	pass
print(u""+urlss)