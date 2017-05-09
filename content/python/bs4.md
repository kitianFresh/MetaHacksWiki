---
title: "bs4"
date: 2017-05-09 21:43
---

beautifulsoup 根据 text 找 tag的问题
```html
<div class="contentBlurb">
       Clinical Information
       <ul class="definitionList">
        <li>
         Acute diarrheal disease endemic in India and southeast Asia whose causative agent is vibrio cholerae; can lead to severe dehydration in a matter of hours unless quickly treated.
        </li>
        <li>
         An acute diarrheal disease endemic in India and southeast Asia whose causative agent is vibrio cholerae. This condition can lead to severe dehydration in a matter of hours unless quickly treated.
        </li>
        <li>
         Cholera is a bacterial infection that causes diarrhea. The cholera bacterium is usually found in water or food contaminated by feces . Cholera is rare in the United States. You may get it if you travel to parts of the world with inadequate water treatment and poor sanitation, and lack of sewage treatment. Outbreaks can also happen after disasters. The disease is not likely to spread directly from one person to another. Often the infection is mild or without symptoms, but sometimes it can be severe. Severe symptoms include profuse watery diarrhea, vomiting, and leg cramps. In severe cases, rapid loss of body fluids leads to dehydration and shock. Without treatment, death can occur within hours. Doctors diagnose cholera with a stool sample or rectal swab. Treatment includes replacing fluid and salts and sometimes antibiotics. Anyone who thinks they may have cholera should seek medical attention immediately. Dehydration can be rapid so fluid replacement is essential. Centers for Disease Control and Prevention.
        </li>
       </ul>
</div>
```

`bsObj.findAll("div", attrs={'class' : 'contentBlurb'}, text=re.compile(".*Clinical.*"))`无法查到该div，因为默认只查找tag里面只包含文本的，而不是tag里面又含有tag的； 只能先查找到所有的div， 然后检查每一个div是否含有text；

```python
for div in bsObj.findAll("div", attrs={'class' : 'contentBlurb'}):
        # print div 
        if div.find(text=re.compile("Clinical Information")):  
            for l in div.find_all('li'):
                clinical_info += l.string + '\n'
            print clinical_info
```
```html
<div class="contentBlurb">
       Clinical Information
       <ul class="definitionList">
        <li>
         <em>
          salmonella
         </em>
         is the name of a group of bacteria. In the United States, it is the most common cause of foodborne illness.
         <em>
          salmonella
         </em>
         occurs in raw poultry, eggs, beef, and sometimes on unwashed fruit and vegetables. Symptoms include fever, diarrhea, abdominal cramps and headache. Symptoms usually last 4 - 7 days. Most people get better without treatment. It can be more serious in the elderly, infants and people with chronic conditions. If
         <em>
          salmonella
         </em>
         gets into the bloodstream, it can be serious, or even life-threatening. The usual treatment is antibiotics. You also can get a salmonella infection after handling pets, particularly reptiles like snakes, turtles and lizards. Typhoid fever, a more serious disease caused by
         <em>
          salmonella
         </em>
         , frequently occurs in developing countries.
        </li>
        <li>
         Infections with bacteria of the genus salmonella.
        </li>
        <li>
         Infekce bakteriemi rodu salmonella.
        </li>
       </ul>
      </div>
```
此时 `l.string` 为空，因为里面又出现了tag；这里使用 `l.get_text()` 更加有效；
 -[beautifulsoup-search-by-text-inside-a-tag](http://stackoverflow.com/questions/31958637/beautifulsoup-search-by-text-inside-a-tag)

后台部署,`nohup` 可以忽略终端控制信号
```
nohup python DataCrawler4.py &
ps -ef | grep pid
```