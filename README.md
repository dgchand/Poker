# Poker

## Introduction

Recent advancements in game AI have produced astounding results, such as Google's AlphaGo and OpenAI's Dota bot. However, for many years, the goal of writing a computer program to beat humans at No Limit Texas Holdem has remained elusive. That is until 2017, when researchers at Carnegie Mellon University were finally able to crack the barrier, when their bot Libratus defeated top human opponents with statistical significance. (cite) Libratus required a supercomputer, but in 2018 a technique called Depth Limited Solving was discovered which vastly reduced the required computational resources to the point that a superhuman poker program could run on your laptop. (cite) The goal of this project is to use depth-limited solving to implement the first opensource superhuman AI for Heads Up No Limit Texas Holdem. 

## Running the program
TODO

## How this works

Poker is an "imperfect information game", which means it's a lot harder to solve than perfect information games like chess. In chess you can just search the game tree for the best move, but in poker, you don't actually know where you are in the game tree because you can't see your opponent's cards. So instead of solving for a "best move" you have to solve for an optimal strategy--the Nash equilibrium.

A simple example of a Nash equilibrium is for Rock Paper Scissors. The optimal strategy is to choose randomly between rock, paper, and scissors, and if you do this, you can't lose in the long run. This is the only strategy that is unexploitable, because if you're biased towards any one move, the opponent can use that information against you. There is no one "best move" because you need to choose randomly, otherwise the opponent can adapt to beat you. This is like in poker, when sometimes there is no one optimal move, but you have to switch it up between several actions, because if you did the same thing every time in each spot, your opponent could figure it out and know what cards you have. 

It's guaranteed that for any two player imperfect information game (like Heads Up No Limit Texas Holdem) there exists at least one Nash Equilibrium strategy, and if you play by it, you cannot lose in the long run. In real life though, if you play a Nash Equilibrium strategy in poker, you will destroy your opponents since chances are they aren't playing anything close to an optimal strategy. 

So we know that this strategy exists, but how do we solve for it? In 2007, researchers at the University of Alberta invented a technique known as "Counterfactual Regret Minimization" (cite) which solves for the Nash Equilibrium. This algorithm works by having two bots play against each other. First they just make random moves, but at each decision point, they write down what they regret not doing. Then over time, they do more of what they regret doing and less of what they don't regret. It is proven that if you run this algorithm for long enough, it will give you a Nash Equilibrium strategy. 

While this algorithm works in theory, in reality HUNL Texas Holdem is just too big a game to solve directly. Checkers has 10<sup>21</sup> possible positions, chess has 10<sup>120</sup>, but HUNL has more like 10<sup>160</sup>, which is more than the number of atoms in the universe squared. So in order to solve it before the universe ends, you need to find a way to shrink the game down. This is accomplished using a technique known as **abstraction**, with two prongs of attack:
1. Treat similar hands as identical. AsKs 2c3cAc is close enough to AsKs 3c4cAc. 
2. Treat similar bet sizes as equal. A bet of $1000 is close enough to a bet of $1001. 

When the bot is training, it translates the bets and cards that it sees into their abstracted versions, and calculates its strategy only for those. Then when it comes to playing the game, you have to hope that your abstract strategy will work well for the full game. However, in practice it turns out that this is hard to do, and a worst-case opponent will be able to destroy most abstract strategies, because there will be chinks in the armor that can be exploited. As the abstraction becomes more realistic, better representing the full game, this becomes less of a problem, and historically poker bots have improved by adopting better and better abstractions, which usually required a ton of memory. For example, the bot Baby Tartanian 8 needed 18 TB of RAM and 2 million core hours. (depth limited paper)

So it would be nice if there were a way to use a mediocre abstraction and improve on it in real time. This can be accomplished using **Depth Limited Solving** (cite) which refines the abstract strategy by playing it against several possible opponent strategies. (explain more once I understand it better) Using Depth Limited Solving allows the bot to close the chinks in the abstract strategy and play a true approximate Nash equilibrium. 

## Implementation Details

### Abstraction

#### Card abstraction

Finding good ways to group hands together for abstraction has historically been a hot research area, and finding better abstractions was the key driver of progress before nested subgame solving was invented. Initially, people would manually write down rules to classify hands, such as by type, draws, and stuff like that. Over time, numeric techniques were used to greater success, such as calculating a hand's expected equity or expected equity squared. However, if you just look at these variables, you run into an issue because some vastly different hands can have similar equities. For example, a straight draw could have a similar equity to a low pair, even though they are vastly different types of hands. To better classify hands, it pays to look at the full **equity distribution** over all possible rollouts. *add pictures of graphs* Then these equity distributions are clustered using k-means clustering with the **Earth Mover's Distance** so that hands with similar equity distributions will be put together. (cite paper)

The above analysis only applies to the flop and turn. On the river, there are no rollouts and no notion of an equity distributions, so hands are just clustered based on their equity. On the preflop, there are only 169 strategically distinct hands, so we can just include all of them in the abstraction.

It is up to the programmer to decide how many clusters to use--more clusters will produce a better strategy, but will be slower to train. I chose to use ?? buckets on the flop, ?? on the turn, and ?? on the river. 

#### Action abstraction

Simplifying the actions is a much simpler task and can be succesfully done by a human. Since there are many possible Nash equilibria, chances are good that one will exist for the actions you choose to include in your abstraction. For my bot I allowed the following bet sizes: (half pot, full pot, 2x pot, all in). 

### Training

Once the abstraction is all set, it is time to calculate a blueprint Nash equilibrium strategy. It's okay if this strategy isn't great, because the bot will improve on it later using depth-limited solving. I trained the blueprint strategy using a variant CFR called **Discounted Regret Minimization** (cite) with parameters ???. 

### Measuring Exploitability

It is useful to have a notion of the quality of a strategy, akin to "loss" in machine learning. This quantity is called **exploitability**, which is a measurement of how badly your opponent could beat you if they knew the exact strategy you play by. The exploitability of a Nash equilibrium strategy is 0 because it is not possible to beat a Nash equilibrium strategy in expectation. 

However, to truly calculate the exploitability, you'd have to run CFR to train an opponent against your strategy, and that would take a while. Fortunately, researchers at ??? came up with a greedy way to calculate a lower bound of the exploitability known as the **Local Best Response** (cite) which ends up being pretty good in practice. You can run this during training to see how good your blueprint strategy is getting. 


## Papers cited

1. [Superhuman AI for heads-up no-limit poker: Libratus beats top professionals](https://science.sciencemag.org/content/359/6374/418)
2. [Depth-Limited Solving for Imperfect-Information Games](https://arxiv.org/abs/1805.08195)
3. [Regret Minimization in Games with Incomplete Information](https://poker.cs.ualberta.ca/publications/NIPS07-cfr.pdf)
4. [Potential-Aware Imperfect-Recall Abstraction with Earth Mover’s Distance in Imperfect-Information Games](https://www.cs.cmu.edu/~sandholm/potential-aware_imperfect-recall.aaai14.pdf)
5. [Solving Imperfect-Information Games via Discounted Regret Minimization](https://arxiv.org/abs/1809.04040)
6. [Equilibrium Approximation Quality of Current No-Limit Poker Bots](https://arxiv.org/abs/1612.07547)

## Further resources

- [An Introduction to Counterfactual Regret Minimization](http://modelai.gettysburg.edu/2013/cfr/cfr.pdf)
- [Noam Brown - YouTube](https://www.youtube.com/channel/UCIOWAxXS5dvYKf73B6FR2iw)
- [AI for Imperfect-Information Games: Beating Top Humans in No-Limit Poker](https://www.youtube.com/watch?v=McV4a6umbAY)
- [The State of Techniques for Solving Large Imperfect-Information Games, Including Poker](https://www.youtube.com/watch?v=QgCxCeoW5JI)
