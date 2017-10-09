#!/usr/bin/env python
# -*- coding: utf-8 -*-
# @Time    : 2017/10/9 22:12
# @Version : 1.0
# @Author  : Leiyu
# @File    : mmtao.py.py
import urllib2
from json import loads
import re

#user-agent 大全，反扒
headerstr = '''Mozilla/5.0 (Macintosh; U; Intel Mac OS X 10_6_8; en-us) AppleWebKit/534.50 (KHTML, like Gecko) Version/5.1 Safari/534.50
Mozilla/5.0 (Windows; U; Windows NT 6.1; en-us) AppleWebKit/534.50 (KHTML, like Gecko) Version/5.1 Safari/534.50
Mozilla/5.0 (compatible; MSIE 9.0; Windows NT 6.1; Trident/5.0;)'''

def headers():
    header = headerstr.split('\n')
    length = len(header)
    return header[random.randint(0,length-1)] #数组下标从零开始

#拿到模特信息
def getUrlList():
    req = urllib2.Request("https://mm.taobao.com/tstar/search/tstar_model.do?_input_charset=utf-8")
    req.add_header('user-agent',headers())
    #decode解码为Unicode  encode编码为utf-8  解决中文乱码问题
    html = urllib2.urlopen(req,data='q=&viewFlag=A&sortType=default&searchStyle=&searchRegion=city%3A&searchFansNum=&currentPage=1&pageSize=100').read().decode('gbk').encode('utf-8')
    #string类型转为字典类型
    json = loads(html)
    return json['data']['searchDOList']

#拿到相册列表
def getAlbumList(userid):
    req = urllib2.Request('https://mm.taobao.com/self/album/open_album_list.htm?_charset=utf-8&user_id%%20=%s' %userid)
    req.add_header('user-agent',headers())
    html = urllib2.urlopen(req).read().decode('gbk').encode('utf-8')
    #使用正则表达式匹配html返回值中的相册列表地址
    reg = r'class="mm-first" href="//(.*?)"'
    #[::2] 参数分别为 起始值，终止值，步长
    return re.findall(reg,html)[::2]

#拿到每张大照片的地址
def getPic(userid,albumid):
    req = urllib2.Request('https://mm.taobao.com/album/json/get_album_photo_list.htm?user_id=%s&album_id=%s' %(userid,albumid))
    req.add_header('user-agent',headers())
    html = urllib2.urlopen(req).read().decode('gbk').encode('utf-8')
    json = loads(html)
    return json['picList']

for i in getUrlList():
    print i['realName']
    userid = i['userId']
    print userid

    for n in getAlbumList(userid):
        albumid = n.split('album_id=')[1].split('&')[0]
        print albumid
        for m in getPic(userid,albumid):
            #去除图片地址尾部图片大小限制
            print m['picUrl'].split('_290')[0]
        print '\n'
    print '-----------------'
