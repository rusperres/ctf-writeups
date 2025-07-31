# Image Steganography Challenge

**Category:** Image Steganography  
**Solver:** Dalrho

## Challenge Overview

We were given an image and nothing else to work with.

## Approach

Since there were no hints, I decided to use online steganography tools. This challenge was solved mainly through trial and error.

Eventually, I found a tool that can extract hidden images embedded in the least significant bits (LSBs) of another image. Using this tool, I was able to reveal a hidden image that contained the flag.

## Flag
CITU{hidden_frame101}
