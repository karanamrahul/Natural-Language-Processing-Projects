# END-Course-NLP

## This repo is used for submission of END- NLP course assignments.

Week-3 Assignment

1. Write a function using only list filter lambda that can tell whether a number is a Fibonacci number or not. You can use a pre-calculated list/dict to store fab numbers till 10000 PTS:100

2. Using list comprehension (and zip/lambda/etc if required) write five different expressions that: PTS:100

        a.add 2 iterables a and b such that a is even and b is odd
        
        b.strips every vowel from a string provided (tsai>>t s)
  
        c.acts like a ReLU function for a 1D array
  
        d.acts like a sigmoid function for a 1D array
  
        e.takes a small character string and shifts all characters by 5 (handle boundary conditions) tsai>>yxfn

3. A list comprehension expression that takes a ~200 word paragraph (write your own paragraph to check), and checks whether it has any of the swear words mentioned in https://github.com/RobertJGabriel/Google-profanity-words/blob/master/list.txt PTS:200

4. Using reduce functions: PTS:100
   
    a.add only even numbers in a list
    
    b.find the biggest character in a string (printable ascii characters)
    
    c.adds every 3rd number in a list

5. Using randint, random.choice and list comprehensions, write an expression that generates 15 random KADDAADDDD number plates, where KA is fixed, D stands for a digit, and A     
   stands for Capital alphabets. 10<<DD<<99 & 1000<<DDDD<<9999 PTS:100

5.1 Write the above again from scratch where KA can be changed to DL, and 1000/9999 ranges can be provided.  PTS:100



Week-8 Assignment

## Task-1[Solution](https://github.com/karanamrahul/END-Course-NLP/blob/main/Week-8%20Sequence_to_Sequence_Model_English-German.ipynb)

To create a seq2seq model which can translate text from english to german using encoder and decoder architecture with three layers of LSTM.

Reference for the seq2seq paper :[seq2seq](https://arxiv.org/pdf/1409.3215.pdf)

## Task-2

To create 100 python programs which will be used as a dataset in the upcoming course.

write 100 such examples. These are the requirements:
if you want the code to print something, then mention "print" in the text


Example file - sample.py
#### write a python program to add two numbers 
num1 = 1.5
num2 = 6.3
sum = num1 + num2
print(f'Sum: {sum}')


#### write a python function to add two user provided numbers and return the sum
def add_two_numbers(num1, num2):
    sum = num1 + num2
    return sum


#### write a program to find and print the largest among three numbers

num1 = 10
num2 = 12
num3 = 14
if (num1 >= num2) and (num1 >= num3):
   largest = num1
elif (num2 >= num1) and (num2 >= num3):
   largest = num2
else:
   largest = num3
print(f'largest:{largest}')


read the sample.py clearly to see the examples and learn from it. If you want the code to write a program, you are NOT asking to write a function. If you want a function, please mention it. 

run the code in the terminal/notebook and make sure it runs!

you cannot write more than 5 simple functions like add 2/3/4/5 numbers or divide, etc. Try and think of something tough, for example, add a list or tuple together. 
stick to python only (no 3rd party library like numpy, etc)

here are some examples:

1.provide the length of list, dict, tuple, etc
2.write a function to sort a list
3.write a function to test the time it takes to run a function
4.write a program to remove stop words from a sentence provided,
etc
