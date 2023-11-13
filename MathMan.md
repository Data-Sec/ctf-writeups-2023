# Math Man
## Info
Math guy is challenging you. Find the flag and beat him!

## Task
Find the Flag `DS-CTF{<to_find>}`

## Solution
How to Solve each Challenge
### Prerequisits
#### Following Libraries used
```Python
from operator import mul        # Simpler Multiplications
from functools import reduce    # Simpler Multiplications
from re import compile,findall  # RegEx in Python
from pwn import remote          # PwnTools remote for the connection
```

#### Setup connection
```Python
conn = remote('<ip>', <port>) # Setup the connection
tmp = b''                     # Setting up variable which will be used for the Input
```
##### General Idea
The only issue is finding out how many lines will be send before the Server waits for a response, this can be simply done by trial and error
2. Use `conn.recvlines(<amount>)` until you find the right amount of lines
2. Print Server Response
3. Check how many responses where given before Server stopped
4. Update the `amount` to the correct amount and check the last line for the infos

Code Example:
```Python
runs = 20                     # This will be updated once you have the correct amount
tmp = conn.recvlines(runs)
print(tmp)
```

### First Challenge
1. Get the Server Context for the Challenge
2. decode the `binary string` to `string` using `utf8`
3. split the string at the `+` sign
4. build the `sum` of the list
5. turn the `integer` back into `string` add a `\n` and send it back

Code Example:
```Python
tmp = conn.recvlines(4)[-1]
print(tmp)

FIRST = str(sum([int(number) for number in tmp.decode('utf8').split('+')])) + '\n'
print(FIRST)
conn.send(FIRST.encode())
```

### Second Challenge
1. Get the Server Context for the Challenge
2. decode the `binary string` to `string` using `utf8`
3. split the string after every Space (`' '`)
4. use the sixth word in the list and remove the last character since it isn't part of the number (`list[5][:-1]`)
5. build the Square by using `<number>**2`
6. turn the `integer` back into `string` add a `\n` and send it back

Code Example
```Python
tmp = conn.recvlines(3)[-1]
print(tmp)

SECOND = str(int(tmp.decode('utf8').split(' ')[5][:-1])**2) + '\n'
print(SECOND)
conn.send(SECOND.encode())
```

### Third Challenge
Two Solutions for this one (for the people that don't like RegEx)
#### Without RegEx
1. Get the Server Context for the Challenge
2. decode the `binary string` to `string` using `utf8`
3. split the string after every space (`' '`)
4. go through every word in the Text and try to turn it into `integer`
  1. if it can be turned into an `integer` add it to the list of numbers
5. multiply all the numbers in the list of numbers
6. turn the `integer` back into `string` add a `\n` and send it back

Code Example:
```Python
tmp = conn.recvlines(4)[-1]
print(tmp)

THIRD_1 = tmp.decode('utf8').split(' ')

## Solution without regex
all_numbers = []
for word in THIRD_1:
    try:
        all_numbers.append(int(word))
    except:
        pass
THIRD_1 = str(reduce(operator.mul, all_numbers)) + '\n'
print(THIRD_1)
conn.send(THIRD_1.encode())
```

#### With RegEx
1. Get the Server Context for the Challenge
2. decode the `binary string` to `string` using `utf8`
3. split the string after every space (`' '`)
4. Write the `regex` to filter out all Numbers (`0-9`) check the Input List and find all `matches`
5. multiply all the numbers that where found by the regex
6. turn the `integer` back into `string` add a `\n` and send it back

Code Example:
```Python
tmp = conn.recvlines(4)[-1]
print(tmp)

THIRD_2 = tmp.decode('utf8').split(' ')

## Solution with regex
THIRD_2 = str(reduce(operator.mul, [int(number) for number in list(filter(re.compile('[0-9]').match, THIRD_2))])) + '\n'
print(THIRD_2)
conn.send(THIRD_2.encode())
```
### Forth Challenge
1. Get the Server Context for the Challenge
2. decode the `binary string` to `string` using `utf8`
3. split the string after every space (`' '`)
4. Make a helper directory to translate the written number to the actual number (`{"One": "1", ..., "Zero": "0"}`)
5. Turn each written number into its own element, this can be done via RegEx
6. Turn each element of the list into a number and `join` them into a single string that can be turned into an `integer` (Do this with both numbers)
7. Do the calculation of the 2 Numbers with the right calculation sign ( `+, -, /, *` )
8. turn the `integer` back into `string` add a `\n` and send it back

Code Example:
```Python
conn.recvlines(4)[-1]
print(tmp)

FORTH = tmp.decode('utf8').split(' ')

## Dict to translate
numbers = {
        "One": "1",
        "Two": "2",
        "Three": "3",
        "Four": "4",
        "Five": "5",
        "Six": "6",
        "Seven": "7",
        "Eight": "8",
        "Nine": "9",
        "Zero": "0"
        }
## Filter out the Words
first_num = re.findall('[A-Z][^A-Z]*', FORTH[0])
second_num = re.findall('[A-Z][^A-Z]*', FORTH[2])

## Turn the written ones into integer Numbers
first_num = int(''.join([numbers[char] for char in first_num]))
second_num = int(''.join([numbers[char] for char in second_num]))
print('1. Number: ', first_num)
print('2. Number: ', second_num)

## Calculate depending on the calculation sign
if FORTH[1] == '+':
    FORTH = first_num + second_num
elif FORTH[1] == '-':
    FORTH = first_num - second_num
elif FORTH[1] == '*':
    FORTH = first_num * second_num
elif FORTH[1] == '/':
    FORTH = first_num / second_num

FORTH = str(FORTH) + '\n'
print(FORTH)
conn.send(FORTH.encode())
```
