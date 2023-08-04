!!! tip

    For comprehensive insights into secure development practices, consider visiting the [Development Recommendations](https://scsfg.io/developers/) section of the Smart Contract Security Field Guide. This resource provides in-depth articles to guide you in developing robust and secure smart contracts.

Do not make refund or claim processes dependent on a specific party performing a particular action
with no other way of getting the funds out. For example, in a rock-paper-scissors game, one common
mistake is to not make a payout until both players submit their moves; however, a malicious player
can "grief" the other by simply never submitting their move - in fact, if a player sees the other
player's revealed move and determines that they lost, they have no reason to submit their own move
at all. This issue may also arise in the context of state channel settlement. When such situations
are an issue, (1) provide a way of circumventing non-participating participants, perhaps through a
time limit, and (2) consider adding economic incentive for participants to submit information in
all of the situations in which they are supposed to do so.
