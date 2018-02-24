---
title: "pipeline-case-study"
date: 2018-02-24 10:12
---
# Python 单机处理30G数据

```python
from sgmllib import SGMLParser
class GetIdList(SGMLParser):
    def reset(self):
        self.IDlist = []
        self.flag = False
        self.getdata = False
        self.verbatim = 0
        SGMLParser.reset(self)
        
    def start_div(self, attrs):
        if self.flag == True:
            self.verbatim +=1 #进入子层div了，层数加1
            return
        for k,v in attrs:#遍历div的所有属性以及其值
            if k == 'class' and v == 'entry-content':#确定进入了<div class='entry-content'>
                self.flag = True
                return

    def end_div(self):#遇到</div>
        if self.verbatim == 0:
            self.flag = False
        if self.flag == True:#退出子层div了，层数减1
            self.verbatim -=1

    def start_p(self, attrs):
        if self.flag == False:
            return
        self.getdata = True
        
    def end_p(self):#遇到</p>
        if self.getdata:
            self.getdata = False

    def handle_data(self, text):#处理文本
        if self.getdata:
            self.IDlist.append(text)
            
    def printID(self):
        for i in self.IDlist:
            print i



lister = GetIdList()
lister.feed(the_page)
lister.printID()

path = "../items.txt"
count = 2
import json
with open(path, "r") as f:
    for line in f:
        if count == 0:
            break
        item = json.loads(line.decode('utf8'), 'utf8')
#         print item.keys()
#         print item['category']
#         print item['info_id']
#         print item['publish_time']
#         print item['title']
#         print item['fromdb']
#         print item['media']
#         print item['item_type']
#         print item['content']
#         print item['source']
#         print item['topic']
#         print item['tag']
#         print item['poi']
#         print item['source_id']
#         print item['position']
        for key, val in item.iteritems():
            print key, val
        count -= 1
        
```
