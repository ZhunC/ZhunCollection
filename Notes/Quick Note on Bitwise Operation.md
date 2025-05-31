---
profileName: Zhun's collected work
postId: "66"
postType: post
categories:
  - 1
---

# C Bitwise Operations Reference

2025-05-20  Tags: #c #programming #embedded #bitwise 

## Overview

When communicating with Mechatronic components like actuators, sensors, or ICs, communication must follow certain protocols. That is, data are transmitted in sequential binary signals, and they are packaged with certain formats. I will write more on protocols, but this note records some common ways to manipulate data, often decimal in their original settings, as binary values. These manipulations are often essential to properly packaging the data values. 

> [!info] Bitwise operations are essential when working with hardware registers, flags, and communication protocols like I2C, SPI, and UART.

## Basic Bitwise Operators

| Operator | Name        | value  | Description                                             |
| -------- | ----------- | -------- | ------------------------------------------------------- |
| `&`      | AND         | `a & b`  | Sets bits to 1 only if both corresponding bits are 1    |
| `\|`     | OR          | `a \| b` | Sets bits to 1 if at least one corresponding bit is 1   |
| `^`      | XOR         | `a ^ b`  | Sets bits to 1 only if corresponding bits are different |
| `~`      | NOT         | `~a`     | Inverts all bits                                        |
| `<<`     | Left shift  | `a << n` | Shifts bits left by n positions                         |
| `>>`     | Right shift | `a >> n` | Shifts bits right by n positions                        |

## Common Bit Manipulation Techniques

### Bit Shifting

This is especially useful in serial communication. Communication protocols often require certain bits as a single message package, and padding 0s are often needed in this setting. It also comes up when we want to merge packages (like 2 8 bit packages) into its original value as a 16 bit variable. 


```c
uint8_t value = 1 << 3; // This line initializes a binary value 1 and shifts it to the left by 3 bits, effectively multiplying by 8;
// so equivalently you can do:
value = 8; 
// or:
value = 0x08; 
```


### Initialization:

```c

uint8_t value = 0b01010101; // This is a binary value. '0b' is the identifier
// equivalently in hexadecimal:
value = 0x55; // '0x' now signifies it is a hexadecimal value. 
// or, in decimal:
value = 85; // No prefix means it's decimal

```

### Setting Bits

We can use 'or' operation to set specific bits from 0 to 1 without affecting others. Conversely, we can use 'and' operation to set specific bits from 1 to 0 in the same fashion.  

```c
// Set bits 0 and 3 in variable 'value' to 1
uint8_t value = 0b00000000;
value |= (0b00000001| 0b00001000);  // value is now 0b00000101
// or to use the decimal to save space:
value |= (1 | 8);
// or to use bit shift to clarify which bit is getting set:
value |= (1) | (1 << 3);
```
### Toggling Bits

Sometimes we want to flip certain bits in a number. This can be done with an 'XOR' operation and a masking variable. The masking variable has 1 at where we want flipping to happen, and 0 at where we don't. 

A quick sanity check on why XOR works in this setting is as below. The masked bits are flipped, and the unmasked remains. 

| Original bit | Mask value | Output value |
| ------------ | ---------- | ------------ |
| 0            | 0          | 0            |
| 0            | 1          | 1            |
| 1            | 0          | 1            |
| 1            | 1          | 0            |
Now in c:
```c
// Flipping bits 0 and 3 in variable 'value'
uint8_t value = 0b11111011;  
uint8_t mask = (1) | (1 << 3); 
value ^= mask; // value becomes 0b11110010
```

### Checking If Bits Are On

We can check if certain bits are on or off by using 'and' operation.

```c
uint8_t value = 0b00110001;  

// Check if bit 3 is 1
if (value & (1 << 3)) {
    // Do stuff here, and for our 'value', we won't enter here.
}

// Check if bit 0 is 1
if (value & (1)) {
    // Do stuff here, and for our 'value', we do enter here.
}
```

### Even / Odd Parity Check

In some protocols, or for personal use, sometimes we want to perform an even or odd parity check to a binary values. We want to count if the number of occurrence of 1s in this binary is even or odd. Check the following table for a clearer illustration of what this value should be.

| Are there even or odd occurrences of 1s in the binary? | Even or Odd Parity Check? | Parity Check Bit Value |
| ------------------------------------------------------ | ------------------------- | ---------------------- |
| Even                                                   | Even                      | 1                      |
| Odd                                                    | Even                      | 0                      |
| Even                                                   | Odd                       | 0                      |
| Odd                                                    | Odd                       | 1                      |

```c
uint8_t value = 0b10110010; // This is the value we're testing

uint8_t counter = 0; // This stores the number of 1s

while (value) // Loop executes until value becomes 0b00000000
{ 
counter += value & 1; // Check if the last bit is 1
value >>= 1; // After checking, pop it off by right shifting 1 bit
} 

bool EvenParityBit = (count % 2) == 0;
bool OddParityBit = (count % 2) == 1;
```
### Putting Everything Together: Sending a Frame of Data

Now let's try putting together a read-and-write exercise process. In this process, we will receive a frame of data, modify it, and send it out, according to the following protocol I made up. 

1. The first two bits are headers that designate the purpose/type of this communication. In this exercise we're first reading values from a register address first and then write into the same register address. 'Response' requires the header 01 in binary, and  'write' operation requires the header 11 in binary.
2. The Next four bits are values of the register address we're writing into. Suppose we are writing into the address 0x0C, equivalently 0b1100. 
3. The next one bit is an even parity check to the 8-bit data. 
4. The next one bit is an odd parity check to the 8-bit data.
5. The next eight bits are the data we're receiving/writing into. In reality, sometimes each bit carries meanings, sometimes the entire 8-bit binary carries meanings, sometimes the grouping of certain bits carries meanings. For this example, we're reading this particular value: 0b11010001. We want to flip its bit 0, 2, 4, 6, and write value 1 to its bit 1 and 3.
6. Data transmission is in large endian format, meaning bit 15-8 comes in as one 8-bit package, and bit 7-0 comes in as the second. 

```c
// Define header and address:
static uint8_t s_nWriteHeader = 0b11;
static uin8_t s_nAddress = 0x0C;

// Define flipping mask:
static uint8_t FlippingMask = (1) | (1<<2) | (1<<4) | (1<<6);

// Define write mask:
static uint8_t WriteMask = (1<<1) | (1<<3);

// Neglect serial communication process, we're focusing on bit manipulation now.
// Suppose a read request has been sent and the following response is received: 
uint8_t nFirstData = 0b01110010; // Remember this is header+address+2 parity checks
uint8_t nSecondData = 0b11010001; // Remember this is the data. 

// Define header and address:
static uint8_t s_nWriteHeader = 0b11;
static uint8_t s_nAddress = 0x0C;

// Define flipping mask:
static uint8_t FlippingMask = (1) | (1<<2) | (1<<4) | (1<<6);

// Define write mask:
static uint8_t WriteMask = (1<<1) | (1<<3);

// Neglect serial communication process, we're focusing on bit manipulation now.
// Suppose a read request has been sent and the following response is received: 
uint8_t nFirstData = 0b01110010; // Remember this is header+address+2 parity checks
uint8_t nSecondData = 0b11010001; // Remember this is the data. 

// Extract components from received frame
uint8_t receivedHeader = (nFirstData >> 6) & 0b11;
uint8_t receivedAddress = (nFirstData >> 2) & 0b1111;
uint8_t receivedEvenParityBit = (nFirstData >> 1) & 0b1;
uint8_t receivedOddParityBit = nFirstData & 0b1;
uint8_t receivedData = nSecondData;

// Verify parity bits for received data like in the previous section
uint8_t tempData = receivedData;
uint8_t counter = 0;

// Count number of 1s in received data
while (tempData) {
    counter += tempData & 1;
    tempData >>= 1;
}

// Calculate expected parity bits
uint8_t expectedEvenParityBit = (counter % 2) == 0;
uint8_t expectedOddParityBit = (counter % 2) == 1;

// Verify parity checks
if (receivedEvenParityBit != expectedEvenParityBit) {
    printf("ERROR: Even parity check failed! Expected: %d, Received: %d\n", 
           expectedEvenParityBit, receivedEvenParityBit);
    // Handle error as needed from here
}

if (receivedOddParityBit != expectedOddParityBit) {
    printf("ERROR: Odd parity check failed! Expected: %d, Received: %d\n", 
           expectedOddParityBit, receivedOddParityBit);
    // Handle error as needed from here
}

printf("Parity checks passed successfully!\n");

// Process the data according to requirements:
// 1. Flip bits 0, 2, 4, 6 using XOR with FlippingMask
uint8_t processedData = receivedData ^ FlippingMask;

// 2. Write value 1 to bits 1 and 3 using OR with WriteMask
processedData |= WriteMask;

// Calculate parity bits for the processed data
tempData = processedData;
counter = 0;

// Count number of 1s in processed data
while (tempData) {
    counter += tempData & 1;
    tempData >>= 1;
}

// Calculate parity check bits for new data
uint8_t evenParityBit = (counter % 2) == 0;
uint8_t oddParityBit = (counter % 2) == 1;

// Construct the frame for writing (big endian format)
// First byte: header (2 bits) + address (4 bits) + parity bits (2 bits)
uint8_t writeFirstByte = (s_nWriteHeader << 6) | (s_nAddress << 2) | (evenParityBit << 1) | oddParityBit;

// Second byte: processed data
uint8_t writeSecondByte = processedData;

// Now send the bytes out sequentially, but for our purpose, this is done! 
```



## Related Notes

[[Communication with SPI]]

---

Last modified on 2025.05.21