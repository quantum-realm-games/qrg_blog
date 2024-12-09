Title: An AI Player for a Quantum Game
Date: 2024-12-05 11:56
Category: Story
Tags: quantum ai
Description: META
Status: Published

<section markdown="1">
You might have noticed that we launched [version 1.0 of Quantum Chess](https://store.steampowered.com/app/453870/Quantum_Chess/) this week. Over the past eight years, we've given many, many presentations about the lessons we learned building the "quantum" side of the game, but we didn't collect them anywhere.

Designing game mechanics for quantum games isn't always obvious. With the game launched, we hope to revisit the lessons we learned here on this blog, while we build The Next Big Thing.

What we haven't talked about publicly, until now, is the challenge of building an AI player for the game.

We didn't feel right about flipping the switch from Early Release to version 1 until the game had a proper AI player. This was our "final boss" in the game of launching Quantum Chess. This seems like a good place to start.

## Size matters

Our first AI was a small comedy. Let's give chess AI its due: this is a serious and deep area of computer science. If this is your first time thinking about chess AI, then strap in.

Mathematicians have [a lively debate](https://en.wikipedia.org/wiki/Solving_chess#The_complexity_of_chess) about the total number of possible, legal chess positions. A position is any configuration of chess pieces on the board. A legal position is one that can be reached from the starting position by performing legal moves. Estimates come in somewhere between 2x10^20 and 10^50.

You read that upper-bound correctly: 10 with 50 zeros after it.

Why is that important? Think about the simplest possible "AI" player that you could imagine. You might try to search all the positions from the current board, and choose the move that ends in a win. Sure, but the approximate age of the universe is only 4.32 x 10^17 seconds. How many positions do you need to evaluate per second to evaluate all of them? And how long do you intend to make your player wait for an answer?

Chess is big and hard, so let's start with a simpler game. [Tic Tac Toe](https://en.wikipedia.org/wiki/Tic-tac-toe#Gameplay) is a good choice because it sits on the edge between small enough to be trivial and large enough to be interesting.

## A small detour into Tic Tac Toe

So, you want to win a Tic Tac Toe game. Imagine that before you sit to play the game, you carry a list of all possible Tic Tac Toe "positions". A position is any configuration of Xs and Os on the board. You're only interested in "legal" positions, which means that you only want configurations of Xs and Os that you can create by playing the game. Your list should be organized as a tree, where any position points to all legal moves that a player can make from that position. After you remove reflections and rotations, you're left with 765 positions.

image-placeholder

<label for="mn-graph" class="margin-toggle">&#8853;</label>
<input type="checkbox" id="mn-graph" class="margin-toggle"/>
<span class="marginnote">
Our graph is a [Directed Acyclic Graph](https://en.wikipedia.org/wiki/Directed_acyclic_graph) (DAG). DAGs are a well-studied part of Mathematics, and especially Computer Science.</sidebar>
</span>
Each time that you need to make a move, you can look up the current state of the board, and figure out which branch of the tree has the highest probability of terminating in a favorable outcome for you. Make that move, and you should (almost) never lose a game.

Sounds simple, right? Sure, if you ignore all that "figure out which branch has the highest probability..." stuff. Also, beware that hiding in the simple solution is an assumption that you and your opponent always make the optimal move.

Simple or not, the problem is computationally tractable. You can write a program to read the list of positions and optimize it in a tree structure. You can search your tree in the blink of an eye, which can give you an edge in Tic Tac Toe.

## Chess, Trees, Uncertainty

Back to our regularly scheduled chess game: the tree of all possible classical chess positions can have up to 10^50 nodes. That's a little harder to carry in your pocket, or in computational terms, it takes a lot more time and energy to figure out the branch with the highest probability of terminating in a favorable outcome. It's not reasonable to search the whole tree for every move.

Let's put a finer point on this: So, you have a fancy computer that runs at 1 GHz. Let's say that it only takes one operation to evaluate a position, and that you can use every operation on the processor for your search (you can't, but it's close enough so assume that you can).

Congratulations! You can look through 10^9 options per second, and we want to look through as many as 10^50 options. (Now, how do I do this again? Divide by zero--no. Carry the one--no. Oh, I've got it.) Using our 1 GHZ computer, we need 10^41 seconds to look through the tree. That's billions and billions of times longer than the lifetime of the universe!

You'd think that, with all the computational power we have as a species, [chess would be a solved game](https://x.com/elonmusk/status/1789745303806980448?ref_src=twsrc%5Etfw), but it's not. Companies like DeepMind, and projects like StockFish, have great tools to play chess better than any human player, but even those systems aren't perfect. Chess AIs have to work with some uncertainty about the outcome.

You can probably see that the step from the Tic Tac Toe AI to a Chess AI is a whopping big step. The step from a classical Chess AI to a quantum chess AI is even bigger.

-   Quantum Chess players can create a quantum superposition of boards by playing a Split move. On the first Split, the AI now has to evaluate the next best move for two different games. At that moment, each game has an equal probability of being the game in your universe. Congratulations, you just doubled the size of your computational problem. (And that's only on the first Split!)

-   So, now you have multiple boards (at least two). Boards can become entangled with each other. This means that some Boards depend on others for their existence. If you measure the system and find that one of your boards is not likely in your reality, others might disappear alongside it. While your AI is trying to figure out which move to make next, it has to account for entangled realities in its calculations.

-   The AI should use quantum phase information (interference) to steer the probabilities in favor of one outcome vs another. So, you're not just considering how to move a piece in one or more (possibly) entangled universes. Ideally, you want to use the movement of a piece to affect the probability that some universes are more likely to be your universe than others.

And we haven't even talked about dealing with the possibility of illegal chess positions in some universes. We'll talk about that later.

We knew that this problem would be a constant companion for years to come, so we started off with easy, naive experiments. Stockfish is one of the top chess AIs in the world, and it's open source, so that was our first stop.
</section>
