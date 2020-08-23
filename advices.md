# Essays on programming I think about a lot

[Source](https://www.benkuhn.net/progessays/)

> Let’s say every company gets about three innovation tokens. You can spend these however you want, but the supply is fixed for a long while. You might get a few more after you achieve a certain level of stability and maturity, but the general tendency is to overestimate the contents of your wallet. Clearly this model is approximate, but I think it helps.
If you choose to write your website in NodeJS, you just spent one of your innovation tokens. If you choose to use MongoDB, you just spent one of your innovation tokens. If you choose to use service discovery tech that’s existed for a year or less, you just spent one of your innovation tokens. If you choose to write your own database, oh god, you’re in trouble.


This essay convinced me that “don’t repeat yourself” (DRY) isn’t a good motto.

> Time passes. 
> A new requirement appears for which the current abstraction is almost perfect.
> Programmer B gets tasked to implement this requirement.
> Programmer B feels honor-bound to retain the existing abstraction, but since isn’t exactly the same for every case, they alter the code to take a parameter….
> … Loop until code becomes incomprehensible.
> You appear in the story about here, and your life takes a dramatic turn for the worse.
