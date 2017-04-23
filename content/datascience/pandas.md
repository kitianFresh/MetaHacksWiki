---
title: "pandas"
date: 2017-04-02 10:59
---

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


# 不能进行比较大小  
# df_A = df[df.icd_code.str[0] >= 'A00' & df.icd_code.str[0] <= 'B99']
print df.dtypes


# A00-B99  Certain infectious and parasitic diseases
# C00-D49  Neoplasms
# D50-D89  Diseases of the blood and blood-forming organs and certain disorders involving the immune mechanism
# E00-E89  Endocrine, nutritional and metabolic diseases
# F01-F99  Mental, Behavioral and Neurodevelopmental disorders
# G00-G99  Diseases of the nervous system
# H00-H59  Diseases of the eye and adnexa
# H60-H95  Diseases of the ear and mastoid process
# I00-I99  Diseases of the circulatory system
# J00-J99  Diseases of the respiratory system
# K00-K95  Diseases of the digestive system
# L00-L99  Diseases of the skin and subcutaneous tissue
# M00-M99  Diseases of the musculoskeletal system and connective tissue
# N00-N99  Diseases of the genitourinary system
# O00-O9A  Pregnancy, childbirth and the puerperium
# P00-P96  Certain conditions originating in the perinatal period
# Q00-Q99  Congenital malformations, deformations and chromosomal abnormalities
# R00-R99  Symptoms, signs and abnormal clinical and laboratory findings, not elsewhere classified
# S00-T88  Injury, poisoning and certain other consequences of external causes
# V00-Y99  External causes of morbidity
# Z00-Z99  Factors influencing health status and contact with health services

indexer = df['icd_code'].astype(str).apply(lambda x: x[0:4] >= 'A00' and x[0:4] <= 'B99')
df_A = df.loc[indexer]


# keep = [] #hold all the rows you want to keep
# for key in frame_dict.keys():
#     frame = frame_dict[key]
#     keep.append(
#         frame[frame['A'].astype(str).str.contains('^\d\d02', regex=True)].copy()
#     ) #append the rows matching regex for start of word (^), digit (\d), digit (\d), 02 
# final = pd.concat(keep) #concatenate the matching rows

df_A.to_csv('A_similar_by_gb_name.csv')
