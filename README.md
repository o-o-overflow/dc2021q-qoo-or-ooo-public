# DEFCON 2021 Quals: QOO or OOO

`crytpo`  `reverse`

This challenge is about quantum computing + cryptography.

## Run the challenge

Type

```bash
./service.py
```

The script is under `service/src/`. You will need package `pycryptodome` and `qunetsim`.

## Description

This challenge has two steps. In the first step, you need to win the game with [zardus](https://twitter.com/Zardus). You guys need to compete with two QOO team members. As described in `game.py`, you and zardus will bet on two binary numbers, say x and y. Your competitors will also bet on two binary numbers, say a and b. To win the game, you and zardus's numbers need to satisfy:

```
x + y = a * b
```

You need to repeatly play this game for 30 times and win more than 85% of the game.

```python
    def run(self):
        print(f"[Round {self.id}]: Your competitor bets on {self.competitor_bet1}")
        print(OPTIONS)
        selection = input().strip()
        if selection == COIN:
            print(COIN_ROTATE)
            selection = input().strip()
            if selection == LEFT:
                self.coin.rotate_left()
            elif selection == RIGHT:
                self.coin.rotate_right()
            elif selection != NOT_CHANGE:
                return self.error()
            player1_bet = self.coin.flip()  # Just clean it up. Not a hint.
        elif selection == ZERO:
            player1_bet = 0
        elif selection == ONE:
            player1_bet = 1
        else:
            return self.error()

        print(f"[Round {self.id}]: zardus's competitor bets on {self.competitor_bet2}, " +
                f"you bet on {player1_bet}")
        return self.play(player1_bet, self.player2_bet)
```


When you decide on your `x`, you only know your competitor's number `a`. You can choose to play 0, 1, or use a magic coin. After your bet, you will know the other competitor's bet and whether you and zardus win or lose the game.


If you win more than 85% of the game, zardus will send you the chat between him and [adamd](https://twitter.com/adamdoupe). You will see messages like such:

```
zardus receives from adamd: 0:0
zardus receives from adamd: 1:0
zardus receives from adamd: 2:0
...
zardus receives from adamd: -1:1fbc40ed0426a2ff4fa659db26a0f7f3
zardus receives from adamd: -2:7c4cfe551ce75e26454b7d9f8b5fe1f8e2a653af2caa56a2f70936f69019ddf23b09fec90cc9506531d460
```

And the key was encrypted in the message.

## Solution

### Win the game

Thanks to indeterminism, there are many ways to win the game. For example, you can win this game by just always flipping the coin without rotating it.

### Encrypt the message

In class `Zardus`'s `chat()` and class `Adamd`'s `chat()`, you will see that zardus will drop those secret bits if the basis is not the same as adamd's. For Adam, he generates basis just randomly. So you can guess that the secret key will be decreased to about half of the original bits, which is 15 bits.... hmm, this is brute forcable. 

## Exploit

Credits to [defund](https://twitter.com/kleptographic):

```
import itertools
import hashlib
from tqdm import tqdm
from Crypto.Cipher import AES

nonce = bytes.fromhex('dbb0fa17a8ad99e0d74230db43bfbc06')
ct = bytes.fromhex('51bc36d2f7036c29c95cc66fa9a6184afc3d10a79610b55b0f94b618b12adbce847119971d1eeca1f8be82731d06297b74ea3065c3')

for length in tqdm(range(20)):
    for guess in itertools.product((0, 1), repeat=length):
        key = hashlib.md5(bytes(guess)).digest()
        cipher = AES.new(key, AES.MODE_EAX, nonce=nonce)
        flag = cipher.decrypt(ct)
        if b'OOO{' in flag:
            print(flag)

```
