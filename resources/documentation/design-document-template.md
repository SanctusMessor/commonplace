# Design Document Template

> Google Link: [https://docs.google.com/document/d/13\_F8\_nAIrYj3c\_L9LVTMS7pZrP66O1fUAvRfonAE9Gk/edit?usp=sharing](https://docs.google.com/document/d/13_F8_nAIrYj3c_L9LVTMS7pZrP66O1fUAvRfonAE9Gk/edit?usp=sharing)
>
> As someone who has written a lot of documentation, I can tell you my approach
>
> First, start with a high level summary of what the thing is does, as well as what it is supposed to do. The second part should outline things like the problem it’s solving, the challenges faced, the assumptions made. This way the reader can understand the why better
>
> I then go through the major components at a high level - here I used an relational data as because xyz requirement, here I used a firewall with xyz ports open because requirements, I used xyz for DNS, etc
>
> Then tackle lower level stuff - database designs, throughout requirement, record sizes, service discovery, failover strategies
>
> Then a config section where I document any touchy or special configs that are relevant
>
> Then a section on operations - what are some common operations tasks this thing may need, or things the oncall folks may need \(how to stop/start. Where logs are, how to access things, how to scale, what the limits are of the system\), any caveats or ugliness that exists, where the code is, how to deploy with teraform etc
>
> Finally, I include some test results that are relevant - how many writes you can do, how fast we reconverge on failover, how many things per unit we can do with the design
>
> I have a template I’ve built up over the years that I use - I just open it up and start filling in sections, removing irrelevant parts. But the overall idea is to start very high level and drill down into the parts that most need describing. Don’t go into detail on common elements or well understood tech, just the stuff that the intended audience would need, but always with an eye to “what details will we all forget in 3 years when we have to fix this thing”
>
> And always lots of pictures/diagrams
>
> _update_ - link to a re-created template as a Google doc: [https://docs.google.com/document/d/13\_F8\_nAIrYj3c\_L9LVTMS7pZrP66O1fUAvRfonAE9Gk/edit?usp=sharing](https://docs.google.com/document/d/13_F8_nAIrYj3c_L9LVTMS7pZrP66O1fUAvRfonAE9Gk/edit?usp=sharing)

{% embed url="https://www.reddit.com/r/aws/comments/dxmkci/how\_do\_you\_document\_your\_infrastructure/f7suoix?utm\_source=share&utm\_medium=web2x" caption="" %}

