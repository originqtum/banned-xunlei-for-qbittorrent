# banned-xunlei-for-qbittorrent
python qbittorrent xunlei banned

## 功能
屏蔽迅雷客户端,每隔3秒运行一次. 可配置需要屏蔽的客户端

## 屏蔽列表
-XL0012-***  
Xunlei/***  
7.x.x.x

## 开发测试环境如下
1. 需要安装 Python 3.8
2. qbittorrent v4.1.9.1
3. 开启webui
4. 运行 python.exe main.py

## 脚本
```python
import requests
import json
import time
import random
import math
import string


def getDownloadItem(url, cookie):
    # random number
    rid = random.random() * 1000
    rid = math.floor(rid)

    param = ''.join(random.sample(string.ascii_letters + string.digits, 8))
    content = {'rid': rid, param: ''}
    rsp = requests.get(url, params=content, headers=def_headers, cookies=cookie)
    return json.loads(str(rsp.content, 'utf-8'))['torrents']


def getTorrentHashAndStatus(torrentVal, value):
    status = value.get('state')
    if status is not None:
        status = status.lower()
        if "uploading" == status or "downloading" == status:
            return torrentVal


def queryTorrentClient(url, cookie, hashVal):
    blockList = []
    rid = random.random() * 1000
    rid = math.floor(rid)

    param = ''.join(random.sample(string.ascii_letters + string.digits, 8))
    content = {'rid': rid, 'hash': hashVal, param: ''}

    rsp = requests.get(url, params=content, headers=def_headers, cookies=cookie)
    PeersInfo = json.loads(str(rsp.content, 'utf-8'))['peers']

    for peer in PeersInfo:
        if isNeedBlockClient(PeersInfo[peer]):
            blockList.append(PeersInfo[peer])
    return blockList


def reloadIpFilter(url, bannedIps, cookie):
    reload = {'ip_filter_enabled': True, 'banned_IPs': bannedIps}
    content = {'json': json.dumps(reload, ensure_ascii=False)}

    requests.post(url, content, headers=headers, cookies=cookie)


def getAllFilerClient(itemsHash, url, cookie):
    blockAllList = []
    for item in itemsHash:
        blockList = queryTorrentClient(url, cookie, item)
        if blockList is not None:
            blockAllList.append(blockList)
    return blockAllList


def isNeedBlockClient(peer):
    client = peer['client']
    for deFilter in filterClient:
        if deFilter['findType'] == 1 and client.find(deFilter['name']) > -1:
            return True
        elif deFilter['findType'] == 2 and client.startswith(deFilter['name']):
            return True

    return False


def writeToFile(newAddList):
    with open(fileAddress, 'a+') as f:
        for writeIp in newAddList:
            f.write(writeIp + '\n')
        f.close()


def readExistIp():
    with open(fileAddress, 'r') as f1:
        existFilerClient = f1.readlines()
    f1.close()
    return existFilerClient


def notExistIp(existFilerClient, ipsCollect):
    newAddList = []
    for ip in ipsCollect:
        if ''.join(existFilerClient).strip('\n').find(ip['ip']) == -1:
            newAddList.append(ip['ip'])
            print("find new client: " + ip['ip'] + '\t\t' + ip['client'])
    return newAddList


if __name__ == "__main__":
    # 定义需要过滤的客户端 name区分大小写 findType 1包含匹配 2前缀匹配
    filterClient = [
        {'name': '-XL0012', 'findType': 1},
        {'name': 'Xunlei', 'findType': 1},
        {'name': 'Xfplay', 'findType': 1},
        {'name': 'QQDownload', 'findType': 1},
        {'name': '7.', 'findType': 2}
    ]

    # 将过滤的ip写入文件
    fileAddress = r'C:\ipfilter.dat'
    # webUi访问地址
    Root_url = 'http://127.0.0.1:1000'
    # 登陆接口
    login_url = Root_url + '/api/v2/auth/login'
    # 获取正在下载项或上传项
    mainData_url = Root_url + '/api/v2/sync/maindata'
    # 获取下载文件连接的用户
    peers_url = Root_url + '/api/v2/sync/torrentPeers'
    # 设置生效接口
    filter_url = Root_url + '/api/v2/app/setPreferences'

    headers = {
        'Accept': 'text/javascript, text/html, application/xml, text/xml, */*',
        'Content-Type': 'application/x-www-form-urlencoded; charset=UTF-8',
        'User-Agent': 'Mozilla/5.0 (Windows NT 10.0; WOW64) AppleWebKit/537.36 (KHTML, like Gecko) '
                      'Chrome/78.0.3904.70 Safari/537.36 '
    }

    def_headers = {
        'Accept': 'application/json',
        'Accept-Encoding': 'gzip, deflate, br',
        'User-Agent': 'Mozilla/5.0 (Windows NT 10.0; WOW64) AppleWebKit/537.36 (KHTML, like Gecko) '
                      'Chrome/78.0.3904.70 Safari/537.36 '
    }

    # webUi登陆用户名密码
    data = {
        'username': 'admin',
        'password': '123456789'
    }
    response = requests.post(login_url, data, headers=headers)

    # login error 登陆失败退出
    if response.text != 'Ok.':
        exit(0)

    # 进入循环运行
    while True:
        cookie_jar = response.cookies

        downItem = getDownloadItem(mainData_url, cookie_jar)
        downloadList = []
        for torrent in downItem:
            hashKey = getTorrentHashAndStatus(torrent, downItem[torrent])
            if hashKey is not None:
                downloadList.append(hashKey)

        allFilerClientList = getAllFilerClient(downloadList, peers_url, cookie_jar)
        clients = [y for x in allFilerClientList for y in x]

        existIps = readExistIp()
        newList = notExistIp(existIps, clients)

        if len(newList) > 0:
            writeToFile(newList)
            ips = ''.join(existIps) + '\n'.join(newList)
            # print(ips)
            reloadIpFilter(filter_url, ips, cookie_jar)

        # 3秒执行一次
        time.sleep(3)

```
## 日志
检测到新的连接时
```
find new client: *.*.*.*		Xunlei 0.0.1.8
find new client: *.*.*.*		-XL0012-#�����Q���
find new client: *.*.*.*		-XL0012-^ȕ�Dm�[ǏW
find new client: *.*.*.*		-XL0012-����d+5ۆ�t
find new client: *.*.*.*		-XL0012-�_���c�`F݋
find new client: *.*.*.*		-XL0012-}[���W�Q��
find new client: *.*.*.*		-XL0012-,���|�����2�
...
```
