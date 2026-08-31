![[L5-Transmission-Errors-1.pdf]]

- Noise Types
	- White Noise
	- Impulse Noise (Thunder)
	- Crosstalk (Due to electrical induction)
	- Echo
	- Delay (constant) (can reduce using buffers)
	- Jitter (variability of delaying)
	- Delay Distortion
	- Attenuation (continues signal loss)(controlled lengths)(fiber optics distortion ratios is higher than other medias)(use repeaters to intermediate boosting)

![[L5-Transmission-Errors-2.pdf]]

- Fault Tolerance
	- Some additional data
	- Some redundancy
- Goal
	- Detect error
	- Best case - Can recover at the receiver level  (Coding - include additional bits(codes) to the requested data)
- Scheme for a codeword - Determines if a codeword is valid or not
- Inseparable ->  Requires extra calculations to separate data.

### Parity

##### Even Parity
![[Pasted image 20260831111524.png]]
- Parity-bit => Number of ones is even (4) => So whole word is even => 0

![[Pasted image 20260831111712.png]]
- Parity-bit => Number of ones is odd (3) => We have to make whole even =>1

- Cons. -> Only Deal with odd number of bit errors can be detected.

#### Hamming Distance

![[Pasted image 20260831112211.png|285]]
010 -> 011 using LSB change in to 1 => Hamming distance is 1

(7,4)SEC -> Codeword length = 7  (4 data,3 parity)