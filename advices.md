# Essays on programming I think about a lot

[Source](https://www.benkuhn.net/progessays/)

> Let’s say every company gets about three innovation tokens. You can spend these however you want, but the supply is fixed for a long while. You might get a few more after you achieve a certain level of stability and maturity, but the general tendency is to overestimate the contents of your wallet. Clearly this model is approximate, but I think it helps.
If you choose to write your website in NodeJS, you just spent one of your innovation tokens. If you choose to use MongoDB, you just spent one of your innovation tokens. If you choose to use service discovery tech that’s existed for a year or less, you just spent one of your innovation tokens. If you choose to write your own database, oh god, you’re in trouble.


**This essay convinced me that “don’t repeat yourself” (DRY) isn’t a good motto.**

> * Time passes. 
> * A new requirement appears for which the current abstraction is almost perfect.
> * Programmer B gets tasked to implement this requirement.
> * Programmer B feels honor-bound to retain the existing abstraction, but since isn’t exactly the same for every case, they alter the code to take a parameter….
> * … Loop until code becomes incomprehensible.
> * You appear in the story about here, and your life takes a dramatic turn for the worse.


> **If we see ‘lines of code’ as ‘lines spent’**, then when we delete lines of code, we are lowering the cost of maintenance. Instead of building re-usable software, we should try to build disposable software.
> Business logic is code characterised by a never ending series of edge cases and quick and dirty hacks. This is fine. I am ok with this. Other styles like ‘game code’, or ‘founder code’ are the same thing: cutting corners to save a considerable amount of time.
> The reason? Sometimes it’s easier to delete one big mistake than try to delete 18 smaller interleaved mistakes. **A lot of programming is exploratory, and it’s quicker to get it wrong a few times and iterate than think to get it right first time.**
