---
layout: post
title: "Applying Mathematics - Assessment 2"
tags:
    - python
    - notebook
---
All questions are (number)(letter), so to find question 2a, just do
&lt;Ctrl-F&gt;2a

Some standard library imports

**In [3]:**

{% highlight python %}
from io import StringIO
from functools import wraps
from functools import partial
from operator import itemgetter
from collections import defaultdict
from csv import DictReader, DictWriter
{% endhighlight %}

Imports for web requests, and data manipulation and display

**In [4]:**

{% highlight python %}
# web requests
import requests

# to display and process data
%pylab inline
import ipy_table as pyt
import prettyplotlib as ppl
{% endhighlight %}

    Populating the interactive namespace from numpy and matplotlib
    

Pull in the survey data

**In [5]:**

{% highlight python %}
r = requests.get(
    url='https://docs.google.com/spreadsheet/ccc',
    params={
        'key': '0ApTk7F_aw6vqdGE4b0VLZTBkMmwyZk5RNV9lbC1Nb1E',
        'output': 'csv'
    }
)
    
data = list(DictReader(StringIO(r.text)))
{% endhighlight %}

Some useful functions

**In [6]:**

{% highlight python %}
def surveys_for(field, value):
    return filter(lambda r: r[field] == value, data)


surveys_for_catagory = partial(surveys_for, 'Occupation Catagory')
for_gender = partial(surveys_for, 'Gender')


def for_field(field, spec=None):
    return list(map(itemgetter(field), spec or data))

def ints(seq):
    return list(map(int, seq))

def calc_mus(data):
    rows = []
    for row in data:
        cur_total = 0
        for field in ACTIVITIES:
            cur_total += int(row[field])
        row['MUS'] = cur_total
    return data

{% endhighlight %}

And some useful constants;

**In [7]:**

{% highlight python %}
ACTIVITIES = ['Recording Data', 'Interpreting Data', 'Using Operations', 'Estimating', 'Using a Calculator', 'Problem Solving']
CATAGORIES = {
    'BU': 'Business',
    'HS': 'Health Sciences',
    'HU': 'Humanities'
}
{% endhighlight %}

Calculate and add the MUS data, sort the genders

**In [8]:**

{% highlight python %}
data = calc_mus(data)

females = list(for_gender('F'))
males = list(for_gender('M'))
{% endhighlight %}

## 2a

**In [9]:**

{% highlight python %}
males_mus = for_field('MUS', males)

print('Male average:', np.mean(males_mus))
{% endhighlight %}

    Male average: 7.78571428571
    

## 2b

**In [10]:**

{% highlight python %}
females_mus = for_field('MUS', females)

print('Female average:', np.mean(females_mus))
{% endhighlight %}

    Female average: 8.25
    

## 2c

Judging by these results, I would say that females use math more than males.
Perhaps because they tend towards business roles?

## 3a

**In [11]:**

{% highlight python %}
IMPORTANCE = {
    'V': 2,  # Very important
    'I': 1,  # Important
    'N': 0   # Not important
}
{% endhighlight %}

## 3b

**In [12]:**

{% highlight python %}
male_importance = map(itemgetter('Importance Of Math'), males)
male_importance = list(map(IMPORTANCE.get, male_importance))
male_result = round(np.mean(male_importance), 2)
{% endhighlight %}

**In [13]:**

{% highlight python %}
female_importance = map(itemgetter('Importance Of Math'), females)
female_importance = list(map(IMPORTANCE.get, female_importance))
female_result = round(np.mean(female_importance), 2)
{% endhighlight %}

**In [14]:**

{% highlight python %}
pyt.make_table([
    ('Average female importantance of math', female_result),
    ('Average male importantance of math', male_result)
])
{% endhighlight %}




<table border="1" cellpadding="3" cellspacing="0"  style="border:1px solid black;border-collapse:collapse;"><tr><td>Average&nbspfemale&nbspimportantance&nbspof&nbspmath</td><td>1.31</td></tr><tr><td>Average&nbspmale&nbspimportantance&nbspof&nbspmath</td><td>1.36</td></tr></table>



It would seem that males rate maths as being more important for their jobs than
females.

## 4a

Perhaps the older they are, the more used to using maths directly they are? As
opposed through an abstraction such as a computer? Might this affect how much
they perceive themselves as relying on math?

Essentially;
> Group all people by age group,
> determine average perceived reliance on mathematics?

## 4b

**In [26]:**

{% highlight python %}
# sort people into their age groups
ages = defaultdict(list)
for row in data:
    ages[row['Age']].append(row)

# determine averages of each age group, round to 2 decimal places
rating_to_age = {}
for age, people in ages.items():
    rating = for_field('MUS', people)
    rating_to_age[age] = round(np.mean(rating), 2)

# sort by age range
table_rating_to_age = sorted(rating_to_age.items(), key=lambda r: r[0])

pyt.make_table([['Age Range', 'Rating']] + table_rating_to_age)
pyt.apply_theme('basic')
{% endhighlight %}




<table border="1" cellpadding="3" cellspacing="0"  style="border:1px solid black;border-collapse:collapse;"><tr><td  style="background-color:LightGray;"><b>Age&nbspRange</b></td><td  style="background-color:LightGray;"><b>Rating</b></td></tr><tr><td  style="background-color:Ivory;">15-19</td><td  style="background-color:Ivory;">11.0</td></tr><tr><td  style="background-color:AliceBlue;">20-29</td><td  style="background-color:AliceBlue;">7.4</td></tr><tr><td  style="background-color:Ivory;">30-39</td><td  style="background-color:Ivory;">9.43</td></tr><tr><td  style="background-color:AliceBlue;">40-49</td><td  style="background-color:AliceBlue;">7.7</td></tr><tr><td  style="background-color:Ivory;">50-59</td><td  style="background-color:Ivory;">6.5</td></tr></table>



**In [43]:**

{% highlight python %}
# grab the names and values in the appropriate order
rating_to_age_names = list(rating_to_age.keys())
rating_to_age_names = sorted(rating_to_age_names)
rating_to_age_values = list(map(rating_to_age.get, rating_to_age_names))

index = np.arange(len(rating_to_age))
bar_width = 0.35

ppl.bar(index, rating_to_age_values, bar_width)
plt.title('Age range to average rating')
plt.xticks(index + bar_width, rating_to_age_names)
plt.ylabel('Rating')
None
{% endhighlight %}


![png](data:image/png;base64,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)


## 5a

**In [29]:**

{% highlight python %}
mus = for_field('MUS', data)

ppl.hist(mus)
plt.title('Mathematical Usage Scores')
plt.ylabel('Freq')
plt.xlabel('Score ranges')
None
{% endhighlight %}


![png](data:image/png;base64,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)


## 5b

The histogram shows a large clumping of people perceiving their jobs as
requiring mathematics in high quantity, with another clump showing people who do
not perceive it as being as important.

We can conclude that people in Business, the Humanities, and the Health Sciences
perceive their jobs as requiring maths in quantity.

## 6a

**In [17]:**

{% highlight python %}
# calculate the sums of the different catagories
activities_to_use = {}
for field in ACTIVITIES:
    values = for_field(field)
    activities_to_use[field] = sum(ints(values))

# sort em by sum
activities_to_use = sorted(activities_to_use.items(), key=lambda r: r[1], reverse=True)

pyt.make_table([['Activity', 'Sum']] + activities_to_use)
pyt.apply_theme('basic')
{% endhighlight %}




<table border="1" cellpadding="3" cellspacing="0"  style="border:1px solid black;border-collapse:collapse;"><tr><td  style="background-color:LightGray;"><b>Activity</b></td><td  style="background-color:LightGray;"><b>Sum</b></td></tr><tr><td  style="background-color:Ivory;">Recording&nbspData</td><td  style="background-color:Ivory;">51</td></tr><tr><td  style="background-color:AliceBlue;">Interpreting&nbspData</td><td  style="background-color:AliceBlue;">44</td></tr><tr><td  style="background-color:Ivory;">Estimating</td><td  style="background-color:Ivory;">38</td></tr><tr><td  style="background-color:AliceBlue;">Problem&nbspSolving</td><td  style="background-color:AliceBlue;">37</td></tr><tr><td  style="background-color:Ivory;">Using&nbspOperations</td><td  style="background-color:Ivory;">37</td></tr><tr><td  style="background-color:AliceBlue;">Using&nbspa&nbspCalculator</td><td  style="background-color:AliceBlue;">34</td></tr></table>



## 6b

Unsurprisingly, the recording of data is the most prevalent form of mathematics
in the real world, as the simple act of recording data is perfomed everywhere,
whilst using a calculator seems surprising uncommon.

## 7

**In [18]:**

{% highlight python %}
# calculate the average MUS values for the different profesions
categories = {}
for code in CATAGORIES.keys():
    surveys = surveys_for('Occupation Catagory', code)
    mus_values = for_field('MUS', surveys)

    # get the appropriate name for the catagory
    cat_name = CATAGORIES.get(code)
    categories[cat_name] = round(
         np.mean(mus_values), 2
    )

# sort by MUS average
categories = sorted(categories.items(), key=lambda r: r[1], reverse=True)
    
pyt.make_table([['Category', 'Rating']] + categories)
pyt.apply_theme('basic')
{% endhighlight %}




<table border="1" cellpadding="3" cellspacing="0"  style="border:1px solid black;border-collapse:collapse;"><tr><td  style="background-color:LightGray;"><b>Category</b></td><td  style="background-color:LightGray;"><b>Rating</b></td></tr><tr><td  style="background-color:Ivory;">Humanities</td><td  style="background-color:Ivory;">9.44</td></tr><tr><td  style="background-color:AliceBlue;">Business</td><td  style="background-color:AliceBlue;">8.43</td></tr><tr><td  style="background-color:Ivory;">Health&nbspSciences</td><td  style="background-color:Ivory;">5.8</td></tr></table>



It would seem that the humanities use the most math.

## Postscript

This code generates a final `.csv` with the MUS included for all the records.

**In [19]:**

{% highlight python %}
keys = [
    'Occupation', 
    'Occupation Catagory', 
    'Gender', 
    'Age', 
    'Years Uni', 
    'Importance Of Math', 
    'Recording Data', 
    'Interpreting Data', 
    'Using Operations', 
    'Estimating', 
    'Using a Calculator', 
    'Problem Solving',
    'MUS'
]

with open('output.csv', 'w') as fh:
    w = DictWriter(fh, keys)
    
    w.writeheader()
    w.writerows(data)
{% endhighlight %}