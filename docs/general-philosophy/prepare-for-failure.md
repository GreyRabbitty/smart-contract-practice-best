!!! tip

    For comprehensive insights into secure development practices, consider visiting the [Development Recommendations](https://scsfg.io/developers/) section of the Smart Contract Security Field Guide. This resource provides in-depth articles to guide you in developing robust and secure smart contracts.

Any non-trivial contract will have errors in it. Your code must, therefore, be able to respond to
bugs and vulnerabilities gracefully.

- Pause the contract when things are going wrong ('circuit breaker')
- Manage the amount of money at risk (rate limiting, maximum usage)
- Have an effective upgrade path for bugfixes and improvements
