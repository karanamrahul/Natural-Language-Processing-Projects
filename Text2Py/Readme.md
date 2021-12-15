
##  Text2Py : A transformer based model which will generate python code snippets using natural language considering proper indentations.

### Dataset : We have a curated dataset of 4600+ examples which were taken from all the assignments during END & EVA Course.

You can download the dataset using this [link](https://drive.google.com/file/d/1rHb0FQ5z5ZpaY2HpyFGY6CeyDG0kTLoO/view)

Dataset Examples

```

  Question: 
  # Write a Python program to calculate distance between two points using latitude and longitude.
  
  Answer:
from math import radians, sin, cos, acos

print("Input coordinates of two points:")
slat = radians(float(input("Starting latitude: ")))
slon = radians(float(input("Ending longitude: ")))
elat = radians(float(input("Starting latitude: ")))
elon = radians(float(input("Ending longitude: ")))

dist = 6371.01 * acos(sin(slat)*sin(elat) + cos(slat)*cos(elat)*cos(slon - elon))
print("The distance is %.2fkm." % dist)

Question: 
# Write a Python class to convert a roman numeral to an integer.

Answer:
class Solution:
    def roman_to_int(self, s):
        rom_val = {'I': 1, 'V': 5, 'X': 10, 'L': 50, 'C': 100, 'D': 500, 'M': 1000}
        int_val = 0
        for i in range(len(s)):
            if i > 0 and rom_val[s[i]] > rom_val[s[i - 1]]:
                int_val += rom_val[s[i]] - 2 * rom_val[s[i - 1]]
            else:
                int_val += rom_val[s[i]]
        return int_val



```


Model: I use seq2seq transformer architecture which has encoder and decoder which will convert engish language to python code snippets.

## Dataset Preprocessing

By looking at the above examples from the dataset one can understand that preprocessing need to be done as the code snippets have different indentations,quotes,whitespaces.
The question starts with # or # 01 and the data below the question will be the answer.

First we need to make question answer pairs by considering data starting with # as a question and the line after that will be considered as answer.

Then we need to tokenize both english text and python code using different tokenizer as both have different syntax.

We need to check with the question pairs as some question begin with # and some begin with # 25 | they start directly with the question
```
def tokenize_python (line):
  points = ['<space>', ':', '<new_line>', '(', ')', '[', ']', '>', '<', '=', "'", '"', '.', '%', ',', '+', '-', '*', '\t', '{', '}', '/']

  words = []
  temp = ''
  n = len(line)
  i = 0

  while i < n:
    #print (i, line[i], temp, words)
    if line[i:i+7] == '<space>':
      if temp != '':
        words.append (temp)
        temp = ''
      words.append ('<space>')
      i = i + 7

    elif line[i:i+10] == '<new_line>':
      if temp != '':
        words.append (temp)
        temp = ''
      words.append ('<new_line>')
      i = i + 10

    elif line[i] in split_points:
      if temp != '':
        words.append (temp)
        temp = ''
      words.append (line[i])
      i += 1
    else:
      temp += line[i]
      i += 1
  return words

def tokenize_en(text):
    """
    Tokenizes English text from a string into a list of strings
    """
    return [tok.text for tok in spacy_en.tokenizer(text)]
```



Loss function

I have tried to implent several other loss functions but others failed at some point after lot of research and stress , had to fix to cross entropy loss.
```
criterion = nn.CrossEntropyLoss(ignore_index = TRG_PAD_IDX)
```



Test Loss 

```
model.load_state_dict(torch.load('model.pt'))

test_loss = evaluate(model, train_iterator, criterion)

print(f'| Test Loss: {test_loss:.3f} | Test PPL: {math.exp(test_loss):7.3f} |')

| Test Loss: 0.125 | Test PPL:   1.133 |

```


Examples generated through the trained model 

Example-1
```
problem statement:
# write a function to adds two lists element wise only if numbers are even
truth:
def adds_listevenelements(l1:list, l2:list):
    return [i+j for i, j in zip(l1,l2) if i*j%2 == 0]

prediction:
['def', '<space>', 'adds_listevenelements', '(', 'l1', ':', 'list', ',', '<space>', 'l2', ':', 'list', ')', ':', '<new_line>', '<space>', '<space>', '<space>', '<space>', 'return', '<space>', '[', 'i', '+', 'j', '<space>', 'for', '<space>', 'i', ',', '<space>', 'j', '<space>', 'in', '<space>', 'zip', '(', 'l1', ',', 'l2', ')', '<space>', 'if', '<space>', 'i', '*', 'j', '%', '2', '<space>', '=', '=', '<space>', '1', ']', '<new_line>', '<eos>']
```
#Example-2
```
problem statement:
# python function to print all time when angle between hour hand and minute
truth:
def printtime(theta):
    for hh in range(0, 12):
        for mm in range(0, 60):
            if (calcangle(hh, mm) == theta):
                print(hh, ':', mm, sep='')
                return
    print('input angle not valid.')
    return
theta = 90.0
printtime(theta)

prediction:
['def', '<space>', 'printtime', '(', 'theta', ')', ':', '<new_line>', '<space>', '<space>', '<space>', '<space>', 'for', '<space>', 'hh', '<space>', 'in', '<space>', 'range', '(', '0', ',', '<space>', '12', ')', ':', '<new_line>', '<space>', '<space>', '<space>', '<space>', '<space>', '<space>', '<space>', '<space>', 'for', '<space>', 'mm', '<space>', 'in', '<space>', 'range', '(', '0', ',', '<space>', '60', ')', ':', '<new_line>', '<space>', '<space>', '<space>', '<space>', '<space>', '<space>', '<space>', '<space>', '<space>', '<space>', '<space>', '<space>', 'if', '<space>', '(', 'calcangle', '(', 'hh', ',', '<space>', 'mm', ')', '<space>', '=', '=', '<space>', 'theta', ')', ':', '<new_line>', '<space>', '<space>', '<space>', '<space>', '<space>', '<space>', '<space>', '<space>', '<space>', '<space>', '<space>', '<space>', '<space>', '<space>', '<space>', '<space>', 'print', '(', 'hh', ',', '<space>', "'", ':', "'", ',', '<space>', 'mm', ',', '<space>', 'sep', '=', "'", "'", ')', '<new_line>', '<space>', '<space>', '<space>', '<space>', '<space>', '<space>', '<space>', '<space>', '<space>', '<space>', '<space>', '<space>', '<space>', '<space>', '<space>', '<space>', 'return', '<new_line>', '<space>', '<space>', '<space>', '<space>', 'print', '(', "'", 'input', '<space>', 'angle', '<space>', 'not', '<space>', 'valid', '.', "'", ')', '<new_line>', '<space>', '<space>', '<space>', '<space>', 'return', '<new_line>', 'theta', '<space>', '=', '<space>', '90', '.', '0', '<new_line>', 'printtime', '(', 'theta', ')', '<new_line>', '<eos>']
```

Examole-3
```
problem statement:
# write a function to return the torque when a force f is applied at angle thea and distance for axis of rotation to place force applied is r
truth:
def cal_torque(force:float,theta:float,r:float)->float:
    import math
    return force*r*math.sin(theta)

prediction:
['def', '<space>', 'cal_torque', '(', 'force', ':', 'float', ',', 'theta', ':', 'float', ',', 'r', ':', 'float', ')', '-', '>', 'float', ':', '<new_line>', '<space>', '<space>', '<space>', '<space>', 'import', '<space>', 'math', '<new_line>', '<space>', '<space>', '<space>', '<space>', 'return', '<space>', 'force', '*', 'r', '*', 'math', '.', 'sin', '(', 'theta', ')', '<new_line>', '<eos>']

```
Exampel-4
```
problem statement:
# write a program to calculate and print the volume of a cylender
truth:
r = 3
h = 5
pi = 3.14
volume = pi*(r**2)*h
print(volume)

prediction:
['r', '<space>', '=', '<space>', '3', '<new_line>', 'h', '<space>', '=', '<space>', '5', '<new_line>', 'pi', '<space>', '=', '<space>', '3', '.', '14', '<new_line>', 'volume', '<space>', '=', '<space>', 'pi', '*', 'r', '*', '2', '<new_line>', 'volume', '<space>', '=', 'r', '*', 'h', '<new_line>', 'print', '(', 'volume', ')', '<new_line>', '<eos>']
```
Example-5
```
problem statement:
# write a python function to use a predicate and return entries particition into false entries and true entries
truth:
def partition(pred, iterable):
    from itertools import filterfalse, tee
    # partition(is_odd, range(10)) --> 0 2 4 6 8   and  1 3 5 7 9
    t1, t2 = tee(iterable)
    return filterfalse(pred, t1), filter(pred, t2)

prediction:
['def', '<space>', 'partition', '(', 'pred', ',', '<space>', 'iterable', ')', ':', '<new_line>', '<space>', '<space>', '<space>', '<space>', 'from', '<space>', 'itertools', '<space>', 'import', '<space>', 'filterfalse', ',', '<space>', 'tee', '<new_line>', '<space>', '<space>', '<space>', '<space>', '#', '<space>', 'partition', '(', 'is_odd', ',', '<space>', 'range', '(', '10', ')', ')', '<space>', '-', '-', '>', '<space>', '0', '<space>', '0', '<space>', '4', '<space>', '6', '<space>', '8', '<space>', '<space>', '<space>', 'and', '<space>', '<space>', '1', '<space>', '7', '<space>', '5', '<space>', '7', '<space>', '9', '<new_line>', '<space>', '<space>', '<space>', '<space>', 't1', ',', '<space>', 't2', '<space>', '=', '<space>', 'tee', '(', 'iterable', ')', '<new_line>', '<space>', '<space>', '<space>', '<space>', 'return', '<space>', 'filterfalse', '(', 'pred', ',', '<space>', 't1', ')', ',', '<space>', 'filter', '(', 'pred', ',', '<space>', 't2', ')', '<new_line>', '<eos>']
```
Example-6
```

problem statement:
# write a function that returns sine value of the input
truth:
def sin(x:float) -> float:
    import math
    return math.sin(x)

prediction:
['def', '<space>', 'sin', '(', 'x', ':', 'float', ')', '<space>', '-', '>', '<space>', 'float', ':', '<new_line>', '<space>', '<space>', '<space>', '<space>', 'import', '<space>', 'math', '<new_line>', '<space>', '<space>', '<space>', '<space>', 'return', '<space>', 'math', '.', 'sin', '(', 'x', ')', '<new_line>', '<eos>']

```
Example-7
```
problem statement:
# write a lambda function to add two numbers
truth:
add = lambda a, b: a+b

prediction:
['add', '<space>', '=', '<space>', 'lambda', '<space>', 'a', ',', '<space>', 'b', ':', '<space>', 'a', '+', 'b', '<new_line>', '<eos>']

```
Example-8
```
problem statement:
# python code to merge dictionaries
truth:
def merge1():
    test_list1 = [{'a': 1, 'b': 4}, {'c': 10, 'd': 15},
                  {'f': 'gfg'}]
    test_list2 = [{'e': 6}, {'f': 3, 'fg': 10, 'h': 1},
                  {'i': 10}]
    print('the original list 1 is : ' + str(test_list1))
    print('the original list 2 is : ' + str(test_list2))
    for idx in range(0, len(test_list1)):
        id_keys = list(test_list1[idx].keys())
        for key in test_list2[idx]:
            if key not in id_keys:
                test_list1[idx][key] = test_list2[idx][key]
    print('the merged dictionary list : ' + str(test_list1))

prediction:
['def', '<space>', 'merge1', '(', ')', ':', '<new_line>', '<space>', '<space>', '<space>', '<space>', 'test_list1', '<space>', '=', '<space>', '[', '{', "'", 'a', "'", ':', '<space>', '1', ',', '<space>', "'", 'b', "'", ':', '<space>', '4', '}', ',', '<space>', '{', "'", 'c', "'", ':', '<space>', '10', '}', ',', '<space>', '{', "'", 'd', "'", ':', '<space>', '15', '}', ',', '<new_line>', '<space>', '<space>', '<space>', '<space>', '<space>', '<space>', '<space>', '<space>', '<space>', '<space>', '<space>', '<space>', '<space>', '<space>', '<space>', '<space>', '<space>', '<space>', '<space>', '<space>', '<space>', '{', "'", 'f', "'", ':', '<space>', '3', ',', '<space>', "'", 'c', "'", ':', '<space>', '4', '}', ']', '<new_line>', '<space>', '<space>', '<space>', '<space>', '<space>', '<space>', '<space>', '<space>', '<space>', '<space>', '<space>', 'test_list2', '<space>', '=', '<space>', '[', '{', "'", 'gfg', "'", ':', '<space>', '[', '5', ',', '<space>', '10', ',', '<space>', "'", 'is', "'", ':', '<space>', '10', ',', '<space>', "'", 'best', "'", ':', '<space>', '6', '}', ']', '<new_line>', '<space>', '<space>', '<space>', '<space>', '<space>', '<space>', '<space>', '<space>', '<space>', '<space>', '<space>', '<space>', '<space>', '<space>', '<space>', '<space>', '<space>', '<space>', '<space>', '<space>', '<space>', '<space>', '<space>', '<space>', '<space>', '<space>', '<space>', '{', "'", ':', '<space>', '{', "'", 'gfg', "'", '}', ']', '<new_line>', '<space>', '<space>', '<space>', '<space>', '<space>', '<space>', '<space>', '<space>', '<space>', '<space>', '<space>', '<space>', '<space>', '<space>', '<space>', '<space>', '<space>', '<space>', '<space>', '<space>', '<space>', '<space>', '<space>', '<space>', '<space>', '<space>', '<space>', '<space>', '{', "'", '<space>', '<space>', 'test_list2', '<space>', "'", 'e', "'", ':', '<space>', '10', ',', '<space>', '3', ',', '<space>', "'", 'fg', "'", 'i', "'", ':', '<space>', '+', '<space>', "'", ',', '<space>', "'", 'c', "'", ':', '<space>', '7', '}', ']', '<new_line>', '<space>', '<space>', '<space>', '<space>', '<space>', '<space>', '<space>', '<space>', '<space>', '<space>', '<space>', '<space>', '<space>', '<space>', '<space>', '<space>', '<space>', '<space>', '<space>', '<space>', '<space>', '<space>', '<space>', '<space>', '<space>', '<space>', '<space>', '<space>', '<space>', '<space>', '<space>', '<space>', '<space>', '<space>', '<space>', '<space>', '<space>', '<space>', '<space>', 'n', '<space>', '<space>', '<space>', '=', '<space>', '0', '<new_line>', '<space>', '<space>', '<space>', '<space>', '<space>', '<space>', '<space>', '<space>', '<space>', '<space>', '<space>', '<space>', '<space>', '<space>', '<space>', '<space>', '<space>', '<space>', '<space>', '<space>', '<space>', '<space>', '<space>', '<space>', '<space>', '<space>', '<space>', '<space>', '<space>', '<space>', '<space>', '<space>', '<space>', '<space>', '<space>', '<space>', '<space>', '<space>', '<space>', '<space>', '<space>', '<space>', '<space>', '<space>', '<space>', '<space>', '<space>', '<space>', '<space>', '<space>', '<space>', '<space>', '<space>', '<space>', '<space>', '<space>', '<space>', '<space>', '<space>', '<space>', '<space>', '<space>', '<space>', '<space>', '<space>', '<space>', '<space>', '<space>', '<space>', '<space>', '<space>', '<space>', '<space>', '<space>', '<space>', '<space>', '<space>', '<space>', '<space>', '<space>', '<space>', '<space>', '<space>', '<space>', '<space>', '5', '<space>', '<space>', '<space>', '<space>', '<space>', '<space>', '<space>', '<space>', '<space>', '<space>', '<space>', '<space>', '<space>', '<space>', '<space>', '<space>', '<space>', '<space>', '<space>', '<space>', '<space>', '<space>', '<space>', '<space>', '6', '<space>', '<space>', '<space>', '<space>', '<space>', '<space>', '<space>', '<space>', '<space>', '<space>', '<space>', '7', '<space>', '<space>', '<space>', '<space>', '<space>', '<space>', '<space>', '<space>', '<space>', '<space>', '<space>', '<space>', '<space>', '<space>', '<space>', '<space>', '<space>', '<new_line>', '<space>', '<space>', '<space>', '<space>', '<space>', '<space>', '<space>', '<space>', '<space>', '<space>', '<space>', '<space>', '<space>', '<space>', '<space>', '<space>', '<space>', '<space>', '<space>', '<space>', '<space>', '<space>', '<space>', '<space>', '<space>', '<space>', '<space>', '<space>', '<space>', '<space>', '<space>', '<space>', '<space>', '<space>', '<space>', '<space>', '<space>', '<space>', '<space>', '<space>', '<space>', '<space>', '<space>', '<space>', '<space>', '<space>', '<space>', '<space>', '<space>', '<space>', '<space>', '<space>', '<space>', '<space>', '<space>', '<space>', '<space>', '<space>', '<space>', '<space>', '<space>', '<space>', '<space>', '<space>', '<space>', '<space>', '<space>', '<space>', '<space>', '<space>', '<space>', '<space>', '<space>', '<space>', '<space>', '<space>', '<space>', '<space>', '<space>', '<space>', '<space>', '<space>', '<space>', '<space>', '<space>', '<space>', '<space>', '<space>', '<space>', '<space>', '<space>', '<space>', '<space>', '20', '<new_line>', '<space>', '<space>', '<space>', '<space>', '<space>', '<space>', '<space>', '<space>', '<space>', '<space>', '<space>', '<space>', '<space>', '<space>', '<space>', '<space>', '<space>', '<space>', '<space>', '<space>', '<space>', '<space>', '<space>', '<space>', '<space>', '<space>', '<space>', '<space>', '<space>', '<space>', '<space>', '<space>', '<space>', '<space>', '<space>', '<space>', '<space>', '<space>', '<space>', '<space>', '<space>', '<space>', '<space>', '30', '<space>', '<space>', '<space>', '<space>', '<space>', '<space>', '<space>', '<space>', '<space>', '<space>', '<space>', '<space>', '<space>', '<space>', '<space>', '<space>', '<space>', '<space>', '<space>', '<space>', '<space>', '<space>', '<space>', '<space>', '<space>', '<space>', '<space>', '<space>', '<space>', '<space>', '<space>', '<space>', '<space>', '<space>', '20', '<new_line>', '<space>', '<space>', '<space>', '<space>', '<space>', '<space>', '<space>', '<space>', '<space>', '<space>', '<space>', '<space>', '<space>', '<space>', '<space>', '<space>', '<space>', '<space>', '<space>', '<space>', '<space>', '<space>', '<space>', '<space>', '<space>', '<space>', '<space>', '<space>', '<space>', '<space>', '<space>', '<space>', '<space>', '<space>', '<space>', '<space>', '30', '<space>', '<space>', '<space>', '<space>', '<space>', '<space>', '<space>', '<space>', '<space>', '<space>', '<space>', '<space>', '<space>', '<space>', '<space>', '<space>', '<space>', '<space>', '<space>', '<space>', '<space>', '<space>', '<space>', '<space>', '<space>', '<space>', '<space>', '<space>', '30', '<space>', '<space>', '<space>', '<space>', '<space>', '<space>', '<space>', '<space>', '<space>', '<space>', '<space>', '20', '<new_line>', '<space>', '<space>', '<space>', '<space>', '<space>', '<space>', '<space>', '<space>', '<space>', '<space>', '<space>', '<space>', '17', ')', '<space>', '<space>', '<space>', '<space>', '<space>', '<space>', '<space>', '<space>', '<space>', '<space>', '30', '<space>', '<space>', '<space>', '<space>', '<space>', '<space>', '<space>', '<space>', '<space>', '<space>', '<space>', '<space>', '<space>', '<space>', '<space>', '<space>', '<space>', '<space>', '<space>', '<space>', '<space>', '<space>', '<space>', '<space>', '<space>', '<space>', '<space>', '<space>', '<space>', '<space>', '<space>', '<space>', '<space>', '<space>', '<space>', '<space>', '<space>', '<space>', '<space>', '<space>', '<space>', '<space>', '<space>', '<space>', '<space>', '30', '<space>', '<space>', '<space>', '<space>', '<space>', '<space>', '<space>', '<space>', '<space>', '<space>', '<space>', '<space>', '<space>', '<space>', '<space>', '<space>', '<space>', '<space>', '<space>', '<space>', '<space>', '<space>', '<space>', '<space>', '<space>', '<space>', '<space>', '<space>', '<space>', '<space>', '<space>', '<space>', '<space>', '<space>', '<space>', '<space>', '<space>', '<space>', '<space>', '<space>', '30', '<space>', '<space>', '<space>', '<space>', '<space>', '<space>', '<space>', '<space>', '<space>', '<space>', '<space>', '<space>', '<space>', '<space>', '<space>', '<space>', '<space>', '<space>', '<space>', '<space>', '<space>', '<space>', '10', '<new_line>', '<space>', '<space>', '<space>', '<space>', '<space>', '<space>', '<space>', '<space>', '<space>', '<space>', '<space>', '<space>', '<space>', '30', '<space>', '<space>', '<space>', '<space>', '(', '4', '<space>', '<space>', '<space>', '<space>', '<space>', '<space>', '<space>', '<space>', '<space>', '<space>', '<space>', '<space>', '<space>', '<space>', '<space>', '<space>', '<space>', '<space>', '<space>', '<space>', '<space>', '20', ')', '<space>', '<space>', '<space>', '<space>', '<space>', '<space>', '<space>', '<space>', '<space>', '<space>', '<space>', '<space>', '<space>', '<space>', '<space>', '<space>', '<space>', '<space>', '<space>', '<space>', '<space>', '<space>', '<space>', '<space>', 'del', '<space>', '<space>', '<new_line>', '<space>', '<space>', '<space>', '<space>', '<space>', '<space>', '<space>', '<space>', '<space>', '<space>', '<space>', '<space>', '<space>', '<space>', '<space>', '<space>', '<space>', '<space>', '<space>', '<space>', '<space>', '<space>', '<space>', '<space>', '<space>', '<space>', '<space>', '<space>', '<space>', '<space>', '<space>', '<space>', '<space>', '<space>', '<space>', '<space>', '<space>', '<space>', '<space>', '<space>', '<space>', '<space>', '<space>', '<space>', '<space>', '<space>', '<space>', '<space>', '<space>', '<space>', '<space>', '<space>', '<space>', '<space>', '<space>', '30', '<space>', '<space>', '<space>', '<space>', '<space>', '<space>', '<space>', '<space>', '<space>', '<space>', '<space>', '<space>', '<space>', '<space>', '<space>', '<space>', '<space>', '<space>', '<space>', '<space>', '<space>', '<space>', '<space>', '<space>', '<space>', '<space>', '<space>', '<space>', '<space>', '<space>', '<space>', '<space>', '<space>', '<space>', '30', ',', '<space>', '<space>', '<space>', '<space>', '<space>', '<space>', '<space>', '<space>', '<space>', '<space>', '<space>', '<space>', '<space>', '<space>', '<space>', '<space>', '<space>', '<space>', '<space>', '<space>', '<space>', '<space>', '<space>', '60', '<space>', '<space>', '<space>', '<space>', '<space>', '(', 'test_list1', '[', '5', ',', '<space>', '<space>', '<space>', '<space>', '<space>', '<space>', '<space>', '<space>', '<space>', '<space>', '<space>', '<space>', '<space>', '<space>', '<space>', '<space>', '<space>', '<space>', '<space>', '<space>', '<space>', '<space>', '<space>', '<space>', '<space>', '<space>', '<space>', '<space>', '<space>', '<space>', '<space>', '<space>', '<space>', '<space>', '<space>', '<space>', '<space>', '<space>', '<space>', '<space>', '<space>', '<space>', '<space>', '<space>', '<space>', '<space>', '<space>', '<space>', '<space>', '<space>', '<space>', '<space>', '<space>', '<space>', '<space>', '<space>', '<space>', '<space>', '<space>', '<space>', '<space>', '<space>', '<space>', '<space>', '<space>', '<space>', '<space>', '<space>', '<space>', '<space>', '<space>', '<space>', '<space>', '<space>', '<space>', '<space>', '<space>', '<space>', '<space>', '<space>', '<space>', '<space>', '<space>', '<space>', '<space>', '<space>', '<space>', '<space>', '<space>', '<space>', '<space>', '<space>', '<space>', '<space>', '<space>', '<space>', '<space>', '<space>', '<space>', '<space>', '<space>', '<space>', '<space>', '<space>', '<space>', '<space>', '<space>', '<space>', '<space>', '<space>', '<space>', '<space>', '<space>', '<space>', '<space>', '<space>', '<space>', '<space>', '<space>', '<space>', '<space>', '<space>', '<space>', '<space>', '<space>', '<space>', '30', ',', '<space>', '<space>', '<space>', '<space>', '<space>', '<space>', '<space>', '<space>', '<space>', '<space>', '<space>', '<space>', '<space>', '<space>', '<space>', '<space>', '<space>', '<space>', '<space>', '<space>', '<space>', '<space>', '<space>', '<space>', '<space>', '<space>', '<space>', '<space>', '<space>', '<space>', '<space>', '<space>', '<space>', '<space>', '<space>', '<space>', '<space>', '<space>', '<space>', '<space>', '<space>', '<space>', '<space>', '<space>', '<space>', '<space>', '<space>', '<space>', '<space>', '<space>', '<space>', '<space>', '<space>', '<space>', '<space>', '<space>', '20', ')', '<space>', '<space>', '<space>', '<space>', '<space>', '<space>', '<space>', '<space>', '<space>', '<space>', '<space>', '<space>', '<space>', '<space>', '<space>', '<space>', '<space>', '<space>', '<space>', '<space>', '<space>', '<space>', '<space>', '<space>', '<space>', '<space>', '<space>', '<space>', '<space>', '<space>', '<space>', '<space>', '<space>', '<space>', '<space>', '<space>', '<space>', '(', "'", 'e', "'", 'e', '<space>', '<space>', '<space>', '<space>', '<space>', '<space>', '<space>', '<space>', '<space>', '<space>', '<space>', '<space>', '<space>', '<space>', '<space>', '<space>', '<space>', '<space>', '<space>', '<space>', '<space>', '<space>', '<space>', '<space>', '<space>', '<space>', '<space>', '<space>', '<space>', '<space>', '<space>', '<space>', '<space>', '<space>', '<space>', '<space>', '<space>', '<space>', '<space>', '<space>', '<space>', '<space>', '<space>', '(', '<space>', '<space>', '<space>', '<space>', '<space>', '<space>', '<space>', '<space>', '<space>', '<space>', '<space>', '<space>', '<space>', '<space>', '<space>', '<space>', '<space>', '<space>', '<space>', '<space>', '<space>', '<space>', '<space>', '<space>', '<space>', '<space>', '<space>', '<space>', '<space>', '<space>', '<space>', '<space>', '<space>', '<space>', '<space>', '<space>', '<space>', '<space>', '<space>', '<space>', '<space>', '<space>', '<space>', '<space>', '<space>', '<space>', '<space>', '<space>', '<space>', '<space>', '<space>', '<space>', '<space>', '<space>', '<space>', '<space>', '9', ',', '<space>', '<space>', '<space>', '<space>', '<space>', '<space>', '20', ')', ')', '<new_line>', '<eos>']


```
Example-9
```
problem statement:
# write a python function to return nth item or a default value
truth:
def nth(iterable, n, default=none):
    from itertools import islice
    return next(islice(iterable, n, none), default)

prediction:
['def', '<space>', 'nth', '(', 'iterable', ',', '<space>', 'n', ',', '<space>', 'default', '=', 'none', ')', ':', '<new_line>', '<space>', '<space>', '<space>', '<space>', 'from', '<space>', 'itertools', '<space>', 'import', '<space>', 'islice', '<new_line>', '<space>', '<space>', '<space>', '<space>', 'return', '<space>', 'next', '(', 'islice', '(', 'iterable', ',', '<space>', 'n', ',', '<space>', 'none', ')', ',', '<space>', 'default', ')', '<new_line>', '<eos>']

```
Example-10
```
problem statement:
# write a functin that returns the lcm of two input numbers
truth:
def lcm(a, b):
    if a>b:
        min_ = a
    else:
        min_ = b
    while true:
        if min_%a==0 and min_%b==0:
            break
        min_+=1
    return min_

prediction:
['def', '<space>', 'lcm', '(', 'a', ',', 'b', ')', ':', '<new_line>', '<space>', '<space>', '<space>', '<space>', 'lcm', '.', 'multiple', '=', 'lcm', '(', 'a', '*', 'b', ')', '<new_line>', '<space>', '<space>', '<space>', '<space>', 'if', '(', 'a', '>', 'b', ')', ':', '<new_line>', '<space>', '<space>', '<space>', '<space>', '<space>', '<space>', '<space>', '<space>', '<space>', '<space>', 'greater', '<space>', '=', '<space>', 'x', '<new_line>', '<space>', '<space>', '<space>', '<space>', 'else', ':', '<new_line>', '<space>', '<space>', '<space>', '<space>', '<space>', '<space>', '<space>', '<space>', 'greater', '<space>', '=', '<space>', 'y', '<new_line>', '<space>', '<space>', '<space>', '<space>', 'while', '(', 'true', ')', ':', '<new_line>', '<space>', '<space>', '<space>', '<space>', '<space>', '<space>', '<space>', '<space>', 'if', '(', '(', '(', 'greater', '<space>', '%', '<space>', 'x', '<space>', '=', '=', '<space>', '0', ')', '<space>', 'and', '<space>', '(', 'greater', '<space>', '%', '<space>', 'y', '<space>', '=', '=', '<space>', '0', ')', ')', ')', ':', '<new_line>', '<space>', '<space>', '<space>', '<space>', '<space>', '<space>', '<space>', '<space>', '<space>', '<space>', '<space>', '<space>', 'lcm', '.', 'append', '(', 'greater', ')', '<new_line>', '<space>', '<space>', '<space>', '<space>', '<space>', '<space>', '<space>', '<space>', '<space>', '<space>', '<space>', '<space>', '<space>', '<space>', '<space>', 'break', '<new_line>', '<space>', '<space>', '<space>', '<space>', '<space>', '<space>', '<space>', '<space>', 'if', '(', 'greater', '<space>', '+', '<space>', '1', ')', ':', '<new_line>', '<space>', '<space>', '<space>', '<space>', '<space>', '<space>', '<space>', '<space>', '<space>', '<space>', '<space>', '<space>', '<space>', '<space>', '<space>', 'lcm', '(', 'greater', '<space>', '+', '=', '<space>', '1', ')', '<new_line>', '<space>', '<space>', '<space>', '<space>', '<space>', '<space>', 'return', '<space>', 'lcm', '<new_line>', '<eos>']


```
Example-11
```
problem statement:
#   write a python program to check is all are digit
truth:
print('0xa'.isdigit())

prediction:
['print', '(', "'", '0xa', "'", '.', 'isdigit', '(', ')', ')', '<new_line>', '<eos>']

```
Example-12
```
problem statement:
# write a generator function in python to generate infinite square of numbers using yield
truth:
def nextsquare(): 
    i = 1;  
    # an infinite loop to generate squares  
    while true: 
        yield i*i                 
        i += 1

prediction:
['def', '<space>', 'nextsquare', '(', ')', ':', '<space>', '<new_line>', '<space>', '<space>', '<space>', '<space>', 'i', '<space>', '=', '<space>', '1;', '<space>', '<space>', '<new_line>', '<space>', '<space>', '<space>', '<space>', '#', '<space>', 'an', '<space>', 'infinite', '<space>', 'loop', '<space>', 'to', '<space>', 'generate', '<space>', 'squares', '<space>', '<space>', '<new_line>', '<space>', '<space>', '<space>', '<space>', 'while', '<space>', 'true', ':', '<space>', '<space>', '<space>', '<space>', '<space>', 'yield', '<space>', 'i', '*', 'i', '<space>', '<space>', '<space>', '<space>', '<space>', '<space>', '<space>', '<space>', '<space>', '<space>', '<space>', '<space>', '<space>', '<space>', '<space>', '<space>', '<space>', '<space>', '<space>', '<space>', '<space>', '<space>', '<space>', '<space>', '<space>', '<space>', '<space>', 'i', '<space>', '+', '=', '<space>', '1', '<new_line>', '<eos>']

```
Example-13
```
problem statement:
# write a python function to find the area of a circle , whose radius is given
truth:
def findarea(r): 
    pi = 3.142
    return pi * (r*r)

prediction:
['def', '<space>', 'findarea', '(', 'r', ')', ':', '<space>', '<new_line>', '<space>', '<space>', '<space>', '<space>', 'pi', '<space>', '=', '<space>', '3', '.', '142', '<new_line>', '<space>', '<space>', '<space>', '<space>', 'return', '<space>', 'pi', '<space>', '*', '<space>', '(', 'r', '*', 'r', ')', '<new_line>', '<eos>']

```
Example-14
```
problem statement:
# write a function that acts like a relu function for a 1d array
truth:
def relu_list(input_list:list)->list:
    return [(lambda x: x if x >= 0 else 0)(x) for x in input_list]

prediction:
['def', '<space>', 'relu_list', '(', 'input_list', ':', 'list', ')', '-', '>', 'list', ':', '<new_line>', '<space>', '<space>', '<space>', '<space>', 'return', '<space>', '[', '(', 'lambda', '<space>', 'x', ':', '<space>', 'x', '<space>', 'if', '<space>', 'x', '<space>', '>', '=', '<space>', '0', '<space>', 'else', '<space>', '0', ')', '(', 'x', ')', '<space>', 'for', '<space>', 'x', '<space>', 'in', '<space>', 'input_list', ']', '<new_line>', '<eos>']

```
Example-15
```
problem statement:
# write a python program to check tuple are immutable
truth:
a=(1,2,3)
try:
    a = a+1
except exception as e:
    print(e)

prediction:
['tup', '<space>', '=', '<space>', '(', '1', ',', ')', '<new_line>', 'tup', '[', '0', ']', '<space>', '=', '<space>', 'tup', '[', '0', ']', '<new_line>', 'for', '<space>', 'tup', '<space>', 'in', '<space>', 'tup', ':', '<new_line>', '<space>', '<space>', '<space>', '<space>', '<space>', 'tup', '[', '0', ']', '<space>', '+', '=', '<space>', 'tup', '[', '1', ']', '<new_line>', '<space>', '<space>', '<space>', '<space>', '<space>', '<space>', '<space>', '<space>', 'tup', '[', '0', ']', '<space>', '+', '=', '<space>', '1', '<new_line>', 'print', '(', 'tup', ')', '<new_line>', '<eos>']


```
Example-16
```
problem statement:
# write a function to calculate the moment of inertia of a ring of mass m and radius r
truth:
def cal_mi_ring(mass:float,radius:float)->float:
    return mass*(radius**2)

prediction:
['def', '<space>', 'cal_mi_ring', '(', 'mass', ':', 'float', ',', 'radius', ':', 'float', ')', '-', '>', 'float', ':', '<new_line>', '<space>', '<space>', '<space>', '<space>', 'return', '<space>', 'mass', '*', '(', 'radius', '*', '*', '2', ')', '<new_line>', '<eos>']

```
Example-17
```
problem statement:
# write python program that would merge two dictionaries by adding the second one into the first
truth:
a = {'a': 1, 'b': 3}
b = {'c': 1, 'd': 3}
a.update(b)

prediction:
['a', '<space>', '=', '<space>', '{', "'", 'a', "'", ':', '1', ',', '<space>', "'", 'b', "'", ':', '2', ',', '<space>', "'", 'c', "'", ':', '3', '}', '<new_line>', 'b', '<space>', '=', '<space>', '{', "'", 'v', "'", ':', '<space>', '3', ',', '<space>', '3', '}', '<new_line>', 'c', '.', 'update', '(', 'b', ')', '<new_line>', 'print', '(', 'f', "'", 'dictionary', ':', '{', 'a', '}', "'", ')', '<new_line>', '<eos>']
```
Example-18
```

problem statement:
#   write a python program to check is all are num / int
truth:
print('ab,12'.isalnum())

prediction:
['print', '(', "'", 'ab', ',', '12', "'", '.', 'isalnum', '(', ')', ')', '<new_line>', '<eos>']

```
Example-19
```
problem statement:
# write a python program to print equal lenght of string
truth:
print('ab'.zfill(5))

prediction:
['print', '(', "'", 'ab', ',', '12', "'", '.', 'isalnum', '(', ')', ')', '<new_line>', '<eos>']


```
Example-20
```
problem statement:
# write a function to merge two lists element wise
truth:
def merge_lists(l1:list, l2:list):
    return list(zip(l1,l2))

prediction:
['def', '<space>', 'merge_lists', '(', 'l1', ':', 'list', ',', '<space>', 'l2', ':', 'list', ')', ':', '<new_line>', '<space>', '<space>', '<space>', '<space>', 'return', '<space>', 'list', '(', 'zip', '(', 'l1', ',', 'l2', ')', ')', '<new_line>', '<eos>']

```
Example-21
```
problem statement:
# write a python program to search a key in the text file
truth:
fname = 'sample.txt'
l='keyword' # enter letter to be searched
k = 0
 
with open(fname, 'r') as f:
    for line in f:
        words = line.split()
        for i in words:
            if(i==l):
                k=k+1
print('occurrences of the letter:',k)

prediction:
['import', '<space>', 'os', '<new_line>', 'import', '<space>', 'nltk', '<new_line>', 'import', '<space>', 'string', '<new_line>', 'import', '<space>', 'string', '<new_line>', 'from', '<space>', 'open', '(', 'filename', ',', '<space>', "'", 'r', "'", ')', '<space>', 'as', '<space>', 'f', ':', '<new_line>', '<space>', '<space>', '<space>', '<space>', 'for', '<space>', 'line', '<space>', 'in', '<space>', 'f', ':', '<new_line>', '<space>', '<space>', '<space>', '<space>', '<space>', '<space>', '<space>', '<space>', '<space>', '<space>', '<space>', '<space>', '#', '<space>', 'line', '<space>', 'the', '<space>', 'string', '<space>', 'is', '<space>', 'the', '<space>', 'letter', '<space>', 'in', '<space>', 'string', '<space>', 'the', '<space>', 'string', '<space>', 'is', '<space>', 'i', '<space>', 'good', '<space>', ':', '<new_line>', '<space>', '<space>', '<space>', '<space>', '<space>', '<space>', '<space>', '<space>', '<space>', '<space>', '<space>', '<space>', 'if', '(', 'letter', '.', 'isupper', '(', ')', ')', ':', '<new_line>', '<space>', '<space>', '<space>', '<space>', '<space>', '<space>', '<space>', '<space>', '<space>', '<space>', '<space>', '<space>', '<space>', '<space>', '<space>', '<space>', '<space>', '<space>', '<space>', '<space>', '#', '<space>', 'new', '<space>', 'string', '<space>', '+', '=', '<space>', '1', '<new_line>', '<space>', '<space>', '<space>', '<space>', '<space>', '<space>', '<space>', '<space>', '<space>', '<space>', '<space>', '<space>', '<space>', '<space>', 'if', '(', 'letter', '.', 'upper', '(', ')', ')', ':', '<new_line>', '<space>', '<space>', '<space>', '<space>', '<space>', '<space>', '<space>', '<space>', '<space>', '<space>', '<space>', '<space>', '<space>', '<space>', '<space>', '<space>', '<space>', '<space>', '<space>', '<space>', '<space>', '<space>', '<space>', '<space>', 'break', '<new_line>', '<space>', '<space>', '<space>', '<space>', '<space>', '<space>', '<space>', '<space>', '<space>', '<space>', '<space>', '<space>', 'else', ':', '<new_line>', '<space>', '<space>', '<space>', '<space>', '<space>', '<space>', '<space>', '<space>', '<space>', '<space>', '<space>', '<space>', '<space>', '<space>', '<space>', '<space>', '<space>', '<space>', '<space>', 'res', '<space>', '=', '<space>', 'true', '<new_line>', '<space>', '<space>', 'return', '<space>', 'false', '<new_line>', '<eos>']

```
Example-22
```
problem statement:
# write a function to return the volume of a sphere
truth:
def cal_sphere_volume(radius:float)->float:
    pi=3.14
    return (4/3)*pi*(radius**3)

prediction:
['def', '<space>', 'cal_sphere_volume', '(', 'radius', ':', 'float', ')', '-', '>', 'float', ':', '<new_line>', '<space>', '<space>', '<space>', '<space>', 'pi', '=', '3', '.', '14', '<new_line>', '<space>', '<space>', '<space>', '<space>', 'return', '<space>', '(', '4', '/', '3', ')', '*', 'pi', '*', '(', 'radius', '*', '*', '3', ')', '<new_line>', '<eos>']

```
Example-23
```
problem statement:
# python program to implement gnome sort
truth:
def gnomesort(arr, n):
    index = 0
    while index < n:
        if index == 0:
            index = index + 1
        if arr[index] >= arr[index - 1]:
            index = index + 1
        else:
            arr[index], arr[index - 1] = arr[index - 1], arr[index]
            index = index - 1
    return arr
arr = [34, 2, 10, -9]
n = len(arr)
arr = gnomesort(arr, n)
print('sorted seqquence after applying gnome sort :')
for i in arr:
    print(i)

prediction:
['def', '<space>', 'gnomesort', '(', 'arr', ',', '<space>', 'n', ')', ':', '<new_line>', '<space>', '<space>', '<space>', '<space>', 'index', '<space>', '=', '<space>', '0', '<new_line>', '<space>', '<space>', '<space>', '<space>', 'while', '<space>', 'index', '<space>', '<', '<space>', 'n', ':', '<new_line>', '<space>', '<space>', '<space>', '<space>', '<space>', '<space>', '<space>', '<space>', 'if', '<space>', 'index', '<space>', '=', '=', '<space>', '0', ':', '<new_line>', '<space>', '<space>', '<space>', '<space>', '<space>', '<space>', '<space>', '<space>', '<space>', '<space>', '<space>', '<space>', 'index', '<space>', '=', '<space>', 'index', '<space>', '+', '<space>', '1', '<new_line>', '<space>', '<space>', '<space>', '<space>', '<space>', '<space>', '<space>', '<space>', 'if', '<space>', 'arr', '[', 'index', ']', '<space>', '>', '=', '<space>', 'arr', '[', 'index', '<space>', '-', '<space>', '1', ']', ':', '<new_line>', '<space>', '<space>', '<space>', '<space>', '<space>', '<space>', '<space>', '<space>', '<space>', '<space>', '<space>', '<space>', 'index', '<space>', '=', '<space>', 'index', '<space>', '+', '<space>', '1', '<new_line>', '<space>', '<space>', '<space>', '<space>', '<space>', '<space>', '<space>', '<space>', 'else', ':', '<new_line>', '<space>', '<space>', '<space>', '<space>', '<space>', '<space>', '<space>', '<space>', '<space>', '<space>', '<space>', '<space>', 'arr', '[', 'index', ']', ',', '<space>', 'arr', '[', 'index', '<space>', '-', '<space>', '1', ']', '<space>', '=', '<space>', 'arr', '[', 'index', '<space>', '-', '<space>', '1', ']', '<new_line>', '<space>', '<space>', '<space>', '<space>', '<space>', '<space>', '<space>', '<space>', '<space>', '<space>', '<space>', '<space>', '<space>', '<space>', '<space>', '<space>', '<space>', '<space>', 'index', '<space>', '=', '<space>', 'index', '<space>', '-', '<space>', '1', '<new_line>', '<space>', '<space>', '<space>', '<space>', 'return', '<space>', 'arr', '<new_line>', 'arr', '<space>', '=', '<space>', '[', '34', ',', '<space>', '2', ',', '<space>', '10', ',', '<space>', '-', '9', ']', '<new_line>', 'n', '<space>', '=', '<space>', 'len', '(', 'arr', ')', '<new_line>', 'arr', '<space>', '=', '<space>', 'gnomesort', '(', 'arr', ',', '<space>', 'n', ')', '<new_line>', 'print', '(', "'", 'sorted', '<space>', 'seqquence', '<space>', 'after', '<space>', 'applying', '<space>', 'gnome', '<space>', 'sort', '<space>', ':', "'", ')', '<new_line>', 'for', '<space>', 'i', '<space>', 'in', '<space>', 'arr', ':', '<new_line>', '<space>', '<space>', '<space>', '<space>', 'print', '(', 'i', ')', '<new_line>', '<eos>']


```
Example-24
```
problem statement:
# write a function to subtracts two lists element wise
truth:
def sub_listelements(l1:list, l2:list):
    return [i-j for i, j in zip(l1,l2)]

prediction:
['def', '<space>', 'sub_listelements', '(', 'l1', ':', 'list', ',', '<space>', 'l2', ':', 'list', ')', ':', '<new_line>', '<space>', '<space>', '<space>', '<space>', 'return', '<space>', '[', 'i', '-', 'j', '<space>', 'for', '<space>', 'i', ',', '<space>', 'j', '<space>', 'in', '<space>', 'zip', '(', 'l1', ',', 'l2', ')', ']', '<new_line>', '<eos>']


```
Example-25
```
problem statement:
# write a program to print the multiplication table of a given number
truth:
num = 9
for i in range(1, 11):
   print(f'{num} x {i} = {num*i}')

prediction:
['num', '<space>', '=', '<space>', 'int', '(', 'input', '(', "'", 'please', '<space>', 'enter', '<space>', 'a', '<space>', 'number', '<space>', "'", ')', ')', '<new_line>', 'for', '<space>', 'a', '<space>', 'in', '<space>', 'range', '(', '1', ',', '11', ')', ':', '<new_line>', '<space>', '<space>', '<space>', '<space>', 'print', '(', 'num', ',', "'", 'x', "'", ',', 'i', ',', "'", '=', "'", ',', 'num', '*', 'i', ')', '<new_line>', '<space>', '<space>', '<space>', '<new_line>', '<eos>']


```

Resources:

[Transformer](https://rahulkarnam777.medium.com/transformers-rising-from-the-ashes-of-rnn-lstm-35619d1a1bf6)
[RNN](https://rahulkarnam777.medium.com/visualize-nlp-the-intuition-behind-rnn-lstm-grn-and-attention-mechanism-998022068080)
[English to german Translation](https://rahulkarnam777.medium.com/english-to-german-translation-using-convolution-sequence-to-sequence-architecture-18865bb4becc)




