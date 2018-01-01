---
title:  Let's Build Your Own Blockchain in Ruby (From Zero / Scratch)
---

Crypto God? Ruby Ninja / Rockstar?

Yes, you can.

![](i/fake-dilbert-blockchain.png)


## Code, Code, Code - A Blockchain in Ruby in 20 Lines! A Blockchain is a Data Structure

What's Blockchain?

It's a list (chain) of blocks linked and secured by digital fingerprints (also known as
crypto hashes).


``` ruby
require "digest"    # for hash checksum digest function SHA256

class Block

  attr_reader :index
  attr_reader :timestamp
  attr_reader :data
  attr_reader :previous_hash
  attr_reader :hash

  def initialize(index, data, previous_hash)
    @index         = index
    @timestamp     = Time.now
    @data          = data
    @previous_hash = previous_hash
    @hash          = calc_hash
  end

  def calc_hash
    sha = Digest::SHA256.new
    sha.update( @index.to_s + @timestamp.to_s + @data + @previous_hash )
    sha.hexdigest
  end
```

(Source: [openblockchains/awesome-blockchains/blockchain.rb](https://github.com/openblockchains/awesome-blockchains/blob/master/blockchain.rb/blockchain.rb))

Yes, that's it.


Let's add two helpers (`first`, `next`) for building ("mining") blocks.

``` ruby
class Block

  def self.first( data="Genesis" )    # create genesis (big bang! first) block
    ## uses index zero (0) and arbitrary previous_hash ("0")
    Block.new( 0, data, "0" )
  end

  def self.next( previous, data="Transaction Data..." )
    Block.new( previous.index+1, data, previous.hash )
  end

end  # class Block
```

(Source: [openblockchains/awesome-blockchains/blockchain.rb](https://github.com/openblockchains/awesome-blockchains/blob/master/blockchain.rb/blockchain.rb))


Let's get started -  build a blockchain a block at a time!

``` ruby
b0 = Block.first( "Genesis" )
b1 = Block.next( b0, "Transaction Data..." )
b2 = Block.next( b1, "Transaction Data......" )
b3 = Block.next( b2, "More Transaction Data..." )

blockchain = [b0, b1, b2, b3]

pp blockchain     ## pp (pretty print to console)
```


> Wait, so a blockchain is just a linked list?
>
> No. A linked list is only required to have a reference to the previous element,
> a block must have an identifier depending on the previous block's identifier,
> meaning that you cannot replace a block without recomputing every single block that comes after.
> In this implementation that happens as the previous digest is input in the calc_hash method.


will log something like:

```
[#<Block:0x1eed2a0
  @index         = 0,
  @timestamp     = 1637-09-15 20:52:38,
  @data          = "Genesis",
  @previous_hash = "0",
  @hash          = "edbd4e11e69bc399a9ccd8faaea44fb27410fe8e3023bb9462450a0a9c4caa1b">,
 #<Block:0x1eec9a0
  @index         = 1,
  @timestamp     = 1637-09-15 21:02:38,
  @data          = "Transaction Data...",
  @previous_hash = "edbd4e11e69bc399a9ccd8faaea44fb27410fe8e3023bb9462450a0a9c4caa1b",
  @hash          = "eb8ecbf6d5870763ae246e37539d82e37052cb32f88bb8c59971f9978e437743">,
 #<Block:0x1eec838
  @index         = 2,
  @timestamp     = 1637-09-15 21:12:38,
  @data          = "Transaction Data......",
  @previous_hash = "eb8ecbf6d5870763ae246e37539d82e37052cb32f88bb8c59971f9978e437743",
  @hash          = "be50017ee4bbcb33844b3dc2b7c4e476d46569b5df5762d14ceba9355f0a85f4">,
  ...
```


## What about Proof-of-Work? What about Consensus?

Making (Hash) Mining a Lottery - Find the Lucky Number

``` ruby
def calc_hash
  sha = Digest::SHA256.new
  sha.update( @index.to_s + @timestamp.to_s + @data + @previous_hash )
  sha.hexdigest
end
```

The computer (node) in the blockchain network that computes the
next block with a valid hash wins the lottery.

For adding a block to the chain you get a reward! You get ~25~ 12.5 Bitcoin! (†)

Bitcoin adds a block every ten minutes.

(†) The reward gets halfed about every two years. In Sep'17 you'll get 12.5 Bitcoin.


### What's SHA256?

Random SHA256 hash #1: `c396de4c03ddb5275661982adc75ce5fc5905d2a2457d1266c74436c1f3c50f1`

Random SHA256 hash #2: `493131e09c069645c82795c96e4715cea0f5558be514b5096d853a5b9899154a`

Triva Q: What's SHA256?

- (A) Still Hacking Anyway
- (B) Secure Hash Algorithm
- (C) Sweet Home Austria
- (D) Super High Aperture


A: SHA256 == Secure Hash Algorithms 256 Bits

Trivia: Designed by the National Security Agency (NSA) of the United States of America (USA).

Secure == Random  e.g. Change one Letter and the Hash will Change Completely

Making (Hash) Mining a Lottery - Find the Lucky Number

Find a hash that starts with ten leading zeros e.g.

`0000000000069645c82795c96e4715cea0f5558be514b5096d853a5b9899154a`

Hard to compute! Easy to check / validate.



### Find the Lucky Number (Nonce == Number Used Once)

Making (Hash) Mining a Lottery

``` ruby
def compute_hash_with_proof_of_work( difficulty="00" )
  nonce = 0
  loop do
    hash = calc_hash_with_nonce( nonce )
    if hash.start_with?( difficulty )
      return [nonce,hash]    ## bingo! proof of work if hash starts with leading zeros (00)
    else
      nonce += 1             ## keep trying (and trying and trying)
    end
  end
end

def calc_hash_with_nonce( nonce=0 )
  sha = Digest::SHA256.new
  sha.update( nonce.to_s + @index.to_s + @timestamp.to_s + @data + @previous_hash )
  sha.hexdigest
end
```

(Source: [awesome-blockchains/blockchain_with_proof_of_work.rb](https://github.com/openblockchains/awesome-blockchains/blob/master/blockchain.rb/blockchain_with_proof_of_work.rb))



Let's rerun the sample with the proof of work machinery added.
Now the sample will pretty print (pp) something like:

```
[#<Block:0x1e204f0
  @index         = 0,
  @timestamp     = 1637-09-20 20:13:38,
  @data          = "Genesis",
  @previous_hash = "0",
  @nonce         = 242,
  @hash          = "00b8e77e27378f9aa0afbcea3a2882bb62f6663771dee053364beb1887e18bcf">,
 #<Block:0x1e56e20
  @index         = 1,
  @timestamp     = 1637-09-20 20:23:38,
  @data          = "Transaction Data...",
  @previous_hash = "00b8e77e27378f9aa0afbcea3a2882bb62f6663771dee053364beb1887e18bcf",
  @nonce         = 46,
  @hash          = "00aae8d2e9387e13c71b33f8cd205d336ac250d2828011f5970062912985a9af">,
 #<Block:0x1e2bd58
  @index         = 2,
  @timestamp     = 1637-09-20 20:33:38,
  @data          = "Transaction Data......",
  @previous_hash = "00aae8d2e9387e13c71b33f8cd205d336ac250d2828011f5970062912985a9af",
  @nonce         = 350,
  @hash          = "00ea45e0f4683c3bec4364f349ee2b6816be0c9fd95cfd5ffcc6ed572c62f190">,
  ...
```

See the difference?
All hashes now start with leading zeros (`00`)
and the nonce is the random "lucky number" that makes it happen.
That's the magic behind the proof of work.
