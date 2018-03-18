
# Data analysis exercise by Tianyi Wu

Contact Info: tiw206@ucsd.edu | +1(858)-265-7002
***
## Problem 1
Start by making a list of all of the nested named fields that appear in any record. Concatenate nested field names using a period '.' to defind named fields for nested records. Present the list in alphabetical order. For example, if our data file contained the following

        {"name": "Jane Doe", "address": {"personal": {"street": "123 Main St.", "city": "Springfield"}}}
        {"name": "John Doe", "email": "johndoe@example.com"}
        {"name": {"first": "Anne", "last": "Smyth"}, "phone": "123-245-7890"}

    then the ordered list of fields would be

        ["address.personal.city", "address.personal.street", "email", "name", "name.first", "name.last", "phone"]

Note that a top-level named field such as "name" could contain either text or a nested JSON object. Thus "name" and "name.first" could both exists as separate fields in your list.
***
## Solution:
I used a dictionary to store the list of fields as key and respected values in a list and iterate through the json data recursively to generate list name such as "address.personal.street".

The code is as belows:


```python
import json
import gzip
from collections import defaultdict

dict = defaultdict(list)
with gzip.open("data/ida_wrangling_exercise_data.2017-02-13.jsonl.gz", "r") as f:
#     data = f.readline()
#     print(len(data))
    for line in f:
        line = line.decode('utf-8')
        while True:
            try:
                jfile = json.loads(line)
                break
            except ValueError:
                line += next(f)
        def recursive_read(jfile, dict, key, dict_key):
            if isinstance(jfile[key], str):
                dict[dict_key].append(jfile[key])
            else:
                for subkey in jfile[key]: # 
                    recursive_read(jfile[key], dict, subkey, key+"."+subkey)
        for key in jfile:
            recursive_read(jfile, dict, key, key)
print(sorted(dict.keys()))
```

    ['address', 'address.city', 'address.state', 'address.street', 'address.zip', 'dob', 'email', 'id', 'name', 'name.firstname', 'name.lastname', 'name.middlename', 'phone', 'record_date', 'ssn']


## Problem 2
Answer the following questions for each field in your list from question 1.

        - What percentage of the records contain the field?
        - What are the five most common values of the field?
    
In the example data above, the field "address.personal.city" occurs in 1 out of 3 records so the fraction of records containing that field is 1/3. The exact percentage is 100/3. For this exercise, please approximate numeric answers to a reasonable precision such as 33.3%. Note that the field "name" occurs in the first and second record but not in the third record where the fields "name.first" and "name.last" are present. Thus, the "name" field occurs in 2/3 or roughly 66.6% of the records in the example above.
***
## Solution:

To compute the statistics of the percentage of records for each field, first count total length of records and then compute the length of each field, then formalizing it in hundred percentage. To visualize the result more vividly, I used pandas.dataframe to show the statistics.


```python
# Using the dict generated, first we will compute the total length of the records, using the max function in Python.
from collections import Counter
import pandas as pd
dict = sorted(dict.items(), key = lambda x: x[0])
nums_records = max([len(value) for key,value in dict])
print(nums_records)
```

    150000



```python
res = defaultdict(list)
for key, value in dict:
    res['count percentage'].append('{:.2%}'.format(float(len(value))/nums_records))
    counter = Counter(value)
    comm = [item for (item, _) in counter.most_common(5)]
    res['5 most common items'].append(comm)
statistics = pd.DataFrame.from_dict(res)
statistics.index = [v for (v,u) in dict]
statistics['count percentage']
```




    address             50.13%
    address.city        40.82%
    address.state       40.82%
    address.street      40.82%
    address.zip         40.82%
    dob                 95.92%
    email               87.25%
    id                 100.00%
    name                28.71%
    name.firstname      70.00%
    name.lastname       70.00%
    name.middlename     29.11%
    phone               93.51%
    record_date        100.00%
    ssn                 94.96%
    Name: count percentage, dtype: object




```python
statistics['5 most common items']
# you can select any item like this: statistics['5 most common items']['address']
```




    address            [2653 Darius Bridge\nChristopherfort, IN 43026...
    address.city       [New Michael, Lake Michael, East Michael, Port...
    address.state                                   [NC, DC, MI, MD, OH]
    address.street     [3415 Kenneth Trafficway Apt. 790, 36702 Jessi...
    address.zip                      [53097, 99528, 50032, 17086, 98018]
    dob                [1950-11-13, 1963-01-25, 1989-08-30, 1963-11-3...
    email              [kwilliams@yahoo.com, qsmith@gmail.com, asmith...
    id                 [cf3bb5a2afb64d52a87b57f5bd9592f4, c1475fde161...
    name               [David Smith, John Smith, Michael Smith, Micha...
    name.firstname               [Michael, David, James, Jennifer, John]
    name.lastname               [Smith, Johnson, Williams, Brown, Jones]
    name.middlename              [Michael, David, Jennifer, James, John]
    phone              [1-298-341-1025, (637)303-9079, 1-658-906-1808...
    record_date        [2003-05-23T16:21:08, 2003-09-08T05:27:32, 200...
    ssn                [xxx-xx-6568, xxx-xx-2156, xxx-xx-2027, xxx-xx...
    Name: 5 most common items, dtype: object



## Problem 3
How many distinct first names appear in this data set? Explain your procedure for identifying distinct first names.
***
## Solution:
To compute the distinct first name, I first retrieve from the dictionary containing first names, including the fields with "name.firstname" and the first part in the "name" field. And I count the unique values from both fields for the unique first names.


```python
new_dict = defaultdict(list)
for key,value in dict:
    new_dict[key] = value

first_name = set(new_dict['name.firstname'])
for value in new_dict['name']:
    first = value.split(' ')[0]
    first_name.add(first)
print('Count of unique first name: %d'%len(first_name))
```

    Count of unique first name: 695


## Problem 4
How many distinct street names appear in this data set? Explain your procedure for identifying distinct street names.
***
## Solution:
The street name is the first 3 strings in the "address.street" field. I split the data by empty space and use only the first three items to form the street names and count the unique values using a set().


```python
streets = set()
for item in set(new_dict['address.street']):
    res = ' '.join(item.split(' ')[:3])
    streets.add(res)
print('Count of unique street name: %d'%len(streets))
```

    Count of unique street name: 61230


## Problem 5
What are the 5 most common US area codes in the phone number field? Explain your approach to identify the US area codes in this data set.
***
## Solution:
The valid US area codes is any 3 digit number between 200 and 999. The task is to identify the area code at the correct location and check the validity before couting and output the top 5 ones.

There are different types of US codes in the dataset. From my observation, there are 5 forms:
- 1-688-873-4927x90906 
- 02996289892
- +44(3)7767821923
- 170.324.6181
- (052)824-4446

The idea is to first identify each pattern, using split functions in python. During the first data wrangling, if there is a unknown type identified, I will print out the example and do analysis based on that form.

For instance, to identify the area codes, the answer to the above examples should be :

- 1-688-873-4927x90906  -> US area code 688
- 02996289892           -> US area code 299, ignore 0
- +44(3)7767821923      -> UK number, ignore 
- 170.324.6181          -> US area code 170
- (052)824-4446         -> Invalid code, since the area code should be between 200 and 999



```python
area_code = []
for data in new_dict['phone']:
    if '-' in data:
        res = data.split('-')
        if len(res) > 2:
            code = int(res[1])
            area_code.append(code)
    elif '(' in data:
        code = int(a[a.index("(") + 1:a.rindex(")")])
        area_code.append(code)
    elif '.' in data:
        res = data.split('.')
        if len(res) > 1:
            code = int(res[0])
            area_code.append(code)
    else:
        area_code.append(int(data[1:4]))
res = []
for f in area_code:
    # filter by the validity of area codes
    if f >200 and f <999:
        res.append(f)
print('Top 5 frequent US area code:', [u for (u,_) in Counter(res).most_common(5)])         
```

    Top 5 frequent US area code: [838, 687, 996, 436, 538]

