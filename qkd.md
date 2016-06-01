
# Mock Quantum Key Distribution System
Below is a demonstration of Quantum Key Distripution(QKD) based on the BB84 Protocol.

## Random Bit Generator



```python
import random
count = 0
alice_bits = []

while count < 1000000:
    bit = random.randint(0, 1)
    alice_bits.append(bit)
    count += 1
#print ("Alice's bits:",alice_bits)       
```

## Polarization Generators


```python
# Rectilinear Polerization Generator
def RectGen ( bit ):
    if bit == 1:
        qubit = u'\u2195'
        
    elif bit == 0:
        qubit = u'\u2194'
    return qubit

#Diagnal Polorization Generator
def DiagGen ( bit ):
    if bit == 1:
        qubit = u'\u2196'
    
    elif bit == 0:
        qubit = u'\u2197'   
    return qubit
```

### Random Polarization Generator
This function randomly selects ane uses either a rectilinear or diagonal polarization generator as each bit is fed into it. It then outputs the result and the generator used to calulate it. 


```python
def RandGen(bit):
    n = random.randint(0, 1)
    
    if n == 0:
        result = RectGen(bit)
        gen = "R"
        
    elif n == 1:
        result = DiagGen(bit)
        gen = "D"
    return result, gen
```

### Alice's results


```python
alice_qubits = []
alice_polChoice = []
for bit in alice_bits:
    Generate = RandGen(bit)
    alice_polChoice.append(Generate[1])
    alice_qubits.append(Generate[0])

#print ("Alice's Qubits:",alice_qubits)
#print ("Alice's Polarity Choices:",alice_polChoice)     
```

## Polarization Dectors


```python
#Rectilinear Polarization Dector
def RectDect (qubit ):
    if qubit == "↕":
        bit = 1
    
    elif qubit == "↔":
        bit = 0
    
    else:
        bit = random.randint(0, 1)    
    return bit

#Diagnal Polarization Dector
def DiagDect (qubit ):
    if qubit == "↖":
        bit = 1
    
    elif qubit == "↗":
        bit = 0
    
    else:
        bit = random.randint(0, 1)  
    return bit        
```


## Random Polarity Selector
As Bob receives qubits from Alice, he will randomly select a polarity dectector to filter each qubit through. He will record the resulting bit and also annotate which dector he used to filter each qubit.   



```python
def RandPolSel (qubit):
    process = random.randint(1, 2)
    if process == 1:
        polfilter = "R"
        polgen = RectDect(qubit)

    elif process == 2:
        polfilter = "D"
        polgen = DiagDect(qubit)
    return polfilter, polgen
```


```python

```

## If Eve is in the Middle


```python
eve_bits = []
eve_polChoice = []

#Capture the Qubits From Alice
for qubit in alice_qubits:
    result = RandPolSel(qubit)
    eve_bits.append(result[1])
    eve_polChoice.append(result[0])

#Send Qubits to Bob by pretending to be Alice
alice_qubits = []
alice_polChoice = []
for bit in eve_bits:
    Generate = RandGen(bit)
    alice_polChoice.append(Generate[1])
    alice_qubits.append(Generate[0])

#print ("Alice's Qubits:",alice_qubits)
#print ("Alice's Polarity Choices:",alice_polChoice)         
```

### Bob's Results


```python
bob_bits = []
bob_polChoice = []

for qubit in alice_qubits:
    result = RandPolSel(qubit)
    bob_bits.append(result[1])
    bob_polChoice.append(result[0])
```


```python
#print (bob_bits)
```


```python
#print (bob_polChoice)
```

## Compare Alice's and Bob's Polarity Choices
Over a clasical channel, Bob will send Alice a list of his polarity choices for the qubits he received. Alice reviews his choices ant tells him which ones are correct. The ones that do not match are thrown out. the remaing bits are now the raw key.  The next step is now to determing if the raw key is private.


```python
maxcount = len(alice_polChoice)
count = 0
keepindx = []
while count < maxcount:
    alice_base = alice_polChoice[count]
    bob_base = bob_polChoice[count]
    
    if alice_base == bob_base:
        keepindx.append(count)
        
    count += 1
```


```python

matchcount = len(keepindx)
print ("Matching polarity:", len(keepindx))

bob_rawkey = []
for id in keepindx:
    bob_rawkey.append(bob_bits[id])
print ("Bob's raw key length:",len(bob_rawkey))

alice_rawkey = []
for id in keepindx:
    alice_rawkey.append(alice_bits[id])
print ("Alice's raw key length:", len(alice_rawkey))    
```

    Matching polarity: 500101
    Bob's raw key length: 500101
    Alice's raw key length: 500101
    

## Determine if key is secure
To determine if the key is truly private and has not been tampered with in transit Alice and bob will reveal a few random bits of their raw keys to see if they match. (The random number must be non-repeating and once used, must be removed from the rawkey data sets of both Alice and Bob.) As you will see below their are 0 errors, In this case we can be assuered that the key has not been compromised. We can then use this key to encrypt our traffic over any classical channel and be assured a high level of provacy. 


```python
#pick 1/3 of the random bits and compare 

mpoints = round((matchcount *.33),0)
keyschecked = mpoints
goodkeys = 0
badkeys = 0

while mpoints > 0:
    
    Sample = random.sample(range(0,(matchcount - 1)), 1)


    for testkey in Sample:
        if bob_rawkey[testkey] == alice_rawkey[testkey]:
            goodkeys += 1
            del bob_rawkey[testkey] #removing keys after being used
            del alice_rawkey[testkey] #removing keys after being used
        else:
            badkeys += 1
            del bob_rawkey[testkey]
            del alice_rawkey[testkey]
        mpoints -= 1
        matchcount -= 1
print ("ErrorRate:", round((badkeys/keyschecked),2))
print ("Accuracy:", round((goodkeys/keyschecked),2))
print ("GoodKeys:", goodkeys)
print ("BadKeys:", badkeys)
print ("Alice's New raw key length:", len(alice_rawkey))
print ("Bob's raw key length:",len(bob_rawkey))
```

    ErrorRate: 0.25
    Accuracy: 0.75
    GoodKeys: 123503
    BadKeys: 41530
    Alice's New raw key length: 335068
    Bob's raw key length: 335068
    


```python
mpoints = round((matchcount *.33),0)
print (mpoints)
print (matchcount)
```

    110572.0
    335068
    


```python

```
