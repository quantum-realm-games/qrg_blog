#Getting Started with Unitary
_Before you begin: You can [view this document as a Colab notebook](https://colab.research.google.com/drive/1hLkJByi4TzWJD32uS_xX2LuyZEh9C-0b?usp=sharing), where you can run the code._

The Unitary library enables a developer to create games based on quantum computing concepts.

This API is under active design and development and is subject to change, perhaps radically, without notice.

##Installation

Unitary is not available as a PyPI package. You can either clone the repository and install the library from source, or install the library by using `pip` directly.

###Install from source

1. Clone the repository.
```python
git clone https://github.com/quantumlib/unitary.git
```
1. Change directory to the source code.
```python
cd unitary/
```
1. Run `pip install` in `unitary/`
```python
pip install .
```

###Install through pip
_Before you begin: create a virtual environment for your project_

Run `pip install` and use the `git+` option to pull the source code from GitHub.
```python
pip install --quiet git+https://github.com/quantumlib/unitary.git
```
If you’re using Google Colab, add an exclamation mark before your `pip` command:
```python
!pip install --quiet git+https://github.com/quantumlib/unitary.git
```

##Start using Unitary
Unitary is now ready for you to use. Import the alpha library into the relevant source file.
```python
import unitary.alpha as alpha
```

###Create a quantum object
A Quantum Object represents an object in a game that can have a quantum state. Almost all objects have some state: position, color, orientation in game space, and so on. Some game objects can change their state during game play. A quantum state means that the classical state of the object (e.g. Color.GREEN) is uncertain until the game needs to use the classical state. At that point we observe (measure) the quantum state to retrieve a classical state.

For example, let’s create a 5x5 game board, 25 squares in total. Each square can have one of two states: empty or full. If we use quantum objects to create our game board squares, we can enable richer game play by deferring the decision about whether a piece occupies a square or not until the game needs to generate a score.

<!--TODO(cognigami) image 5x5 game board-->
![A game board of 25 squares, arranged in a five by five square, with alternating colors.](./board.png)

You must use an enumeration to track state. 

```python
import enum

class Square(enum.Enum):
  EMPTY=0
  FULL=1
```

A `QuantumObject` requires a name and an initial state.

```python
example_square = alpha.QuantumObject('b1', Square.EMPTY)
```

Now let’s create a collection of quantum objects to represent our game board. An empty game board isn’t very useful, so we also populate our board with a single row of tokens at the near edge (rank 1) and far edge (rank 5).

<!--TODO(cognigami) image 5x5 game board with tokens-->
![The game board described above, with round tokens in each square of the top row and the bottom row.](./board_pieces.png)

```python
board = {}
for col in "abcde":
  for rank in "234":
    square_name = col + rank
    board[square_name] = alpha.QuantumObject(square_name, Square.EMPTY)
  for rank in "15":
    square_name = col + rank
    board[square_name] = alpha.QuantumObject(square_name, Square.FULL)
```

###Create a Quantum World
A Quantum World is a container that enables you to define a scope for Quantum Objects. All quantum objects in your Quantum World have the opportunity to interact with each other. The scope enables us to reason about the probability of a Quantum Object having one state versus another. 

A `QuantumObject` must belong to a `QuantumWorld` object before you can use it. A `QuantumObject` can only belong to one `QuantumWorld`.

Let’s create a `QuantumWorld` object to hold our game board.

```python
game_board = alpha.QuantumWorld(board.values())
```

Now you can see the status of the Quantum Objects in a `QuantumWorld` by using the `peek` method. 

```python
print("Square a1 is the near edge-square; it should be full:")
print(game_board.peek([board["a1"]]))
print("Square c3 is the center square on the board; it should be empty:")
print(game_board.peek([board["c3"]]))
```

Note: If you use the `peek` method with no arguments, the `QuantumWorld` returns a sample of all the `QuantumObjects` that belong to the Quantum World.


###Apply Quantum Effects
When you want to change the quantum state to reflect game play, you apply a Quantum Effect to one or more Quantum Objects. For example, you can change the state of any square by using the `Flip` effect (to “flip” the bit, or state; on to off, green to red, and so on).

```python
flip = alpha.Flip()
flip(board["c3"])
```

If we `peek` at our game board, the c3 square should be in the `Square.FULL` state.

```python
print(game_board.peek([board["c3"]]))
```

We could manually implement basic movement by flipping the state of our source square and our target square.

```python
print('Square a1 and a2 before the move:')
print(game_board.peek([board["a1"], board["a2"]]))

flip(board["a1"])
flip(board["a2"])

print('Square a1 and a2 after the move:')
print(game_board.peek([board["a1"], board["a2"]]))
```

However, the `QuantumEffects` API provides a number of convenience methods for basic game actions. For example, we can do a basic move by applying the `Move` effect.

```python
print('Square b1 and b2 before the move:')
print(game_board.peek([board["b1"], board["b2"]]))

move = alpha.Move()
move(board["b1"], board["b2"])

print('Square b1 and b2 after the move:')
print(game_board.peek([board["b1"], board["b2"]]))
```

###Retrieve the classical state
Similarly, you can use a `Split` effect to create a superposition. In our example, we can create uncertainty about the actual location of our token by creating a superposition.

Let’s create a superposition by splitting the token from square d1 to squares c2 and e2. 

<!--TODO(cognigami)image create superposition from d1 to c2e2-->

```python
print('Squares d1, c2, and e2 before the split:')
print(game_board.peek([board["d1"], board["c2"], board["e2"]]))

split = alpha.Split()
split(board["d1"], board["c2"], board["e2"])
```

Before we validate whether the `Split` effect worked, let’s consider what we expect to see. 

Each square has a 50% probability of hosting our token. When we look at our target squares, we observe their classical state at the moment that we observe them, not a probability. If you `peek` at one or both squares once, you see a definite classical state. 
```python
print("Squares d1, c2, and e2 after the split:")
print(game_board.peek([board["d1"], board["c2"], board["e2"]]))
```

If you `peek` again, you might see a different state (or, you might see the same state).
```python
print("Squares c2 and e2 again:")
print(game_board.peek([board["c2"], board["e2"]], count=10))
```

The more often you `peek` at the squares, the closer you get to observing the token in each square about half the time. That’s how we calculate the probability of the outcomes.
```python
print("Square c2, observed 100 times:")
print(game_board.peek([board["c2"]], count=100))
```

You can use the `get_histogram` method to retrieve a histogram of the states of any `QuantumObject` in your `QuantumWorld`.
```python
print("Historgram of square c2 and e2:")
print(game_board.get_histogram([board["c2"]]), game_board.get_histogram([board["e2"]]))
```

By default, the `get_histogram` method samples your `QuantumWorld` 100 times. You can use the `count` option to change the number of samples that `get_histogram` uses to calculate the probability distribution.

When you’re ready to use the classical state, use the `pop` method to retrieve the classical state from your `QuantumWorld`. After you `pop` the classical state, subsequent calls to the `peek` method gives you a stable answer.  

```python
print("Final state of square c2 and e2:")
game_board.pop([board["c2"], board["e2"]])

print("Histogram of square c2, after a call to `pop`:")
print(game_board.get_histogram([board["c2"]], count=10))
```

###Control flow in quantum and classical states separately
You cannot control the flow of your quantum state from your classical game logic, and vice versa. However, Unitary provides a `quantum_if` method to modify the state of one `QuantumObject` based on the state of another.

For an example, we start with a scenario where we know the outcome, then explore a scenario where the outcome is conditional. 

Before we begin, let’s destroy the token on c3 that we created earlier. Set the square to the `Square.EMPTY` state.
```python
flip(board["c3"])
```

On our board, let’s assume a rule that a token can slide to another square only when the path is empty. If we move the token from b5 to b4, the token on a5 cannot reach square c3 (blocked by token on b4).

```python
move(board["b5"], board["b4"])

alpha.quantum_if(board["b4"]).equals(Square.EMPTY).apply(move)(board["a5"], board["c3"])

print(game_board.peek([board["a5"]]))
print(game_board.peek([board["b4"]]))
print(game_board.peek([board["c3"]]),100)
```

Let’s move the token on b4 back to b5, then try again.

```python
move(board["b4"], board["b5"])

alpha.quantum_if(board["b4"]).equals(Square.EMPTY).apply(move)(board["a5"], board["c3"])

print(game_board.peek([board["a5"]]))
print(game_board.peek([board["b4"]]))
print(game_board.peek([board["c3"]],100))
```

The token on the B file was in a classical state, so it blocked the progress of the token on the A file. After we removed the token on the B file, the token on the A-file was free to move. 

Let’s put the token on the B file in a superposition, and observe what happens to the token on the A file.
```python
split(board["b5"], board["b4"], board["c4"])

alpha.quantum_if(board["b4"]).equals(Square.EMPTY).apply(move)(board["a5"], board["c3"])

print(game_board.peek([board["a5"]]))
print(game_board.get_histogram([board["b4"]]))
print(game_board.get_histogram([board["c3"]]))
```

By passing a token in a classical state (a5) through a token in a quantum state (b4), we entangled the classical token and made its outcome quantum. When you `pop` the classical state of the b4/c4 token, the Quantum World also calculates the classical state of the a5/c3 token. You find the token on a5 if the b4/c4 token is on b4, or on c3 if the b4/c4 token is on c4.

<!--image or table chart of outcomes-->

Note that the `apply` method only accepts a `QuantumEffect` object. You cannot provide classical control flow to the quantum state.

<!--TODO(cognigami): Add Phased* QuantumEffect example.-->
<!--TODO(cognigami): Add Qudit States example.-->
