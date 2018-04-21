---
title: "python-string"
date: 2017-04-30 11:22
---
[TOC]

## 格式字符串手册
数字格式化

下面的表格展示了使用Python的后起新秀str.format()格式化数字的多种方法，包含浮点数格式化与整数格式化示例。可使用 print("FORMAT".format(NUMBER)); 来运行示例，因此你可以运行： print("{:.2f}".format(3.1415926)); 来得到第一个示例的输出。

        数字	格式	输出	描述
        3.1415926	{:.2f}	3.14	保留小数点后两位
        3.1415926	{:+.2f}	+3.14	带符号保留小数点后两位
        -1	{:+.2f}	-1.00	带符号保留小数点后两位
        2.71828	{:.0f}	3	不带小数
        5	{:0>2d}	05	数字补零 (填充左边, 宽度为2)
        5	{:x<4d}	5xxx	数字补x (填充右边, 宽度为4)
        10	{:x<4d}	10xx	数字补x (填充右边, 宽度为4)
        1000000	{:,}	1,000,000	以逗号分隔的数字格式
        0.25	{:.2%}	25.00%	百分比格式
        1000000000	{:.2e}	1.00e+09	指数记法
        13	{:10d}	        13	右对齐 (默认, 宽度为10)
        13	{:<10d}	13	左对齐 (宽度为10)
        13	{:^10d}	    13	中间对齐 (宽度为10)



## Python获取application运行的路径
```python
import sys

def is_packaged():
    if hasattr(sys, 'frozen'):
        return sys.frozen

    return False


def get_root():
    return sys._MEIPASS

```
```python

from . import pyinstaller


def get_mclouds_root():
    if pyinstaller.is_packaged():
        return pyinstaller.get_root()

    cur_path = os.path.dirname(__file__)
    cur_path = os.path.abspath(cur_path)
    assert(cur_path.endswith('clouds/common'))
    pos = cur_path.rfind('meituan/server')
    assert(pos > 0)

    return cur_path[:pos]


def get_mclouds_scripts_path():
    return os.path.join(get_mclouds_root(), 'meituan/scripts')


def get_mclouds_workspace_path():
    root_path = get_mclouds_root()
    if pyinstaller.is_packaged():
        root_path = '/opt/cloud/'

    return os.path.join(root_path, 'workspace')
```
## chroot机制可以更改当前程序运行的参考根目录
chroot 可以实现安全隔离，从新制造一个新的根目录，并且程序只对该目新的根目录具有权限，其他目录对该进程就不可见了。从而实现安全隔离，但是如果程序还想要访问其他的库，命令，需要组织根目录，把相应的库文件放到相应的目录下面，可以用 busybox实现这个功能。就是有制造出一个linux系统资源目录出来给某个进程使用，类似容器隔离技术。
```python
import os, sys
newroot = '/Users/kiki/tmp'
if not os.path.exists(newroot):
    os.makedirs(newroot)
os.chroot(newroot)
print __file__
print os.getcwd()
print os.path.abspath(__file__)
```
## argparse 中 `metavar` 不能和`action=store_true` 同时用。



```python
# User Instructions
#
# Write a function best_wild_hand(hand) that takes as
# input a 7-card hand and returns the best 5 card hand.
# In this problem, it is possible for a hand to include
# jokers. Jokers will be treated as 'wild cards' which
# can take any rank or suit of the same color. The 
# black joker, '?B', can be used as any spade or club
# and the red joker, '?R', can be used as any heart 
# or diamond.
#
# The itertools library may be helpful. Feel free to 
# define multiple functions if it helps you solve the
# problem. 
#
# -----------------
# Grading Notes
# 
# Muliple correct answers will be accepted in cases 
# where the best hand is ambiguous (for example, if 
# you have 4 kings and 3 queens, there are three best
# hands: 4 kings along with any of the three queens).

import itertools

black_cards = [r + s for r in '23456789TJQKA' for s in 'SC']
red_cards = [r + s for r in '23456789TJQKA' for s in 'HD']

def best_hand(hand):
    return max(itertools.combinations(hand, 5), key=hand_rank)

def replacements(card):
    if card == '?B':
        return black_cards
    elif card == '?R':
        return red_cards
    else:
        return [card]

def best_wild_hand(hand):
    "Try all values for jokers in all 5-card selections."
    all_hands = [best_hand(hand) 
                for hand in itertools.product(*map(replacements, hand))]
    return max(all_hands, key=hand_rank)
    # Your code here

def test_best_wild_hand():
    assert (sorted(best_wild_hand("6C 7C 8C 9C TC 5C ?B".split()))
            == ['7C', '8C', '9C', 'JC', 'TC'])
    assert (sorted(best_wild_hand("TD TC 5H 5C 7C ?R ?B".split()))
            == ['7C', 'TC', 'TD', 'TH', 'TS'])
    assert (sorted(best_wild_hand("JD TC TH 7C 7D 7S 7H".split()))
            == ['7C', '7D', '7H', '7S', 'JD'])
    return 'test_best_wild_hand passes'

# ------------------
# Provided Functions
# 
# You may want to use some of the functions which
# you have already defined in the unit to write 
# your best_hand function.

def hand_rank(hand):
    "Return a value indicating the ranking of a hand."
    ranks = card_ranks(hand) 
    if straight(ranks) and flush(hand):
        return (8, max(ranks))
    elif kind(4, ranks):
        return (7, kind(4, ranks), kind(1, ranks))
    elif kind(3, ranks) and kind(2, ranks):
        return (6, kind(3, ranks), kind(2, ranks))
    elif flush(hand):
        return (5, ranks)
    elif straight(ranks):
        return (4, max(ranks))
    elif kind(3, ranks):
        return (3, kind(3, ranks), ranks)
    elif two_pair(ranks):
        return (2, two_pair(ranks), ranks)
    elif kind(2, ranks):
        return (1, kind(2, ranks), ranks)
    else:
        return (0, ranks)
    
def card_ranks(hand):
    "Return a list of the ranks, sorted with higher first."
    ranks = ['--23456789TJQKA'.index(r) for r, s in hand]
    ranks.sort(reverse = True)
    return [5, 4, 3, 2, 1] if (ranks == [14, 5, 4, 3, 2]) else ranks

def flush(hand):
    "Return True if all the cards have the same suit."
    suits = [s for r,s in hand]
    return len(set(suits)) == 1

def straight(ranks):
    """Return True if the ordered 
    ranks form a 5-card straight."""
    return (max(ranks)-min(ranks) == 4) and len(set(ranks)) == 5

def kind(n, ranks):
    """Return the first rank that this hand has 
    exactly n-of-a-kind of. Return None if there 
    is no n-of-a-kind in the hand."""
    for r in ranks:
        if ranks.count(r) == n: return r
    return None

def two_pair(ranks):
    """If there are two pair here, return the two 
    ranks of the two pairs, else None."""
    pair = kind(2, ranks)
    lowpair = kind(2, list(reversed(ranks)))
    if pair and lowpair != pair:
        return (pair, lowpair)
    else:
        return None 

```