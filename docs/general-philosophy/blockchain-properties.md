!!! tip

    For comprehensive insights into secure development practices, consider visiting the [Development Recommendations](https://scsfg.io/developers/) section of the Smart Contract Security Field Guide. This resource provides in-depth articles to guide you in developing robust and secure smart contracts.

While much of your programming experience will be relevant to Ethereum programming, there are some
pitfalls to be aware of.

- Be extremely careful about external contract calls, which may execute malicious code and change
  control flow.
- Understand that your public functions are public, and may be called maliciously and in any order.
  The private data in smart contracts is also viewable by anyone.
- Keep gas costs and the block gas limit in mind.
- Be aware that timestamps are imprecise on a blockchain, miners can influence the time of
  execution of a transaction within a margin of several seconds.
- Randomness is non-trivial on blockchain, most approaches to random number generation are gameable
  on a blockchain.
