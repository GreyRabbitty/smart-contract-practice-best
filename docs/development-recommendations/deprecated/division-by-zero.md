!!! tip

    For comprehensive insights into secure development practices, consider visiting the [Development Recommendations](https://scsfg.io/developers/) section of the Smart Contract Security Field Guide. This resource provides in-depth articles to guide you in developing robust and secure smart contracts.

Prior to version 0.4, Solidity [returns zero](https://github.com/ethereum/solidity/issues/670) and
does not `throw` an exception when a number is divided by zero. Ensure you're running at least
version 0.4.
