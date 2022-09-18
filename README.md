# digirule2A-multiply-23bit
Mulitplication of two 23-bit-numbers on a Digirule 2A

```

Program for the Digirule 2A to multiply two unsigned 23-bit integers, giving
a maximum result of 8388607, e.g. by multiplying
ALPHA1: 10101010 least significant byte
ALPHA2: 10101010
ALPHA3: 00101010 most significant byte
and
BETA1:  00000011 least significant byte
BETA2:  00000000
BETA3:  00000000 most significant byte
,giving:
GAMMA1: 11111110 least significant byte
GAMMA2: 11111111
GAMMA3: 01111111 most significant byte
i.e. 2796202 * 3 = 8388606

You enter the desired numbers ALPHA and BETA, then return to address 0, then
press Run. Then examine addresses 8, 9 and 10 for GAMMA1, GAMMA2 and GAMMA3,
where GAMMA1 is the least significant result byte and GAMMA3 is the most
significant result byte.

CODE IN CAPS, binary translation next to it, comments in lowercase. Jump
target addresses are before colons.

prepare variables
JUMP 18          00011100 00010010          ; skip the variables' space
ALPHA1           xxxxxxxx                   ; first number, user set:
ALPHA2           xxxxxxxx                   ; alpha1 is the least significant,
ALPHA3           xxxxxxxx                   ; alpha3 the most significant byte
BETA1            xxxxxxxx                   ; second number, user set:
BETA2            xxxxxxxx                   ; beta1 least significant byte,
BETA3            xxxxxxxx                   ; beta3 most significant byte
GAMMA1           00000000                   ; result bytes - gamma1 least
GAMMA2           00000000                   ; significant, gamma3 most
GAMMA3           00000000                   ; significant, gamma4 is an
GAMMA4           00000000                   ; internal carry flag
DELTA1           00000000                   ; copy of beta used for shifting
DELTA2           00000000                   ; and producing, bit by bit, the
DELTA3           00000000                   ; YPSILON flag that decides to add
YPSILON          00000000                   ; or not to add the shifted ALPHA
(UNUSED)         00000000                   ; (reserved rest from experiments)
PSI              00000000                   ; the wordlength, later set to 24b

make a temporary working copy of beta
JUMP 135         00011100 10000111          ; go copy beta into delta

main loop
20: COPYLR 24 PSI         00000011 00011000 00010001 ; count bit progression
23: DECRJZ YPSILON        00010100 00001111 ; main loop - decide to add or not
JUMP 29          00011100 00011101          ; if no 1 is found, skip addition
JUMP 88          00011100 01011000          ; otherwise go to addition subprog
29: COPYLR 0 YPSILON      00000011 00000000 00001111 ; reset it to 0
JUMP 54          00011100 00110110          ; goto lshift alpha, rshift delta
NOP              00000001                   ; and slicing off delta an ypsilon
35: COPYRR GAMMA1 BETA1   00000111 00001000 00000101 ; copy the intermediate
COPYRR GAMMA2 BETA2       00000111 00001001 00000110 ; result back into beta
COPYRR GAMMA3 BETA3       00000111 00001010 00000111 ; for another addition
COPYLR 0 GAMMA4           00000011 00000000 00001011 ; clear the extra carry
DECRJZ PSI       00010100 00010001          ; last bit sliced off - terminate
JUMP 23          00011100 00010111          ; otherwise, loop another additon
JUMP 146         00011100 10010010          ; post-process: rshift the result
(UNUSED)         00000000                   ; end of main loop

shift delta to the right through ypsilon to determine whether to add
another power of alpha later on to the intermediate result
54: COPYLR 0 252 00000011 00000000 11111100 ; clear the flags before shifting
SHIFTRL ALPHA1   00010110 00000010          ; left-shift through carry all
SHIFTRL ALPHA2   00010110 00000011          ; alpha in preparation of an
SHIFTRL ALPHA3   00010110 00000100          ; addition at the next power of 2
COPYRL 0 252     00000011 00000000 11111100 ; clear the flags before shifting
SHIFTRR DELTA3   00010111 00001110          ; rshift delta, slicing of its
SHIFTRR DELTA2   00010111 00001101          ; least sign. byte into ypsilon to
SHIFTRR DELTA1   00010111 00001100          ; determine adding alpha or not
COPYRR 252 YPSILON        00000111 11111100 00001111 ; get ypsilon from flags
SHIFTRR YPSILON  00010111 00001111          ; make the carry flag go on pos. 0
COPYRA YPSILON   00000110 00001111          ; isolate in ypsilon solely deltas
ANDLA 1          00001100 00000001          ; least significant bit=cf, if set
COPYAR YPSILON   00000101 00001111          ; save the bit into ypsilon and
COPYRL 0 252     00000011 00000000 11111100 ; clear the flags, the go decide
JUMP 35          00011100 00100011          ; if to add next power of alpha

perform a long addition between alpha and the intermediary result in beta
88: COPYLR 0 252 00000011 00000000 11111100 ; clear the flags before addition
COPYRA ALPHA1    00000110 00000010          ; add alphas and betas least
ADDRA BETA1      00001001 00000101          ; significant bytes and store them
COPYAR GAMMA1    00000101 00001000          ; into gammas least signif. byte
COPYLR 0 GAMMA2  00000011 00000000 00001001 ; place the carry flag into the
SHIFTRL GAMMA2   00010110 00001001          ; middle-significant by of gamma
COPYRA GAMMA2    00000110 00001001          ; and put it into the accumulator
ADDRA ALPHA2     00001001 00000011          ; then add to the possible carry
COPYLR 0 GAMMA4  00000011 00000000 00001011 ; the midsignificant byte of alpha
SHIFTRL GAMMA4   00010110 00001011          ; and save any resulting carry
ADDRA BETA2      00001001 00000110          ; add now the midsig. byte of beta 
COPYAR GAMMA2    00000101 00001001          ; to the accumulator and store the
COPYLR 0 GAMMA3  00000011 00000000 00001010 ; final midsig. byte into gamma
SHIFTRL GAMMA3   00010110 00001010          ; get a possible carry into gamma3
COPYRA GAMMA3    00000110 00001010          ; get any such carry into the acc.
ORRA GAMMA4      00001111 00001011          ; try the prev. saved carry, too
ADDRA ALPHA3     00001001 00000100          ; and finally add each most sign.
ADDRA BETA3      00001001 00000111          ; byte of alpha and beta, storing
COPYAR GAMMA3    00000101 00001010          ; the result into gamma, then
COPYLR 0 252     00000011 00000000 11111100 ; cleaning up any set flags and
JUMP 29          00011100 00011101          ; returning the interm. result

create a temporary copy of beta used for shifting and determining to
perform or not to perform the next addition of another power of two of alpha
135: COPYRR BETA1 DELTA1  00000111 00000101 0001100  ; copy beta into delta
COPYRR BETA2 DELTA2       00000111 00000110 0001101  ; in preparation for
COPYRR BETA3 DELTA3       00000111 00000111 0001110  ; rshifting the number
JUMP 20                   00011100 00010100

post-processing - our result is one power of two too much, shift it back
146: COPYLR 0 252         00000011 00000000 11111100 ; clear all flags
SHIFTRR GAMMA3   00010111 00001010          ; restore gamma: it is x2 too much
SHIFTRR GAMMA2   00010111 00001001          ; because we left-shift alpha once
SHIFTRR GAMMA1   00010111 00001000          ; too much, so rshift gamma back
HALT             00000000

```
