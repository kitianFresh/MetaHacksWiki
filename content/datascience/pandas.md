---
title: "pandas"
date: 2017-04-02 10:59
---



## 
遍历 pandas.DataFrame 所有行的方式

[DataFrame.iterrows()](http://pandas.pydata.org/pandas-docs/stable/generated/pandas.DataFrame.iterrows.html)
```
for index, row in df.iterrows():
    print row["c1"], row["c2"]
```

[DataFrame.itertuples()](http://pandas.pydata.org/pandas-docs/stable/generated/pandas.DataFrame.itertuples.html)
```
for row in df.itertuples(index=True, name='Pandas'):
    print getattr(row, "c1"), getattr(row, "c2")
```

## pandas
```python
# To select rows whose column value equals a scalar, some_value, use ==:

# df.loc[df['column_name'] == some_value]
# To select rows whose column value is in an iterable, some_values, use isin:

# df.loc[df['column_name'].isin(some_values)]
# Combine multiple conditions with &:

# df.loc[(df['column_name'] == some_value) & df['other_column'].isin(some_values)]
# To select rows whose column value does not equal some_value, use !=:

# df.loc[df['column_name'] != some_value]
# isin returns a boolean Series, so to select rows whose value is not in some_values, negate the boolean Series using ~:

# df.loc[~df['column_name'].isin(some_values)]


df = pd.read_csv('similar_reduced_by_gb_name.csv')

print df.head()
print df.tail()

# print df[df['similar_names'] == '[]'].count()
# df = df[df['similar_names'] != '[]']
# df.to_csv('similar_reduced_by_gb_name.csv', index=False)

# ValueError: cannot index with vector containing NA / NaN values
# df_A = df.loc[df.icd_code.str.contains('A')]
```

## python dump json file 
将 dict 直接 dump 到文件中, 会出现 **json.dump　的　ensure_ascii 会把非ascii码变通过转义字符的形式写出去**，因此会出现　`\uXXXX`　字符串;
因此如果需要 `.json` 文件看到正确的字符串, 需要先把 dict 变成 Unicode str, 然后以写文件的形式将字符串写出即可;

```python
def save_by_gb_name(self, path='similar_by_gb_name.json'):
    # json.dump　的　ensure_ascii 会把非ascii码变通过转义字符的形式写出去，因此会出现　\uXXXX　字符串
    #If ensure_ascii is true (the default), all non-ASCII characters in the output are escaped with \uXXXX sequences, and the result is a str instance consisting of ASCII characters only. If ensure_ascii is False, some chunks written to fp may be unicode instances. This usually happens because the input contains unicode strings or the encoding parameter is used. Unless fp.write() explicitly understands unicode (as in codecs.getwriter) this is likely to cause an error.
    # 这里可以先通过　dumps 变成 python unicode string,然后把　unicode string 写入文件即可
    with io.open(path, 'w') as json_file:
        unicode_str = json.dumps(self.results, ensure_ascii=False, indent=4)
        json_file.write(unicode_str)
    
    df = pd.DataFrame(self.results)
    # print df.head()
    # print df.tail()
    df.to_csv('similar_by_gb_name.csv', index=False, encoding='utf-8')
```

## pandas
```python
df = pd.read_csv('similar_by_gb_name.csv')

df = df[df['similar_names'] != '[]']
df['icd_code'] = df['icd_code'].apply(lambda x: x.upper())

df.to_csv('similar_gb_name/similar_reduced_by_gb_name.tmp.csv', index=False)

# ValueError: cannot index with vector containing NA / NaN values
# df_A = df.loc[df.icd_code.str.contains('A')]


# 不能进行比较大小  
# df_A = df[df.icd_code.str[0] >= 'A00' & df.icd_code.str[0] <= 'B99']
# print df.dtypes


df = pd.read_csv('similar_gb_name/similar_reduced_by_gb_name.tmp.csv')
class_maps = {
    'A00-B99': 'Certain infectious and parasitic diseases',
    'C00-D49': 'Neoplasms',
    'D50-D89': 'Diseases of the blood and blood-forming organs and certain disorders involving the immune mechanism',
    'E00-E89': 'Endocrine, nutritional and metabolic diseases',
    'F01-F99': 'Mental, Behavioral and Neurodevelopmental disorders',
    'G00-G99': 'Diseases of the nervous system',
    'H00-H59': 'Diseases of the eye and adnexa',
    'H60-H95': 'Diseases of the ear and mastoid process',
    'I00-I99': 'Diseases of the circulatory system',
    'J00-J99': 'Diseases of the respiratory system',
    'K00-K95': 'Diseases of the digestive system',
    'L00-L99': 'Diseases of the skin and subcutaneous tissue',
    'M00-M99': 'Diseases of the musculoskeletal system and connective tissue',
    'N00-N99': 'Diseases of the genitourinary system',
    'O00-O9A': 'Pregnancy, childbirth and the puerperium',
    'P00-P96': 'Certain conditions originating in the perinatal period',
    'Q00-Q99': 'Congenital malformations, deformations and chromosomal abnormalities',
    'R00-R99': 'Symptoms, signs and abnormal clinical and laboratory findings, not elsewhere classified',
    'S00-T88': 'Injury, poisoning and certain other consequences of external causes',
    'V00-Y99': 'External causes of morbidity',
    'Z00-Z99': 'Factors influencing health status and contact with health services'
}

# 必须要转换成字符串, 否则会被认为是 float
# indexer = df['icd_code'].apply(lambda x: x[0:4] >= 'A00' and x[0:4] <= 'B99')
# indexer = df['icd_code'].astype(str).apply(lambda x: x[0:4] >= 'A00' and x[0:4] <= 'B99')
# df_A = df.loc[indexer]
s = 0
for k, v in class_maps.items():
    low, high = k.split('-')
    indexer = df['icd_code'].astype(str).apply(lambda x: x[0:4] >= low and x[0:4] <= high)
    df_temp = df.loc[indexer]
    # print len(df_temp.index)
    s += len(df_temp.index)
    df_temp.to_csv('similar_gb_name/' + k + '_similar_by_gb_name.csv', index=False)
print 'sum: ', s



# keep = [] #hold all the rows you want to keep
# for key in frame_dict.keys():
#     frame = frame_dict[key]
#     keep.append(
#         frame[frame['A'].astype(str).str.contains('^\d\d02', regex=True)].copy()
#     ) #append the rows matching regex for start of word (^), digit (\d), digit (\d), 02 
# final = pd.concat(keep) #concatenate the matching rows

```