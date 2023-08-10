## Approach

Since this was fairly a large codebase with complex implementations, I decided to only look into contracts that I found interesting. Thus I was able to focus greatly on the given section of code and clearly check if the code is strictly implementing what it is supposed to do. This enabled me to dig 'deeper' into the selected codebases and uncover vulnerabilities. I targeted codebases by following the lifecycle of the Election 'process', which helped me to gain deep understanding.

## Takeaways

Learned a ton about Arbitrum protocol and its underlying architecture and its constitution. It was fun to learn more about upgradeable governor contracts. The code was very well written with loads of comments and kudos to the team to almost creating a perfect system. Also there were existing test suits with large coverage which helped as reference at times.

## Comments

The code is well written however, persistent changes are not taken into account at some times which can lead to bad implementation logic. And the error in understanding this issue is carried out through multiple issues. Which has lead to multiple different implementation issues stemming from some root understanding faults.  



### Time spent:
10 hours