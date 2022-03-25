# -demo_Bilibili
import requests,re
import json
from pprint import pprint

# 定义一个函数用来获取 响应的内容
def get_response(html_url):
    '''
    该函数主要用来获取响应的内容
    :param html_url: 目标网址
    :return: 返回的是目标网址的响应内容
    '''

    #加上请求头，目的是伪装成浏览器
    #加入防盗链 目的是为了告诉浏览器去哪里找这个目标url
    header = {
        'User-Agent': 'Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/99.0.4844.74 Safari/537.36',
        'referer' : 'https://www.bilibili.com'
    }

    response = requests.get(url=html_url,headers=header )
    return response

# 定义一个函数用来获取视频的相关信息
def get_video_info(html_url):
    '''
    用来获取视频的标题，音频链接，视频链接
    :param html_url: 视频的详细页面
    :return: 标题，音频链接，视频链接
    '''
    response = get_response(html_url=html_url)
    #打印出网页源码
    # print(response.text)
    #解析数据获取视频的中文名称
    title = re.findall(' <title data-vue-meta="true">(.*?)_哔哩哔哩_bilibili</title>',response.text)[0]
    # print(title)

    # 解析获取视频的链接
    html_data = re.findall('<script>window.__playinfo__=(.*?)</script>',response.text)[0]
    # print(html_data)
    #将以上的html转成字典形式的数据
    json_data = json.loads(html_data)
    # pprint(json_data)

    #获取音频url
    audio_url = json_data['data']['dash']['audio'][0]['baseUrl']
    #获取视频的url
    video_url = json_data['data']['dash']['video'][0]['baseUrl']
    # print(audio_url)
    # print(video_url)

    #将相应的内容以返回值的形式输出
    video_info = [title,audio_url,video_url]
    return video_info

#定义一个函数用来保存数据
def save(title,audio_url,video_url):
    '''
    保存数据
    :param title:
    :param audio_url:
    :param video_url:
    :return:
    '''
    #获取音频数据
    audio_contend = get_response(html_url=audio_url).content
    #获取视频数据
    video_contend = get_response(html_url=video_url).content

    # 创建文件用来保存获取的数据
    with open(title+'.mp3','wb')as file:
        file.write(audio_contend)
    with open(title+'.mp4','wb')as file:
        file.write(video_contend)

#定义一个主函数用来整合所有函数
def main(bv_id):
    '''
    主函数
    :param bv_id:要爬取的视频的id
    :return:
    '''
    url = f'https://www.bilibili.com/video/{bv_id}'
    video_info = get_video_info(url)
    save(video_info[0],video_info[1],video_info[2])

keyword = input('请输入要爬取的视频bv_id号码：')
main(keyword)
