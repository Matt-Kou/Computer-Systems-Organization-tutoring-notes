# Homework 22

## Question
Consider a cache with the following properties, which are essentially the ones we have been using to date:
```
movl $0x11ff, 0x0
movl 0x0, %r8
movl $0x22FF, 0x80
movl 0x0, %r9
movl $0x33FF, 0x8
movl $0x44FF, 0x8
movl $0x55FF, 0x38
movl $0x66FF, 0x28
movl 0x38, %r10
```
Direct mapped (there is only one possible location in the cache for the contents of a given memory location).
Write-through and store-allocate (i.e., the Decstation 3100).
All references are to (4-byte) words.
The block size is one word.
The memory uses 32-bit addresses.
The cache has 16 entries (i.e., 16 blocks or 16 lines).
The cache is initially empty, i.e. all the valid bits are 0. Then the references on the right are issued in the order given.

For each instruction tell whether the memory reference is a cache hit, a cache miss due to the entry being invalid, or a cache miss due to a tag mismatch.
After all the instructions have been executed what is the contents of the cache, that is, for each line, give its validity, its tag, and its contents. If one or more fields are still junk, label them junk.

## Answer

### Separation of bits in the address
1. Since the cache has 16 entries, there will be $\log_2{16}=4$ index bits.
1. Since all reference are to one 4-byte word, the number of offset bit is $\log_2 4=2$.
1. The number of tag bits is $32-4-2=26$.

32-bit-address:
|-------------26 tag bits-------------|--4 index bits-|-2 offset bits-|

### Stepping through each instructions (the bits are represented in binary)

1. ``movl , 0x0`` 
    
    index: 0, tag: 0. index 0 is originally uninitialized, so its valid bit was 0. So now the cache would set the valid bit of index 0 to 1 and set the tag to 0. This is a cache miss due to the entry being invalid. 
    
    
    Current cache table:
    | index | valid | tag | data |
    |-------|-------|-----|------|
    | 0000  |  1     | 0    |  $0x11ff    |
    
1. ``movl 0x0, %r8``
    
    index: 0, tag: 0. Since this is accessing the address we just cached into the memory, this is going to be a cache hit. 
    
    
    Current cache table:
    | index | valid | tag | data |
    |-------|-------|-----|------|
    | 0000  |  1     | 0    |  $0x11ff    |

1. `movl	$0x22FF,	0x80`
    
    0x80 is 1000 0000 in binary. By splitting it into tag, index, and offset: 10|0000|00, we get: index: 0, tag: 10. The index of 0 currently has the tag of 0, so this is a cache miss due to a tag mismatch. Now we change the tag of index 0 to 10 and move the new data inside.
    
    
    Current cache table:
    | index | valid | tag | data |
    |-------|-------|-----|------|
    | 0000  |  1     | 10    |  $0x22FF    |

1. `movl	0x0,	%r9`
    
    index: 0, tag: 0. This case is very similar to the one above. The tag does not match at index 0 so this is a cache miss due to a tag mismatch. Then, the data of 0x0 is moved back into the cache.
    
    
    Current cache table:
    | index | valid | tag | data |
    |-------|-------|-----|------|
    | 0000  |  1     | 0    |  $0x11ff    |

1. `movl	$0x33FF,	0x8`

    0x8 is 0|00 10|00 in binary (the "|" sign is used to separate the bits), so the index is 10 and the tag is 0. Since there is no index of 10 in the cache, this is a cache miss due to the entry being invalid. 
    
    
    Current cache table:
    | index | valid | tag | data |
    |-------|-------|-----|------|
    | 0000  |  1     | 0    |  $0x11ff    |
    | 0010  |  1     | 0    |  $0x33FF    |

1. `movl	$0x44FF,	0x8`

    Since this is accessing the address that was just cached, this is a cache hit. 
    
    
    Current cache table:
    | index | valid | tag | data |
    |-------|-------|-----|------|
    | 0000  |  1     | 0    |  $0x11ff    |
    | 0010  |  1     | 0    |  $0x44FF    |

1. `movl	$0x55FF,	0x38`

    0x38 is 00|11 10|00 in binary. index: 1110, tag: 0. Since this is a new index, this is a cache miss due to the entry being invalid.
    
    
    Current cache table:
    | index | valid | tag | data |
    |-------|-------|-----|------|
    | 0000  |  1     | 0    |  $0x11ff    |
    | 0010  |  1     | 0    |  $0x44FF    |
    | 1110  |  1     | 0    |  $0x55FF    |

1. `movl	$0x66FF,	0x28`

    Same as the one above.
    
    
    Current cache table:
    | index | valid | tag | data |
    |-------|-------|-----|------|
    | 0000  |  1     | 0    |  $0x11ff    |
    | 0010  |  1     | 0    |  $0x44FF    |
    | 1010  |  1     | 0    |  $0x66FF    |
    | 1110  |  1     | 0    |  $0x55FF    |

1. `movl	0x38,	%r10`

    0x38 is 00|11 10|00 in binary. index: 1110, tag: 0. Since both the index and tag bits matches in the cache. This is a cache hit.
    
    
    Current cache table:
    | index | valid | tag | data |
    |-------|-------|-----|------|
    | 0000  |  1     | 0    |  $0x11ff    |
    | 0010  |  1     | 0    |  $0x44FF    |
    | 1010  |  1     | 0    |  $0x66FF    |
    | 1110  |  1     | 0    |  $0x55FF    |

Everything that is not mentioned in the table are invalid and their data is junk.